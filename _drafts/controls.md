---
title: Controls: ship the floor model with your library
---

# Controls: ship the floor model with your library

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

The Anoma project does exactly this. Their codebase carries a whole `lib/examples/` tree alongside the implementation. Here's a trimmed control from their registry examples:

```elixir
defmodule Anoma.Node.Examples.ERegistry do
  alias Anoma.Node.Registry
  alias Anoma.Node.Registry.Address

  import ExUnit.Assertions

  @doc """
  I create an address for a given node id and module.
  I assert that the address is created correctly.
  """
  def create_address() do
    node_id = "londo_mollari" <> Base.url_encode64(:crypto.strong_rand_bytes(16))

    address = Registry.address(node_id, :module)

    assert address == %Address{label: nil, engine: :module, node_id: node_id}

    address
  end
end
```

Read what that function does. It builds a real address the canonical way. It asserts the result is what the library promises. Then it returns the address -- a useful object, not `nil`.

That return value is the whole point. Call `ERegistry.create_address()` in IEx and you get a live `Address` struct to inspect, pass into the next function, or use as a fixture. The same function that proves the code is correct also hands you something to build with. Anoma's contributor guide leans on this directly: to understand a module, you open its example file and run it line by line, watching the real data move through `:observer`.

## What a control does that a test can't

A control earns its place in `lib/` by doing four jobs at once. The Eventide definition lists them, and they map cleanly onto the daily friction of working in someone else's code.

**Reference.** It shows the intended usage pattern in executable form. Not "here's the function signature" but "here's how the function is meant to be called, and what comes back."

**Affordance.** It makes the design intent discoverable. A reader scanning `lib/examples/` sees the shape of the library the author had in mind, the same way a handle tells you a door is meant to be pulled.

**Fixture.** Because it returns a useful object, code that depends on the library can call the control to get realistic data instead of hand-rolling it. Your example becomes your test data.

**Diagnostic.** When something breaks in production, you run the control in a console and watch where it fails. A smoke test you already wrote, sitting where you need it.

A test does the first of these accidentally and the rest not at all.

## The trade-offs, honestly

Controls are not free, and a few of the costs are real.

They ship in your `lib/`, which means they compile into your application and travel to production. For most projects that's a rounding error. If you're shipping a constrained artifact, it's a line item you'll want to look at.

That example above imports `ExUnit.Assertions` into a library module. Pulling a test framework into `lib/` reads as strange the first time you see it, and it should -- you're deciding that the assertion belongs next to the code it describes rather than across the `test/` boundary. It's a defensible call, but it's a call.

And controls are code. They need maintenance like any other code. An example that has drifted from the library is worse than no example, because it lies with the authority of something that compiles.

This isn't an argument to delete your test suite. ExUnit still owns coverage, edge cases, regression, and CI. Controls own the canonical example -- the happy-path demonstration of how an object is meant to be used. The two overlap, and that overlap is fine. When you get back to the office on Monday, you shouldn't start moving every test into `lib/examples/`. You should pick one.

## Where to start

Find the module in your codebase that people ask about most -- the one whose tests get opened not to run them but to read them. Write a single control for it. One function that builds the object the canonical way, asserts it's correct, and returns it for the next person to use.

Then the next time someone runs `h` on that module and comes up short, they'll have somewhere better to go than `test/`. The floor model will be out where they can switch it on.

---

*Controls are a design value from the [Eventide project](https://eventide-project.org/). The examples here draw on Anoma's [examples-over-testing](https://github.com/anoma/anoma/blob/base/documentation/contributing/examples-over-testing.livemd) approach and their [registry example module](https://github.com/anoma/anoma/blob/base/lib/examples/e_registry.ex).*
