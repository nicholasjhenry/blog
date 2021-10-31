---
title: Preventing Brain Freeze - Leverage your source code for onboarding
published_url:
---

# Leverage your source code for onboarding

So, let’s start with building the foundation for Living Documentation by leveraging your source code
to onboard new developers.

### ExDoc

The key to Living Documentation is **Automating Documentation**. We aim to create **Living
Documents** that evolve at the same pace as the project it describes. In Elixir, ExDoc enables
living documents.

Now this may sound incredibly basic, but simply adding the `ex_doc` package, Elixir's documentation
library, to your project will result in some immediate benefits.

You've interacted with the beautiful interface when navigating the Elixir and Phoenix documentation,
leverage this for your in-house projects as well.

### @moduledoc

Often in-house projects will be using Elixir's documentation facilities such as `@moduledoc` and
`@doc`, Elixir’s documentation attributes, but are not actually generating the documentation with
`mix docs`.

Running the documentation generator will actually parse and produce warnings if module references
are not correct.

```
$ mix doc

warning:documentation references file "./" but it does not exist README.md
```

Treat the documentation you write with the same care as you would with your code. On CI, you are
probably running `mix compile --errors-as-warnings` and `mix dialzyer` as steps in your build to
verify your code and typespecs. Add an additional step to verify the project's documentation with
`mix docs`.

This mix task doesn’t return a non-zero status if there are warnings. A little bash magic can fix
this.

```
$ mix compile — errors as warnings
$ mix docs | (! grep warning)
```

### A story from the Field

At the beginning one consulting engagement I was happy to see a consolidated effort to document key
concepts in a project, with references to other modules, but when running `mix docs` there was a
slew of warnings that showed many of these references were broken. Running `mix docs` as part of CI
would have prevented this.

### Publishing ExDoc

Ok, so we have ex_doc installed and running as a part of our CI, but to make this documentation a
first class member of the project, it needs to be published and accessible by all members of the
team.

You may ask yourself, “why publish it when we can read the documentation from the source code?”

As discussed in the section on README-Driven Development, I've discovered you'll learn something
different about the way your application is structured when viewing it via the generated
documentation.  Generating documentation as part of your staging environment and PR review
application allows you to include it the peer review process. Add this item to your PR template
checklist as a reminder for your fellow team members to review.

A simple way to include it in your review application is to generate the documentation, copy it into
the public/static directory and let the static plug serve it. Just make sure you do this based on
the environment and not include it in production.

```elixir
plug Plug.Static,
  at: "/",
  from: :pockets_web,
  gzip: false,
  only: ~w(css fonts images js vendors favicon.ico robots.txt doc)
```

### A story from the Field

When I was first onboarding to a specialized publishing platform there was a comprehensive overview
on their ingestion pipeline. While I was grateful for an overview of the problem and solution
domains, I had some trust issues with the documentation.

Although the documentation was included in the source code repository, it was in a separate
directory called "docs" which was treated as a miscellaneous bucket for domain specific and
operational documentation.  The domain specific documentation was not integrated in the source code
at all.  I would have trusted the documentation if the overview was included as part of the
ingestion module itself, and referenced the actual sub-modules within in the moduledoc, this could
have been validated using `mix docs`.

### The downsides of umbrella Documentation

I suspect most of your in-house applications are probably setup as an umbrella project, a parent
project with multiple child applications located in the `apps` directory. ex_doc with it's support
for umbrella applications can actually lead us down the wrong path. Our immediate instinct is to run
`mix docs` in parent directory of the umbrella project which will generate all the docs in one
parse.

This has several down sides:

1. You're having to run the documentation generator for the entire umbrella project, that is, all
   the child applications which is time consuming.
2. All the modules end up in a single table of contents in the sidebar which can be difficult to
   navigate.
3. You have a single global configuration for your project that is applied to all of the child
   applications. Perhaps for one child application you would like to control the grouping of the
   modules, but for another, you would like to use the default grouping.

### Follow the ecosystem for a better way

Is there a better way? Let's think about our favourite web framework, Phoenix. We don't have a
single Phoenix package, we have multiple packages that provide the functionality for a complete web
framework. The Phoenix framework publishes six packages in total. Sure, each of these packages have
different concerns, allowing us to opt in, but it also makes the curating and publishing of the
documentation easier.

My recommendation is to configure the umbrella project to ignore all apps and then generate the
documentation for each child application individually. We want to use the umbrella project as an
entry point to explore the rest of the application. See the example application on the exact
configuration for this.

```
$ Mix doc #ignore child apps
$ Mix cmd mix docs
```

A template to follow is to have the umbrella project README document the “problem domain” and
provide a catalog of the child applications, our "solution domain".

The problem domain can be described in terms of a *vision, stakeholders* and the *business
capabilities* the application provides.
