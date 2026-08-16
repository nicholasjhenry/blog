---
title: Your Rails Application is Missing a Domain Controller
canonical_url: https://blog.firsthand.ca/2011/12/your-rails-application-is-missing.html
---
The mantra of [Skinny Controller, Fat Model](http://weblog.jamisbuck.org/2006/10/18/skinny-controller-fat-model) reminds us to keep our business logic in our models, not in Rails Action Controllers. We know Action Controllers, sub-classes of `ActionController::Base`, are only responsible for delegating service requests to the domain layer. These models, which encapsulate this business logic, forming the domain layer, represent *entities* such as a person, place, event or thing. They are often, **but not always**, mapped directly to database tables through the inheritance of `ActiveRecord::Base`. When we develop our domain layer for our complex Rails application, purely **with models representing entities**, we encounter two problems:

## Incoherent domain API

When Action Controllers interact with model entities directly, the API to the domain layer becomes clouded. It is unclear what services the domain layer offers and how the UI should request those services. Rail’s ActiveRecord exasperates the issue, for example, there are several ways to create and update model entities and their associations.

## Bloated domain models

Business logic that is workflow-oriented involving multiple entities is assigned to a single entity, bloating it’s responsibilities. It becomes difficult to differentiate the entities in the domainrepresenting what the system is, from the use cases or workflow representing what the system does<sup>[1](#fn1)</sup>.

The solution is a Domain Controller. **A Domain Controller is responsible for co-ordinating service requests** from your Rails Action Controllers. A Domain Controller can come in two flavours, a *facade* or a *use case*.

*A clarification on terminology: Throughout this blog post I will refer to the typical Rails model as a **Model Entity**, meaning a model that represents a person, place, event or thing.*

## Facade Controller

A Facade Controller represents the overall system that encapsulates the domain layer. For example a *Catalog* object can act as facade for the domain layer containing taxonomies, taxons, product groups, products and variants.

Without the Facade Controller, our service requests to the domain typically look like this:

``` ruby
Product.active
Taxon.active.find(1)
Product.create(:name => 'Ruby Meta-Programming')
```

With a Facade Controller instance, in this case, a *catalog* object, service requests appear as the following, delegating to the appropriate Model Entities:

``` ruby
catalog.fetch_products
catalog.fetch_taxon(1)
catalog.create_product(:name => 'Ruby Meta-Programming')
```

The Facade Controller provides a clear, explicit interface into the catalog domain layer that a programmer can grok within seconds.

## Use Case Controller

A Use Case Controller is useful for more complex workflows. They are also helpful when a Facade becomes bloated and you wish to split the incoming service requests across multiple Domain Controllers. It’s important to note, **you can use more than one Domain Controller for your application**.

Let’s look at an example. The code below represents the processing of a Payroll<sup>[2](#fn2)</sup>. Often this type of workflow logic gets placed in a Model Entity that it doesn’t belong to or worst, left in a Rails Action Controller.

``` ruby
date = Date.today
employees = Employee.active
employees.each do |e|
  if e.pay_date?(date)
    pc = PayCheck.new(e.calculate_pay(date))
    e.send_pay(pc)
  end
end
```

So what object is responsible for this workflow logic? We know a Rails Action Controller isn’t a candidate. Its responsibility is to handle HTTP requests and delegate to the domain layer. Neither the models `Employee` or `PayCheck` are valid candidates. They represents the entities in our domain layer. Therefore we, require a new object *PayDayService* to encapsulate the logic, a Use Case Controller.

``` ruby
class PayDayService
    def initialize(date=Date.now)
      @date = date
    end

    def call
      Employee.active.each do |e|
        if e.pay_date?(@date)
          pc = PayCheck.new(e.calculate_pay(@date))
          e.send_pay(pc)
        end
      end
    end
  end
```

The benefit of this approach is that we have encapsulated this workflow logic without bloating existing Model Entities and allows the reuse of this logic in different contexts.

## Domain Controllers in a Rails Application

When an application has multiple Domain Controllers I place them in a dedicated directory under app called *services* to keep them separate from the Model entities in `app/models`. This helps communicate that Rails Action Controllers should delegate to the Domain Controllers, not directly to Model Entities. The Domain Controllers and Model Entities together, represent the domain layer of the system. I have also discussed extracting the entire domain layer from the Rails framework, but that’s [another discussion](http://blog.firsthand.ca/2011/10/rails-is-not-your-application.html). Let’s keep this blog post civil for now!

I hope the Domain Controller pattern provides you with another tool for modelling your domain, reduces the coupling with the UI, and provides a clean API to your application logic.

## Further Reading

If you are interested in learning more about Domain Controllers and Services, check out the following resources:

- [Applying UML and Patterns](http://www.amazon.com/Applying-UML-Patterns-Introduction-Object-Oriented/dp/0131489062) by Craig Larman. Craig includes the *Controller* pattern as one of the *General Responsibility Assignment Software Patterns or Principles (Grasp)*.
- [Architecture the Lost Years](http://confreaks.net/videos/759-rubymidwest2011-keynote-architecture-the-lost-years) a presentation by Robert Martin discusses modelling your application using Interactors and Entities. *Interactors\_* (which I have used the name Domain Controller) contain application specific logic, and *Entities* (which I have use the term Model Entities) contains application independent logic. Martin choses to use the term *Interactors over Controllers* to avoid confusing with Model-View-Controller (MVC). I prefer to use Domain Controller or Service to differentiate from Rails Action Controllers as I just can’t get my head around the term Interactors.
- [Object-Oriented Software Engineering—A Use Case Driven Approach](http://www.amazon.com/Object-Oriented-Software-Engineering-Approach/dp/0201544350) by Ivar Jacobson introduced the concept of interface, entity (Model Entity) and control (Domain Controller) objects over two decades ago. Robert Martin often refers to Ivar Jacobson when discussing application architecture.
- [Patterns of Enterprise Application Architecture](http://www.amazon.com/Patterns-Enterprise-Application-Architecture-Martin/dp/0321127420) by Martin Flower discusses the concept of a “Service Layer” to give the domain layer an explicit API.
- [Domain Driven Design](http://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software/dp/0321125215) by Eric Evans. Evans discusses the concept of Services for when “some concepts from the domain aren’t natural to model as objects”.

## Footnotes:

1.  “What-the-system-is” and “What-the-system-does” terminology is from the book [Lean Architecture](http://www.amazon.com/Lean-Architecture-Agile-Software-Development/dp/0470684208/) by James O. Coplien and Gertrud Bjørnvig.
2.  Logic based on the Payroll case study in [Robert Martin’s Agile Software Development](http://www.amazon.com/Software-Development-Principles-Patterns-Practices/dp/0135974445).
