---
title: "Controls: ship the floor model with your library"
---

You join an unfamiliar Elixir codebase and open a module you need to use. You run `h MyApp.Registry` in IEx. The docs tell you the function names, the arguments, the return types. They don't tell you the one thing you actually came for: what does it look like to use this correctly?

So you do what every Elixir developer does. You open `test/`.

The tests know. Buried in a `setup` block and three `describe` cases is the canonical way to start the registry, register a process, and look it up. You copy the setup into IEx, peel off the assertions, fix the aliases, and eventually you have something running. You've just reverse-engineered the example you needed from the one place it was hidden.

That instinct, reaching for the tests to learn the code, is correct. The location is wrong.

## The example you needed was real, but locked up

A test holds a working example of how the code is meant to be used. That's most of what a test is. But the test format buries it. Tests don't load in IEx. They return `nil`, not the object you'd want to keep poking at. They resist abstraction, they're invisible to Dialyzer and your language server, and the setup you need is tangled with assertions you don't. The most useful artifact in the file is a live, correct example, and it's the one part you can't reach without copy-pasting.

The Eventide project names the thing that should live there instead. They call it a **control**:

> Libraries, whether utility or applicative, provide canonical examples of the objects contained within. These controls serve as references and affordances to help users understand the intension of the library's implementations, for exercising implementations that use the library, for demo data, or diagnostic uses.

A control is the canonical example of an object, shipped with the library, written to be used.

## The floor model

Think of a control as the floor model in a showroom. Not a spec sheet, not a photo in a catalog -- the actual unit, assembled and plugged in, sitting on the floor where you can switch it on and watch it run. You see how the parts fit before you commit to building your own.

Tests are the floor model locked in the stockroom. Same unit, same working order, but behind a door marked `test/` that customers don't open. Controls move it onto the floor. You put the example where the user is, in `lib/`, and you write it so they can turn it on.

Here's a control for the registry's address, the object the opening story had to dig out of `test/`. It lives at `lib/my_app/controls/registry/address.ex`, the path that mirrors `MyApp.Registry.Address`:

```elixir
defmodule MyApp.Controls.Registry.Address do
  alias MyApp.Registry

  def node_id, do: "some node id"
  def engine, do: :some_engine

  def example do
    Registry.address(node_id(), engine())
  end
end
```

The control builds the address the way the library means it to be built, through `Registry.address/2`. Its values are fixed, so every call returns the same address. And it returns the object, not `nil`.

Call it in IEx and you have a live address to inspect or pass into the next function:

```elixir
iex> MyApp.Controls.Registry.Address.example()
%MyApp.Registry.Address{label: nil, engine: :some_engine, node_id: "some node id"}
```

The fixed values are what let a test build on it. A test in code that depends on the registry takes the control's address, then compares its result with the control's own value:

```elixir
test "routes a message to the address's node" do
  control_address = MyApp.Controls.Registry.Address.example()

  {:remote, node_id} = Router.route(control_address)

  assert node_id == MyApp.Controls.Registry.Address.node_id()
end
```

The test never repeats a literal id. When the control changes, the test follows it.

The Anoma project takes a related approach it calls examples over testing. Its example functions build an object, assert that it's correct, and return it, so the example is also the test. A control leaves the assertion to the test and keeps its values fixed, which is what lets other code build on it.

## What a control does that a test can't

The Eventide definition gives a control two roles and three uses. The roles help a reader understand the library. The uses put the control to work in running code.

As a **reference**, the control shows how the object is meant to be built and what comes back. Not the function signature, but the call.

As an **affordance**, it makes the design intent discoverable. A reader scanning `lib/my_app/controls/` sees which objects the author expects them to build, and what each one looks like.

The first use is **exercising code that uses the library**. A consumer's test takes the control's value instead of hand-rolling its own, as the router test above does. The library states once what a valid address looks like, and every project that depends on it agrees.

The second is **demo data**. A seed script calls the controls and saves the values through the library's own functions, so the demo shows the objects the library calls canonical.

The third is **diagnostics**. When something breaks in production, you run the control through the failing code in a remote console and watch where it fails. It's a known input you already wrote, sitting where you need it.

A test serves the reference role by accident, and the rest not at all.

## Isn't that what doctests are for?

A doctest puts a runnable example next to the function, and for a pure function that's often all the example a reader needs. `Registry.address/2` could carry one.

But the doctest's address exists only inside its `iex>` block. The router test can't call it, a seed script can't call it, and neither can a remote console in production. A doctest shows the reader the object. A control hands the object to the reader's code.

The two don't compete. The doctest shows the call, and the control is the value everything else builds on.

## The trade-offs, honestly

Controls are not free, and a few of the costs are real.

They ship in your `lib/`, which means they compile into your application and travel to production. For most projects that's a rounding error. If you're shipping a constrained artifact, it's a line item you'll want to look at.

And controls are code. They need maintenance like any other code. An example that has drifted from the library is worse than no example, because it lies with the authority of something that compiles.

This isn't an argument to delete your test suite. ExUnit still owns coverage, edge cases, regression, and CI. Controls own the canonical example -- the happy-path demonstration of how an object is meant to be used. The two overlap, and that overlap is fine. When you get back to the office on Monday, you shouldn't start moving every test into `lib/my_app/controls/`. You should pick one.

## Where to start

Find the module in your codebase that people ask about most -- the one whose tests get opened not to run them but to read them. Write a single control for it. One `example/0` that builds the object the canonical way, from fixed values, and returns it for the next person to use.

Then the next time someone runs `h` on that module and comes up short, they'll have somewhere better to go than `test/`. The floor model will be out where they can switch it on.

---

*Controls are a design value from the [Eventide project](https://eventide-project.org/). Anoma's [examples-over-testing](https://github.com/anoma/anoma/blob/base/documentation/contributing/examples-over-testing.livemd) guide describes the related approach.*
