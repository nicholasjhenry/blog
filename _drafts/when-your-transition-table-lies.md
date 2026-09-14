# When Your Transition Table Lies

Most workflow bugs I've chased down were not really bugs in the code. They
were bugs in a data structure that quietly claimed to say more than it
could. This is a post about one of those data structures — a table of
"which things can follow which other things" — and about what happens when
you follow the thread of *what is this actually specifying?* all the way
down.

The domain here is a conference talk proposal moving through review. I
picked it because if you've submitted to a CFP you already know the rules,
so I can spend the words on the design instead of the domain. But the shape
is completely generic: anything that moves through stages, where what's
allowed next depends on what happened before.

## The starting point

Say you have a few kinds of things that can happen to a proposal, and you
want to describe which can follow which. The obvious first move is a map
from each type to the ones that may follow it. When I've seen this in the
wild, it often carries both directions at once — what can precede a thing
*and* what can follow it:

```elixir
%{
  Submitted => %{from: [],          to: [Revised, Reviewed, Decided]},
  Revised   => %{from: [Submitted, Revised, Reviewed], to: [Revised, Reviewed, Decided]},
  Reviewed  => %{from: [Submitted, Revised, Reviewed], to: [Revised, Reviewed, Decided]},
  Decided   => %{from: [Submitted, Revised, Reviewed], to: []}
}
```

The first problem is visible if you stare at it: every edge is written
twice. `Submitted -> Reviewed` shows up in `Submitted`'s `to` list and
again in `Reviewed`'s `from` list. Two copies of one fact is two things
that can disagree, and the day someone edits one list and not the other,
the table starts lying. The `from` direction is entirely derivable from the
`to` direction anyway — it's a query, not stored state — so the fix is to
drop it and keep one direction:

```elixir
@transitions %{
  Submitted => [Revised, Reviewed, Decided],
  Revised   => [Revised, Reviewed, Decided],
  Reviewed  => [Revised, Reviewed, Decided],
  Decided   => []
}
```

One source of truth. If you ever need predecessors, you compute them.

## "Is this underspecified?"

Here's where it gets interesting, and it's the question that drove the rest
of the design. That map is *data*, but it's data with an implicit contract
that nothing enforces. Nothing checks that every type appearing in a `to`
list is itself a key. Nothing stops a typo'd key. The default you'll
inevitably reach for on lookup —

```elixir
def can_follow?(a, b), do: b in Map.get(@transitions, a, [])
```

— has that `[]` doing load-bearing work: an unknown type silently answers
"nothing may follow," which is indistinguishable from a real dead end. A
missing entry and a genuine terminal look the same.

Because this is a module attribute, you can close every one of those gaps
*at compile time*. This is the first real theme of the post: **data plus
compile-time invariants is a much tighter specification than data alone,
and the invariants are where the design decisions actually live.**

```elixir
@types Map.keys(@transitions)
@targets @transitions |> Map.values() |> List.flatten() |> Enum.uniq()

# every target must be a declared type — catches the drift bug structurally
case @targets -- @types do
  [] -> :ok
  orphans -> raise CompileError, description: "targets missing from graph: #{inspect(orphans)}"
end
```

And swap the tolerant lookup for a strict one, so an unknown type is a loud
crash instead of a quiet `false`:

```elixir
def can_follow?(a, b), do: b in Map.fetch!(@transitions, a)
```

The map wasn't so much underspecified as *unaccompanied*. Give it a closure
check and a strict lookup and it now means exactly what it says.

## The underspecification that mattered more

There's a second sense of "underspecified," and this is the one that
actually bites in production. It's not about the transition table. It's
about what you're storing as the record of what happened.

The version I've seen — and written — stores the trail as a list of type
atoms:

```elixir
%{types_seen: [Submitted, Reviewed, Reviewed]}
```

That looks reasonable until a rule arrives that the list can't answer. Ours
is going to be: **once the program chair has reviewed a proposal, ordinary
committee reviews no longer count — only the chair may add further
reviews.** Now "was this review by the chair or a committee member?" is a
question your stored data has already thrown away. Every record written
before the rule existed dropped the author, because at write time nobody
knew author mattered.

That is the real meaning of underspecified here: not vague, but *prematurely
projected*. The list committed to "the type sequence is all that will ever
matter" by omission, and that commitment is irreversible — no migration
recovers a fact you never captured.

The lesson generalizes into a rule of thumb I now reach for constantly:

> Spend your specification effort in inverse proportion to your ability to
> change the thing later. Compile-checked code costs nothing to change, so
> specify it loosely and tighten as you go. Persisted history you can never
> re-derive, so store the *facts* at full fidelity, because "captured it and
> never used it" is cheap and "needed it and never captured it" is a
> data-archaeology project.

## Two artifacts, not one

The fix splits one confused thing into two things with clear jobs.

**The log** is the append-only record of events, stored at full fidelity.
Full maps, not type atoms — author, timestamp, payload, everything. This is
the source of truth and the one artifact whose schema is expensive to get
wrong.

**The facts** are a small struct of *named* booleans (and counts, and
whatever else) that the rules actually care about, computed by folding over
the log. The fields of this struct are a precise, reviewable answer to the
question "what about the past matters?"

```elixir
defmodule Facts do
  defstruct submitted?: false, decided?: false, chair_reviewed?: false
end

def advance(%Facts{} = f, %{type: Submitted}), do: %{f | submitted?: true}
def advance(%Facts{} = f, %{type: Decided}),   do: %{f | decided?: true}
def advance(%Facts{} = f, %{type: Reviewed, author_role: :chair}),
  do: %{f | chair_reviewed?: true}
def advance(%Facts{} = f, _event), do: f

def replay(log), do: Enum.reduce(log, %Facts{}, &advance(&2, &1))
```

The facts are a *cache* of the fold; you can rebuild them from the log at
any time. When rules change, you change the fold and re-derive — old
records need no migration, because the fold ignoring a fact yesterday
didn't discard it. The list was failing at *both* jobs: too lossy to be a
log, too unstructured (an unbounded list of atoms) to be good state. Split
them and the problem dissolves.

The temptation to resist is passing the raw log around and writing
predicates over it — `Enum.any?(log, &chair_review?/1)` scattered wherever a
rule needs it. Fold the log into named facts *once*, and route on the
facts. That keeps the rules readable and keeps "what matters" in one place
instead of smeared across `Enum` calls.

## Why the naive table can't be rescued

At this point a reasonable person asks: can't I keep the rule in the table?
Just make the key richer — split `Reviewed` into `{Reviewed, :chair}` and
`{Reviewed, :reviewer}` and encode the restriction there?

It doesn't work, and *why* it doesn't work is the heart of the matter. The
table keys on the previous event. It has exactly one event of memory. But
the chair rule is not about the previous event — it's about whether a chair
review has *ever* happened. Watch it fail:

```
chair reviews  ->  speaker revises  ->  committee member reviews
```

Check each adjacent pair. Chair review followed by a revision? Allowed.
Revision followed by a committee review? Allowed — because by the time we're
looking at the revision, the one-event-memory table has completely forgotten
a chair ever reviewed. Every pair is individually legal, and the sequence as
a whole violates the rule. The table only blocks a committee review
*immediately* after a chair review. The requirement blocks it *forever*
after.

You can force it to work, but only by threading the historical fact through
every single key — `{event, chair_has_reviewed?}` — at which point you've
doubled the table and reinvented the fold, just smeared across the key space
instead of named in a struct. Any rule containing the words *once*, *ever*,
*no longer*, or *only after* is telling you the same thing: this fact
belongs in the folded state, not in the previous-event key.

This is the part I find worth sitting with, because it's a specific,
mechanical instance of a general trap: **every pairwise check passes, and
the design is still broken.** No amount of example-based testing of
adjacent pairs will ever surface it. You need a test that generates whole
sequences.

## The routing decision, factored

With facts in hand, "can this event happen now?" becomes a lookup keyed on a
small projection of the facts rather than on the previous event:

```elixir
def allowed?(%Facts{} = facts, event) do
  event_key(event) in Map.fetch!(@allowed, phase(facts))
end
```

`phase/1` collapses a `Facts` value into which stage of life the proposal is
in; `@allowed` is a flat map from phase to the events permitted there.
Both stay dumb data:

```elixir
defp phase(%Facts{submitted?: false}),      do: :unsubmitted
defp phase(%Facts{decided?: true}),          do: :concluded
defp phase(%Facts{chair_reviewed?: true}),   do: :review_closed
defp phase(%Facts{}),                        do: :review_open

# Read each row as: "while in <phase>, <these events> may occur."
@allowed %{
  unsubmitted:   [:submitted],
  review_open:   [:revised, {:reviewed, :reviewer}, {:reviewed, :chair}, :decided],
  review_closed: [:revised, {:reviewed, :chair}, :decided],
  concluded:     []
}
```

The chair rule now lives in one place, as data: `review_closed` simply
doesn't list `{:reviewed, :reviewer}`. And the same compile-time discipline
from earlier applies — assert every phase has a row, every target is a
known event key, `:concluded` is terminal, and (a check I only added after
getting confused myself) that the phase vocabulary and the event vocabulary
share no atoms, so you can never misread one as the other.

### Why two projections and not one

The sharp objection here is: why fold the log into facts and *then* project
facts into a phase? Why not fold straight to a phase and have a plain
four-state machine?

Because the two projections change for different reasons, and fusing them
couples those reasons. The fold (`advance/2`) is *mechanism* — "which events
establish which facts about the world" — and those are near-permanent domain
truths. The phase map is *policy* — "how does current routing carve up those
facts." When the committee changes a rule, that's a policy edit: you change
`phase/1` or the table and recompute from existing facts, no replay needed.
Fold straight to phase and every policy change becomes a change to the fold
itself, invalidating anything derived from the old one.

There's also a subtler reason. The phase set is *minimal* — the smallest
state that suffices for today's rules. Minimal states are brittle: add a
rule referencing a distinction the phases discarded (say, "at most three
revisions") and the phase set has to split, but a stored phase can't be
split without replaying, because the distinguishing information is gone.
Facts are deliberately non-minimal; that slack is the buffer that makes new
rules cheap. It's the `types_seen` lesson again, one level up: folding
straight to phase is a premature projection of the *derived* state.

The honest caveat: when the machine genuinely *is* the domain — a protocol
like a TCP connection or an OAuth flow, where the states are the
specification — the facts layer is ceremony and a direct state machine is
clearer. The test is whether your phases are *definitional* (the domain is
specified as these states) or *derived* (the domain is specified as events
and rules, and phases are a convenient summary). CFP review is the second
kind. Nobody specifies "review_closed" as a primitive; they say "the chair
reviewed it, so committee reviews stop counting" — a fact and a policy.

## The payoff: the model tests itself

Here's the part that makes all of it click. Look at what we built:

- `%Facts{}` — a state
- `allowed?/2` — a precondition on that state
- `advance/2` — a transition function on that state

That is exactly the triple a stateful property test wants. In PropCheck's
`StateM`, the model *is* your `Facts`, the precondition *is* `allowed?`, the
transition *is* `advance` — almost verbatim:

```elixir
def initial_state, do: %Facts{}
def precondition(facts, {:call, _, :try, [event]}), do: Routing.allowed?(facts, event)
def next_state(facts, _r, {:call, _, :try, [event]}), do: Routing.advance(facts, event)

def postcondition(facts, {:call, _, :try, [event]}, _result) do
  # the property the pairwise table could never enforce:
  not (facts.chair_reviewed? and
       match?(%{type: Reviewed, author_role: :reviewer}, event))
end
```

The property "no committee review is ever accepted after a chair review"
generates whole sequences and would produce the length-3 counterexample on
its first run. And notice: the model needed no separate implementation. If
it had — if the test's model diverged from `Facts` — that divergence would
itself be the signal that `Facts` is missing a fact. The design and its test
are the same shape because they're derived from the same source.

## The vocabulary was half the work

I'll be honest about how much of this was naming, because I think that's
underrated. The design didn't stabilize until the words did. It went:

- **signal → event.** Once there was an append-only log and a fold and a
  replay, we had reinvented event sourcing, so the elements are *events*.
  The pattern supplied the noun.
- **state → facts.** The process already owned "state" (it's a GenServer);
  the domain projection needed its own name, and "facts folded from the log"
  is what it is.
- **facts_key → phase.** Both sides of the table were atoms, and atoms don't
  know which domain they belong to. Giving phases noun-names and events
  past-tense-verb-names (plus a compile-time check that the two never
  overlap) made every key self-describing.
- **history → log.** "History" was doing double duty — the abstract concept
  *and* the list field. Renaming the field to `log` freed "history" to mean
  only the concept.

Every one of those renames was forced by a real confusion, and every
resolution came from letting the underlying pattern name things rather than
inventing neutral labels. That's ubiquitous language emerging *from* the
refactor instead of being imposed before it — the reverse of how the DDD
books usually tell the story, and in my experience the way it actually
happens.

## Is this too much for a four-event workflow?

If you're picturing the reaction from your team, you're right to. The full
thing — the log, the fold, the phase layer, the compile-time checks, the
handling of events from unknown future versions — is a lot for something
that started as a fifteen-line map. So be clear about what's load-bearing
and what's earned later.

The **core** is a struct with a few booleans, `allowed?/2`, `advance/2`, and
`replay/1`. Thirty lines. Nobody calls that an architecture. Everything else
is severable hardening, each piece with a trigger:

- compile-time table checks — when the table is big enough to fat-finger
- the phase indirection — when a second consumer of the facts appears, or
  the table outgrows function heads
- unknown-event handling — when you actually run mixed versions in a cluster
- the fold — the day a rule says *once* / *ever* / *no longer after*

And crucially: compare against the *honest* baseline. The pushback will be
"the old map was fifteen lines," but the old map couldn't express the chair
rule at all. The real alternative your teammates would write is that rule as
an `if` scanning the history inside the handler — same logic, facts unnamed,
scan repeated per rule, second rule landing somewhere else in the file. The
fold is the *compression* of that, not an elaboration.

So lead with the failing test, not the pattern. "Here's a sequence the rules
forbid, here's the current code accepting it, here's the smallest change
that fixes it" is a bug fix nobody argues with. "I'd like to introduce an
event-sourcing-inspired routing layer" is an architecture debate you don't
need to have. The design survives translation into boring words —
"we track a couple of booleans about what's happened and check the rule
against those" — completely intact. If a design only sounds justified in
pattern-language, that's a tell. This one passes.

And keep the honest exit visible: if your real rules genuinely are pairwise,
with no *once*/*ever* requirements now or coming, the simple table is
*correct* and the fold is overkill. It isn't a better default. It's the
answer to a specific class of requirement — the class where every pairwise
check passes and the design is still broken.
