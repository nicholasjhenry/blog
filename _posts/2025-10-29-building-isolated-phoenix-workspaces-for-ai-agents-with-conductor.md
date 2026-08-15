---
title: Building Isolated Phoenix Workspaces for AI Agents with Conductor
published_url: https://nicholasjhenry.medium.com/building-isolated-phoenix-workspaces-for-ai-agents-with-conductor-a438d161f191
---

# Building Isolated Phoenix Workspaces for AI Agents with Conductor

![Banner](/assets/posts/2025-10-29-building-isolated-phoenix-workspaces-for-ai-agents-with-conductor/conductor.png)

I’ve been experimenting with agentic cerl_crash_banneroding — letting AI agents handle multiple tasks in parallel across my codebase. The promise is compelling: one agent refactors authentication, another builds a new dashboard feature, a third optimizes database queries, all working simultaneously. But I quickly realised the infrastructure requirements.

The core challenge became clear: AI agents can easily work on five features at once, but traditional development environments assume a single developer working on one branch at a time. Without proper isolation, I’d face inevitable conflicts — shared database state, port conflicts, Docker container collisions.

What I needed was infrastructure that matched the agent workflow — isolated workspaces where each agent has its own database, its own running server, and its own resources, all without manual coordination.

[Conductor](https://conductor.build) solved this for me by creating fully isolated workspaces for each branch. This post walks through how I configured my Phoenix application to work seamlessly with Conductor’s workspace model, enabling multiple agents to work on different features in my codebase simultaneously without conflicts.

The recipe uses [mise-en-place](https://mise.jdx.dev) to manage both Elixir/Erlang versions and environment variables, though any dependency manager (like [asdf](https://asdf-vm.com)) combined with any env file manager (like [direnv](https://direnv.net)) would work just as well.

## The Three Requirements

For Conductor to work effectively with any framework, we need to satisfy three constraints:

1. **Fast setup** — New workspaces must spin up quickly (seconds, not minutes)
2. **Isolated resources** — Each workspace needs its own database and HTTP server
3. **Fast teardown** — Cleaning up a workspace should be instant

Let’s see how the recipe accomplishes each.

## Conductor’s Magic: Environment Variables

Before diving into the implementation, it’s important to understand the foundation that makes workspace isolation possible. When Conductor creates a workspace, it automatically provides two critical [environment variables](https://docs.conductor.build):

- `CONDUCTOR_WORKSPACE_NAME` – A unique identifier for the workspace (e.g., `prague`, yes, they are named after cities.)
- `CONDUCTOR_PORT` – The first port in a range of 10 consecutive ports allocated to the workspace (e.g., `55100`)

These environment variables are the “magic sauce” for isolation. In this example, with `CONDUCTOR_PORT`, you get ports 55100-55109 exclusively for this workspace — no other workspace will use them. This means you can bind Postgres to 55100, Phoenix to 55101, and have 8 more ports available for other services if needed.

The `CONDUCTOR_WORKSPACE_NAME` lets you namespace Docker containers, Elixir nodes, and any other resources that need unique identifiers. Together, these two variables enable each workspace to operate in complete isolation without manual coordination.

We’ll see how the recipe uses these variables to satisfy all three requirements, starting with fast setup.

## Fast Setup: Sharing Build Artifacts

The biggest bottleneck when creating a new workspace is recompiling dependencies. Phoenix projects typically have dozens of deps that take minutes to compile from scratch. Conductor workspaces use git worktrees, which share the `.git` directory but have separate working trees — perfect for parallel branch work, but each worktree starts with empty `deps` and `_build` directories.

The solution is `script/worktree`, which runs before setup. This script does three things: symlinks shared config files, copies build artifacts, and transforms Conductor's environment variables into workspace-specific configuration.

First, it symlinks files that should be shared across workspaces:

```sh
symlink_if_exists() {
    filename="$1"
    if [ -e "$filename" ]; then
        echo "==> Skipping $filename already linked."
    elif [ -e "../../$filename" ]; then
        echo "==> Symlinking $filename from main repo root…"
        ln -s "../../$filename" "$filename"
    else
        echo "==> No $filename found in main repo root, skipping."
    fi
}
```

```sh
symlink_if_exists .claude
symlink_if_exists .env
```

Then it copies compiled dependencies and build artifacts from the main workspace:

```sh
echo "==> Copying deps…"
cp -r ../../deps ./deps
echo "==> Copying build…"
cp -r ../../_build ./_build
```

This can be customised to add other artifacts such as [Dialyzer’s Persistent Lookup Table (PLT)](https://www.erlang.org).

Finally, it transforms the Conductor environment variables into workspace-specific configuration by generating a `.env.local` file that sets workspace-specific values, which we will explore next:

```sh
create_env_local() {
    compose_project_name="${COMPOSE_PROJECT_NAME:-}"
    workspace_name="${CONDUCTOR_WORKSPACE_NAME:-}"
    conductor_port="${CONDUCTOR_PORT:-}"

    # ... validation checks ...

    port=$((conductor_port + 1))
    cat >.env.local <<EOF
COMPOSE_PROJECT_NAME=${compose_project_name}-${workspace_name}
NODE_NAME=${workspace_name}
DATABASE_PORT=${conductor_port}
PORT=${port}
EOF
}
```

The result: a new workspace is ready in **seconds instead of minutes**.

## Isolated Resources: Namespacing Everything

The harder problem is resource isolation. If two workspaces try to bind to port 4000, one fails. If they share a database, migrations conflict. We need each workspace to have its own universe of resources.

### Docker Compose Namespacing

As we saw earlier, Conductor provides `CONDUCTOR_WORKSPACE_NAME` and `CONDUCTOR_PORT` to each workspace. The `script/worktree` script transforms these into workspace-specific env vars in `.env.local`:

- `COMPOSE_PROJECT_NAME` – Namespaced with the workspace name
- `DATABASE_PORT` – Set to `CONDUCTOR_PORT` (first port in the allocated range)

Docker Compose reads these values to isolate containers. The `docker-compose.yaml` binds Postgres to `DATABASE_PORT`:

```yaml
services:
  db:
    image: postgres:18.0-alpine3.22
    ports:
      - "${DATABASE_PORT:-5432}:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      PGDATA: /pgdata
    volumes:
      - pgdata:/pgdata
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 2s
      timeout: 5s
      retries: 10
      start_period: 10s
volumes:
  pgdata: {}
```

Because `COMPOSE_PROJECT_NAME` includes the workspace name (e.g. `my-app-prague`), Docker creates separate container sets for each workspace. Workspace A’s Postgres runs on port 55100. Workspace B’s runs on `55200`. No conflicts.

### Phoenix Configuration

Phoenix needs to connect to the correct database port and serve HTTP on a unique port. In `config/dev.exs`:

```elixir
config :conductor_recipe, ConductorRecipe.Repo,
  username: "postgres",
  password: "postgres",
  hostname: "localhost",
  database: "conductor_recipe_dev",
  port: String.to_integer(System.get_env("DATABASE_PORT", "5432")),
  stacktrace: true,
  show_sensitive_data_on_connection_error: true,
  pool_size: 10

config :conductor_recipe, ConductorRecipeWeb.Endpoint,
  http: [ip: {127, 0, 1}, port: String.to_integer(System.get_env("PORT") || "4000")],
  check_origin: false,
  code_reloader: true,
  debug_errors: true,
  # ... rest of config
```

The Repo connects to `DATABASE_PORT` for Postgres. The HTTP server runs on `PORT`. When these env vars are undefined (local development), everything defaults to standard ports (`5432` and `4000`).

The same pattern applies in `config/test.exs`, ensuring test databases are isolated:

```elixir
config :conductor_recipe, ConductorRecipe.Repo,
  username: "postgres",
  password: "postgres",
  hostname: "localhost",
  database: "conductor_recipe_test#{System.get_env("MIX_TEST_PARTITION")}",
  port: String.to_integer(System.get_env("DATABASE_PORT", "5432")),
  pool: Ecto.Adapters.SQL.Sandbox,
  pool_size: System.schedulers_online() * 2
```

## The Script Chain

Conductor’s `conductor.json` defines three lifecycle hooks:

```json
{
  "scripts": {
    "setup": "script/worktree; script/setup",
    "run": "script/server",
    "archive": "script/archive"
  }
}
```

When Conductor creates a workspace, it runs `setup`, which chains `script/worktree` (copies deps and generates `.env.local`) and `script/setup` (installs runtime dependencies, starts Docker, and runs `mix setup`).

The `script/server` launches Phoenix with the workspace’s unique node name for the `run` task:

```sh
#!/bin/sh
set -e
cd "$(dirname "$0")/.."

script/update

NODE_NAME=${NODE_NAME:-test}
iex --name $NODE_NAME@127.0.0.1 --cookie mycookie -S mix phx.server
```

The `NODE_NAME` comes from the `.env.local` file generated by `script/worktree`, giving each Elixir node a unique name and preventing distributed Erlang conflicts if multiple workspaces run simultaneously.

## Fast Teardown: Docker’s Gift

Since all workspace resources live in Docker containers, namespaced by `COMPOSE_PROJECT_NAME`, cleanup is trivial. The `script/archive` simply runs for the `archive` task:

```sh
#!/bin/sh
set -e
cd "$(dirname "$0")/.."

eval "$(mise env)"
docker compose down --volumes --remove-orphans
```

Docker Compose removes the containers, volumes, and networks in seconds. No orphaned processes, no lingering databases, no manual cleanup.

## The Result

With this configuration, you can safely run AI agents on multiple branches simultaneously. Each workspace is completely isolated:

- Environment variables automatically configured per workspace
- Unique ports for Postgres database and Phoenix servers
- Separate Docker namespaces preventing container conflicts
- Separate Elixir node names preventing distributed system collisions
- Shared compiled dependencies for fast startup

**Demonstration** using Conductor for development with Phoenix: https://youtu.be/ZXyyITH58DY

The pattern generalises to any Phoenix app. Copy the `script/` directory, add the `conductor.json`, update your `config/dev.exs` and `config/test.exs` to read `DATABASE_PORT` (`PORT` is already in place), and update your `docker-compose.yaml` to use `DATABASE_PORT`. The `script/worktree` will handle generating workspace-specific `.env.local` files automatically. Your agents can then work on multiple features in parallel without tripping over each other.

The full recipe is available in the [GitHub repo](https://github.com) as a starting template for Phoenix projects.
