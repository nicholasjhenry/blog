---
title: "Stub Queries, Expect Commands: Why GWT Collapses to Given/Then in ExUnit"
---

# Stub Queries, Expect Commands: Why GWT Collapses to Given/Then in ExUnit

I'd been writing ExUnit tests in Given/When/Then for a while. `describe` carried "when X," the test string carried "given Y, then Z," and the structure stayed consistent across the suite. But every test description was a small paragraph, and the verbosity bothered me.

Tracing it the other day exposed something structural. Per Rose and Nagy's framing in *Effective Behavior-Driven Development*, **Given declares state**, **When is the action**, and **Then is the outcome**. The three labels are right; the slots underneath them stop working at the unit level. Once you see why, the test structure rearranges itself, and the rearrangement also lines up with the CQS distinction that Mox's `stub` and `expect` already encode.

This post walks the collapse, then the alignment, then how to extend the same idea to a service you don't own.

## Three slots, two nesting points

Here's a representative test from the suite as it was:

```elixir
describe "when creating a user" do
  test "given the email is not yet registered, then returns {:ok, user}" do
    assert {:ok, _user} = Accounts.create_user(valid_attrs())
  end
end
```

The labels match the slots. "Creating a user" is the action under **When**. "The email is not yet registered" is state under **Given**. "Returns `{:ok, user}`" is the outcome under **Then**. Strict GWT, applied consistently across the suite.

It also reads as a small paragraph per test. The verbosity is structural. GWT has three slots: precondition, action, outcome. ExUnit gives me two nesting points: `describe` and `test`. Three into two means something has to double up. I'd been doubling up by cramming Given and Then into the test string while `describe` held When. The structure works, but most of those words are carrying scaffolding rather than what makes any individual test distinct.

## The "when" is already pinned

There's a way out, and it falls out of looking at what actually varies. The action in every test under that `describe` is the same function call: `Accounts.create_user/1`. The When doesn't vary across the block. **It's pinned by the function under test**, and a pinned action doesn't need the "when" keyword to mark it — it can sit in `describe` as the noun phrase the block is about, leaving room for the Given alongside it.

So the structure that fits keeps the When in `describe`, sets the Given next to it, and lets `test` carry the Then:

```elixir
describe "creating a user given the email is not yet registered" do
  setup do
    {:ok, attrs: valid_attrs()}
  end

  test "returns the persisted user", %{attrs: attrs} do
    assert {:ok, user} = Accounts.create_user(attrs)
    assert user.id
  end
end
```

Three things moved. The When stayed in `describe` and dropped its keyword, because at the unit level it never varies. The Given joined it there as the state class, with the actual precondition data moving into `setup`. The Then collapsed to the `test` description.

GWT was built for acceptance tests where the action varies. At the unit level it doesn't, and the third slot disappears. What's left is Given/Then.

## Granularity: describe per state class, test per behaviour

The first worry I had once I saw this was that I'd end up with one `describe` and one `test` per assertion. That happens if you collapse three different granularities into one. Keep them distinct and the counts stop being 1:1:1:

- a physical `assert` is the smallest unit, and several are fine in one test
- a `test` is one behaviour, which may take several asserts to pin down
- a `describe` is one state class, shared through `setup`

```elixir
describe "creating a user given the email is not yet registered" do
  setup do
    {:ok, attrs: valid_attrs()}
  end

  test "persists the user", %{attrs: attrs} do
    assert {:ok, user} = Accounts.create_user(attrs)
    assert user.id
    assert Repo.get(User, user.id)
  end

  test "hashes the password", %{attrs: attrs} do
    assert {:ok, user} = Accounts.create_user(attrs)
    refute user.password_hash == attrs.password
  end
end
```

One given, two behaviours, five assertions. Not five blocks. The block count tracks real complexity rather than ceremony.

The corollary: `describe` should group by the *class* of state, not each individual state. "given the email is not yet registered" and "given the email is already registered" are two describes. The individual error cases (different invalid fields, different malformed inputs) don't each earn a describe; they're rows in a table.

## Where mocks fit: stub for state, expect for outcomes

Mocking slots into this cleanly once you notice that a mock has two faces, and they belong on opposite sides of the given/then split.

- the **stubbed return** (`the mailer succeeds`) is part of the world the code runs in. That's a given. It goes in `setup`.
- the **verification** (`the mailer was called with this address`) is an outcome. That's a then. It goes in the test body.

Mox's two primitives encode this exactly. `stub` describes state. `expect` makes a claim. The discipline is to use them deliberately. Imagine `create_user/1` also sends a welcome email through an injected `Mailer` behaviour:

```elixir
describe "creating a user given the email is not yet registered" do
  setup do
    # GIVEN: the world. Mailer just works -- a stub, not an assertion.
    stub(MailerMock, :deliver, fn _email -> {:ok, %{}} end)
    {:ok, attrs: valid_attrs()}
  end

  test "persists the user", %{attrs: attrs} do
    assert {:ok, user} = Accounts.create_user(attrs)
    assert Repo.get(User, user.id)
  end

  test "sends a welcome email to the new user", %{attrs: attrs} do
    # THEN: this test is *about* the mailer, so the expectation lives here.
    expect(MailerMock, :deliver, fn email ->
      assert email.to == attrs.email
      {:ok, %{}}
    end)

    assert {:ok, _user} = Accounts.create_user(attrs)
  end
end
```

The "persists the user" test doesn't care about email, so it leans on the setup stub and stays silent about it. The "sends a welcome email" test is about that interaction, so it overrides with `expect` right where the behaviour is named.

## Why the split aligns with command/query separation

There's a deeper symmetry under that pattern, and it's the principle that keeps it consistent across new tests.

With a query you depend on the *answer*, not the *occurrence*. Your code asked billing how many seats remain; what matters is what it does with the number. The call itself is incidental, because a refactor might cache it, call it twice, or swap it for an equivalent query, all without changing observable behaviour. Verifying the call would couple the test to implementation.

With a command the reverse: you depend on the *occurrence*, not the *answer*. The mailer's `{:ok, %{}}` is meaningless. The side effect is invisible to the rest of the system, so the only observable behaviour is the interaction itself. Verifying it is the only specification available.

```
query:   return is essential, call is incidental  ->  stub   -> given
command: call is essential, return is incidental  ->  expect -> then
```

Two failure signals fall out of this. If I reach for `expect` on a query, proving the code "called billing," that's the over-specification smell; the consultation is proven through its consequence, so assert the outcome instead. If a dependency forces both stubbing a return and expecting the call, that's a command-query hybrid, a CQS violation in the dependency itself. The testing awkwardness is the design smell surfacing. The fix is usually upstream, splitting the dependency, not in the test.

## A boundary for what I don't own

The pattern needs one more piece to extend past code I wrote. Take Stripe. I want to test code that asks Stripe for the org's invoices and rolls them up into a statement. I don't want every domain test hitting Stripe, and I can't substitute `stripity_stripe` (or any other third-party client) with Mox anyway, because there's no behaviour of mine to swap.

Even if I could, stubbing the Stripe client directly would couple my domain tests to Stripe's wire format: HTTP endpoints, JSON shapes, idempotency keys, pagination cursors. That's the over-specification smell relocated to the infrastructure layer.

The move that keeps everything consistent: don't mock Stripe. Mock a boundary I own that happens to be implemented by Stripe.

There are two distinct things people conflate when they say "mock Stripe":

- the **semantic query** ("give me this org's invoices for the current period," expressed in my domain language)
- the **transport** (the HTTP endpoints, JSON decoding, idempotency keys, API authentication)

I want a seam at the first, not the second. Define a behaviour that speaks the domain. Let a real adapter translate it into Stripe. Let Mox substitute the behaviour.

```elixir
# The port -- my language, no Stripe vocabulary leaks through.
defmodule MyApp.InvoiceSource do
  @callback invoices_for(org_id :: String.t(), period :: Date.Range.t()) ::
              {:ok, [Invoice.t()]} | {:error, term()}
end

# The real adapter -- the only place Stripe exists.
defmodule MyApp.InvoiceSource.Stripe do
  @behaviour MyApp.InvoiceSource

  @impl true
  def invoices_for(org_id, period) do
    # call Stripe.Invoice.list/2 via stripity_stripe, decode results...
  end
end
```

Wire it through config so test swaps the mock in:

```elixir
# config/test.exs
config :my_app, :invoice_source, InvoiceSourceMock

# config/prod.exs
config :my_app, :invoice_source, MyApp.InvoiceSource.Stripe
```

Now domain tests treat the port like any other query. Its return is given-state, stubbed in setup, never expected.

```elixir
describe "preparing a billing statement given the org has outstanding invoices" do
  setup do
    stub(InvoiceSourceMock, :invoices_for, fn _org, _period ->
      {:ok, [%Invoice{number: "INV-2026-05-001", amount_due: 12_000}]}
    end)

    {:ok, org: org_fixture()}
  end

  test "summarises total amount due", %{org: org} do
    assert {:ok, statement} = Billing.statement_for(org)
    assert statement.total_due == 12_000
  end
end

describe "preparing a billing statement given the billing source is unavailable" do
  setup do
    stub(InvoiceSourceMock, :invoices_for, fn _org, _period -> {:error, :timeout} end)
    {:ok, org: org_fixture()}
  end

  test "returns a degraded statement", %{org: org} do
    assert {:ok, %{status: :degraded}} = Billing.statement_for(org)
  end
end
```

The two failure modes I most want from a query I don't own (it returns data, it falls over) become two state classes differentiated entirely by the stubbed return. The `{:error, :timeout}` branch is the real prize. Simulating a flaky external service through a stub is trivial. Reproducing the same failure against real Stripe (network timeouts, rate limiting, partial responses) is fragile and slow.

What this deliberately doesn't test is whether my adapter's API calls are correct, or whether Stripe's responses actually decode into `Invoice` the way I assumed. That's a separate contract test against Stripe's test mode, tagged so it stays out of the fast suite:

```elixir
@moduletag :integration
```

Mock the abstraction over the third party for the bulk of tests. Write one narrow integration test that pins the adapter's assumptions to reality. If Stripe changes its API, that integration test fails, not three hundred domain tests that had no business knowing Stripe existed.

## The template

The shape that fell out of all of this:

```elixir
defmodule MyExampleTest do
  use ExUnit.Case, async: true
  import Mox

  setup :verify_on_exit!   # checks every expect/2 actually fired
  setup :defaults

  defp defaults(ctx) do
    default_ctx = %{key: :value}
    Enum.into(ctx, default_ctx)
  end

  # GIVEN (the WHEN is the action carried by the function under test)
  describe "<the action in domain language> given <some state>" do
    setup [:stub_dependency_1, :stub_dependency_2]

    # THEN: outcomes derived from the stubbed query answers
    test "some outcome of the subject under test" do
      assert {:ok, _} = Subject.my_function(:value)
    end

    test "some other outcome of the subject under test" do
      assert {:ok, _} = Subject.my_function(:value)
    end

    # THEN: a command -- the interaction IS the behaviour, so expect lives here
    test "expected behaviour of dependency 1", _ctx do
      expect(Dependency1, :my_function, fn arg ->
        assert arg == :value
        {:ok, arg}
      end)

      assert {:ok, _} = Subject.my_function(:value)
    end

    test "expected behaviour of dependency 2", _ctx do
      expect(Dependency2, :my_function, fn arg ->
        assert arg == :value
        {:ok, arg}
      end)

      assert {:ok, _} = Subject.my_function(:value)
    end
  end

  defp stub_dependency_1(_ctx) do
    stub(Dependency1, :my_function, fn arg -> {:ok, arg} end)
    :ok
  end

  defp stub_dependency_2(_ctx) do
    stub(Dependency2, :my_function, fn arg -> {:ok, arg} end)
    :ok
  end
end
```

Two pieces of plumbing matter. `verify_on_exit!` checks that every `expect/2` actually fired. Without it, an expect that never runs is never checked, and the whole point of `expect` over `stub` evaporates silently. And the dependency has to be injectable (a behaviour resolved through config), or there's nothing for Mox to substitute.

One gotcha worth flagging. `async: true` with Mox in private mode means the stub is owned by the test process. If the subject delegates the dependency call into a spawned `Task` or `GenServer`, that process won't see the stub and Mox will raise. When that happens, reach for `setup :set_mox_from_context` plus `Mox.allow/3` rather than global mode. Global mode would force `async: false` and cost the parallelism.

## What you can do now

Before this collapse, my ExUnit tests carried structure as English. "When X, given Y, then Z" stretched across `describe` and `test` strings, with stubs and expects scattered without a principle for which went where.

Now the structure lives in the shape of the test. `describe` names the action and the state class, in domain language. `setup` carries the Given as data, with `stub` for the query returns the code runs against. `test` names the outcome, and uses `expect` only when the test is about a command's firing.

You don't need to rip a suite apart to try this. Pick a test module you're already maintaining. Find a `describe` whose first word is "when," and rewrite it as `"<action> given <state>"`, dropping the keywords. See whether the prose gets shorter and the test names start reading like outcomes. If there's an `expect` in `setup`, look at it next; it's likely an assertion smuggled into a precondition, and moving it into the test body is the same kind of refactor.
