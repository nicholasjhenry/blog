---
title: "DCI in Elixir: Giving Your Data Roles to Play"
meta_description: "Exploring how DCI (Data-Context-Interaction) architecture maps to Elixir, using a farm-to-table delivery scheduling example to separate what data is from what it does in a given use case."
tags: "elixir, architecture, dci, domain-modeling, phoenix"
---

# DCI in Elixir: Giving Your Data Roles to Play

I keep running into the same problem in Phoenix applications. A schema starts lean, then accumulates functions from every use case it participates in. `SeasonalContract` grows a `record_commitment/2` for deliveries, a `flag_for_audit/1` for compliance, a `summarize_for_invoice/1` for billing. The module becomes a junk drawer of behaviors that have nothing in common except the data type they operate on.

That pressure reminded me of something I hadn't thought about in a while. Years ago I read James Coplien and Gertrud Bjørnvig's [*Lean Architecture*](https://books.google.ca/books/about/Lean_Architecture.html?id=gUWhCwAAQBAJ), which introduced me to DCI — Data-Context-Interaction — an architectural paradigm by Trygve Reenskaug (the same person who gave us MVC). The core idea is to separate code into three perspectives: **Data** (how information is represented), **Context** (the runtime scenario assembling objects for a use case), and **Interaction** (the roles those objects play within that scenario).

There's something fun about an idea that lodges in your head for years and then resurfaces when you're working in a completely different language. I'd been thinking about [DCI in the context of Ruby](https://www.saturnflyer.com/clean-ruby) back then. Now I'm curious how it maps to Elixir — and it maps more naturally than I expected.

## The Mapping

| DCI Perspective      | Elixir Equivalent                        |
|----------------------|------------------------------------------|
| Data                 | Ecto schemas / plain structs             |
| Context              | A use-case module with a struct          |
| Interaction (Roles)  | Plain modules that accept data structs   |

The thing that caught my attention: in OO languages, DCI requires injecting roles into objects at runtime — mixins, decorators, method_missing tricks. Elixir doesn't need any of that. A role is just a module that takes a struct as its first argument. The "injection" is the function call itself. At least, that's how I'm reading it.

## A Concrete Example

I keep coming back to a domain I use for talks — Harvest Tracker, an application for managing seasonal contracts between farms and restaurants. Let me walk through scheduling a delivery, where three structs each play a distinct role.

### Data — Just the Facts

In this experiment, I'm keeping the schemas free of use-case-specific behavior:

```elixir
defmodule HarvestTracker.SeasonalContract do
  use Ecto.Schema
  schema "seasonal_contracts" do
    field :produce, :string
    field :committed_kg, :decimal
    field :delivered_kg, :decimal
    field :status, Ecto.Enum, values: [:active, :fulfilled, :cancelled]
    belongs_to :farm, Farm
    belongs_to :restaurant, Restaurant
  end
end
```

### Interaction — Roles Reveal Intent

Here's where it gets interesting. Each participant plays a named role in this scenario:

```elixir
defmodule HarvestTracker.ScheduleDelivery.FulfillmentLedger do
  def record_commitment(contract, quantity_kg) do
    remaining = Decimal.sub(contract.committed_kg, contract.delivered_kg)

    if Decimal.compare(quantity_kg, remaining) != :gt do
      {:ok, %{contract | delivered_kg: Decimal.add(contract.delivered_kg, quantity_kg)}}
    else
      {:error, {:exceeds_allocation, remaining}}
    end
  end
end

defmodule HarvestTracker.ScheduleDelivery.HarvestSupplier do
  def reserve(farm, produce, quantity_kg) do
    # Check and reserve available inventory
  end
end

defmodule HarvestTracker.ScheduleDelivery.DeliveryRecipient do
  def validate(%{account_status: :active}), do: :ok
  def validate(%{account_status: :suspended, name: name}),
    do: {:error, {:account_suspended, name}}
end
```

`FulfillmentLedger`, `HarvestSupplier`, `DeliveryRecipient` — what I like about these names is they tell you *why* each struct is in this scenario, not just what type it is. A `SeasonalContract` is a `FulfillmentLedger` here, but it could be an `AuditSubject` in a compliance flow. The role stays with the use case, not the data.

### Context — The Scenario as Data

The context assembles participants into a struct, then executes the scenario:

```elixir
defmodule HarvestTracker.ScheduleDelivery do
  alias __MODULE__.{FulfillmentLedger, HarvestSupplier, DeliveryRecipient}

  defstruct [:contract, :farm, :restaurant, :quantity_kg,
             :scheduled_date, :requested_by, :correlation_id, :outcome]

  def execute(%__MODULE__{} = ctx) do
    with :ok             <- DeliveryRecipient.validate(ctx.restaurant),
         {:ok, contract} <- FulfillmentLedger.record_commitment(ctx.contract, ctx.quantity_kg),
         {:ok, farm}     <- HarvestSupplier.reserve(ctx.farm, ctx.contract.produce, ctx.quantity_kg) do
      {:ok, %{ctx | contract: contract, farm: farm, outcome: :scheduled}}
    end
  end
end
```

Making the context a struct is where this started clicking for me. Before `execute/1` runs, the struct is an inspectable snapshot of the assembled scenario. After, it carries the outcome. Tests read like scenario specifications with no database required:

```elixir
test "rejects delivery exceeding allocation" do
  ctx = %ScheduleDelivery{
    contract:    %SeasonalContract{committed_kg: Decimal.new("100"), delivered_kg: Decimal.new("90")},
    farm:        %Farm{inventory: [%{produce: "kale", available_kg: Decimal.new("50")}]},
    restaurant:  %Restaurant{account_status: :active},
    quantity_kg: Decimal.new("20")
  }

  assert {:error, {:exceeds_allocation, _}} = ScheduleDelivery.execute(ctx)
end
```

## Noun vs. Verb: A Naming Convention That Emerged

One convention fell out of this exercise that I find clarifying. The domain context is a **noun** — `ScheduleDelivery` names a domain concept. The application service that orchestrates I/O around it is a **verb** — `ScheduleHarvestDelivery` names what the system does. If you see a noun, you're in pure domain territory. If you see a verb, you're at the boundary where side effects happen. I'm not sure this holds up in every case, but it's been a useful heuristic so far.

## Where CRC Fits In

Bruce Tate's CRC pattern (Constructors, Reducers, Converters) initially seemed at odds with DCI to me. CRC pushes behavior toward the data module — `record_commitment/2` would naturally live on `SeasonalContract` as a reducer. DCI pulls it out into a role module.

I think they might operate at different scopes. If a transformation is *intrinsic* to the type regardless of use case — `SeasonalContract.cancel/1` — it probably belongs on the data module per CRC. If it's *role-specific* to a scenario — recording a commitment as part of scheduling a delivery — maybe it belongs in the role module per DCI.

That boundary — intrinsic vs. role-specific — seems like a useful question to ask whenever I'm deciding where a function lives. Though I suspect the line isn't always clean in practice.

## What I'm Still Thinking About

I haven't battle-tested this in a large codebase yet. The pattern feels right for complex domains where the same data participates in multiple use cases with different behaviors. For simpler CRUD flows, it would be overhead.

But the core idea — that a `SeasonalContract` shouldn't have to know about every scenario it might participate in — that feels like something worth exploring further. And Elixir's module system seems particularly well suited to it.
