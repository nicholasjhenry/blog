---
title: dbg/1 adds a new reason to use `with` more often
---

Wrap `dbg()` around a `with` chain and Elixir doesn't return one value. It walks the chain and prints every clause beside its matched value, with a marker on the one that failed.

That behavior is the operational case for an argument Dave Thomas made in 2016. In [Over-using `with` in Elixir 1.2](https://pragdave.github.io/blog/2016/02/23/over-using-with-in-elixir-1-dot-2.html), his thesis was: reach for `with` *more* than strictly necessary, not just for chained `{:ok, _}` returns. Every time you create a function-level local, try fitting it into a `with`. The pressure to do so, he wrote, "drives me to create simpler, single-purpose functions."

Dave's case was about code quality on its own terms. `dbg/1` shipped in Elixir 1.14, six years later, and adds a second reason.

## Each `with` clause is its own debug label

Here's a transfer flow. One `=` for an idempotency key, then four `<-` steps that can fail:

```elixir
def transfer(from_id, to_id, amount) do
  with idempotency_key = build_key(from_id, to_id, amount),
       {:ok, source} <- get_account(from_id),
       {:ok, dest} <- get_account(to_id),
       :ok <- check_sufficient_funds(source, amount),
       {:ok, txn} <- execute_transfer(source, dest, amount, idempotency_key) do
    {:ok, txn}
  end
end
```

To see why this fails for a given input, pipe it into `dbg`:

```elixir
def transfer(from_id, to_id, amount) do
  with idempotency_key = build_key(from_id, to_id, amount),
       {:ok, source} <- get_account(from_id),
       {:ok, dest} <- get_account(to_id),
       :ok <- check_sufficient_funds(source, amount),
       {:ok, txn} <- execute_transfer(source, dest, amount, idempotency_key) do
    {:ok, txn}
  end
  |> dbg()
end
```

`dbg` walks the chain and prints each clause's call beside the value it produced:

```
[lib/wealth/transfers.ex:12: Wealth.Transfers.transfer/3]
With clauses:
build_key(from_id, to_id, amount) #=> "txn:1:2:9999"
get_account(from_id) #=> {:ok, %{id: 1, balance: 5000}}
get_account(to_id) #=> {:ok, %{id: 2, balance: 0}}
check_sufficient_funds(source, amount) #=> {:error, :insufficient_funds}

With expression:
with idempotency_key = build_key(from_id, to_id, amount),
     {:ok, source} <- get_account(from_id),
     {:ok, dest} <- get_account(to_id),
     :ok <- check_sufficient_funds(source, amount),
     {:ok, txn} <- execute_transfer(source, dest, amount, idempotency_key) do
  {:ok, txn}
end #=> {:error, :insufficient_funds}
```

The `=` clause shows up in the walk too. `dbg` doesn't care whether a clause is `<-` or `=`; every clause in the chain gets printed beside its value, and the chain stops at the first one that fails to match.

No `IO.inspect/2` calls sprinkled through the function. No labels to invent. The call expression in each clause is its own label. The chain `dbg` shows is the chain you wrote.

## The missing half of Dave's argument

Write the same transfer with `=` assignments and a `case` at the end, and the same `dbg()` call gives you one value: the final result. To trace the chain you'd scatter inspect calls through the body, invent labels, then remember to take them out.

The `with` version costs nothing extra to write and hands you a step trace for free. The structure that makes the code readable is the structure `dbg` understands.

That doesn't change the trade-offs Dave's critics raised at the time — a single `else` block can flatten errors awkwardly, and not every chain wants pattern-match semantics. It does shift the cost-benefit on the cases where `with` is a judgment call.

If you take Dave's wider advice, Credo's [`Refactor.WithClauses`](https://credo.hexdocs.pm/Credo.Check.Refactor.WithClauses.html) check will fight you. It flags `with` blocks whose first or last clause is a plain `=` instead of `<-` — the exact pattern Dave was advocating for in his 2016 example. `dbg` walks those `=` clauses too, so the operational case for using `with` widely covers them as well. My recommendation: disable the check.

## Try this

Find a function with two or three `=` assignments leading into a `case`. Rewrite it as a `with`. Pipe the result into `dbg()` and run it against the input you want to trace. See whether the output tells you what you need without a single inspect call.
