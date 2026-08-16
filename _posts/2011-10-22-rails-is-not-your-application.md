---
title: Rails is Not Your Application
canonical_url: https://blog.firsthand.ca/2011/10/rails-is-not-your-application.html
---
## Background: The Beginnings of My Obsession with Application Architecture and Domain Modelling

Last year I was involved in my most complex Ruby on Rails application to date that is now 23K lines of model code, 14K lines of controller code, and 4.5K lines of lib code. You get the point, let's now worry about the view code. It was during this project that I observed the pain of working within the bounds of how we typically build Rails applications. Did I mention it was slow to develop and test? During that contract and ever since, I have been **obsessing about application architecture and domain modelling**.

## Establish an Application Interface with a Service Layer

During the time I was working on said application, I introduced the concept of **Services** (in the early stages I called them managers). Although a lot of domain logic was pushed into the models, there was too much application logic in the controllers. Services, co-ordinating domain models, allowed me to extract that application logic to really keep the controllers thin. I'm not sure exactly sure what prompted me to start using Service objects, but after the fact I found [references](http://martinfowler.com/eaaCatalog/serviceLayer.html) to them in Martin Fowler's [Patterns of Enterprise Application Architecture](http://www.amazon.com/Patterns-Enterprise-Application-Architecture-Martin/dp/0321127420) (P of EAA). Other frameworks such as [Grails](http://www.grails.org/) also use [Services](http://www.grails.org/doc/1.3.x/guide/8.%20The%20Service%20Layer.html), it even has a generator for them. Services resolves the [discussion](http://david.heinemeierhansson.com/posts/43-think-of-emails-as-views-delivered-through-smtp) ([started here](http://www.robbyonrails.com/articles/2009/11/16/sending-email-controllers-versus-models)) on where to trigger email notifications. Add them to your Services.

In P of EAA, Randy Stafford defines the Service Layer as:

*"Defines an application's boundary with a layer of services that establishes a set of available operations and coordinates the application's response in each operation."*

And in a way, this definition helped me recognize another frustration I was having with how I (and most of the Rails community) was developing with Rails. **There was no explicit application**. Or rather, it was hard to define what the application was and hard to know how to interact with it. The application had no interface. My application was a ball of ActiveRecord objects.

## Lots of Activity in the Rails Community Regarding Domain Modelling

I haven't been the only one thinking about application architecture and domain modelling. I'm not a unique snowflake. Others in our community have too: Mike Dietz from ThoughtWorks introduce me to [DCI at RailsConf 2011](http://en.oreilly.com/rails2011/public/schedule/detail/19424), [Andrzej Krzywda has been blogging](http://andrzejonsoftware.blogspot.com/2011/02/dci-and-rails.html) about it before that, and [Jim Gay has recently been blogging](http://www.saturnflyer.com/blog/jim/2011/10/04/oop-dci-and-ruby-what-your-system-is-vs-what-your-system-does/) about it on this side of the pond. On another tact, [Corey Haines has been been selling fast tests](http://confreaks.net/videos/641-gogaruco2011-fast-rails-tests) over the last few months. Hidden in that sales pitch of course, is a way to extract domain logic. Talking of extracting domain logic, [Steve Klabnick has illustrated some practical examples](http://blog.steveklabnik.com/2011/09/22/extracting-domain-models-a-practical-example.html).

These guys have kept me busy! I've been quietly researching this topic for over a year now. You can check out some of my tags on Delicious:

- [DCI](http://delicious.com/nicholas.henry/dci)
- [domain-driven-design](http://delicious.com/nicholas.henry/domain-driven-design)
- [design-patterns](http://delicious.com/nicholas.henry/design-patterns)
- [software-construction](http://delicious.com/nicholas.henry/software-construction)

## Extracting the Application from the Framework

Ok, time to finish up this post. All of the efforts describe above have been with the purpose of managing the application and domain logic in the software we build. It benefits us with faster tests and code that clearly communicates it's intent. And although the Service Layer was helping me define an interface to my application, I was only using them when my controllers methods/actions were getting heavy with application logic. Then last night, I watched the [latest release of Clean Code](http://www.cleancoders.com/codecast/clean-code-episode-7/show) from Uncle Bob that highlights the need to extract your application from the framework you use (he's been [teasing me](http://cleancoder.posterous.com/architecture-deference) for a while on this topic). With my previous experience with Services, and my interest in DCI, Uncle Bob's guidance was the piece I was missing.

After taking his example pseudo code, and turning it into some Ruby/Rails example (more like snippet) I gleefully [tweeted](http://twitter.com/#!/nicholasjhenry/status/127779236487507968):

*"Rails is not your application. It might be your views and data source, but it's not your application. Put your app in a Gem or under lib/."*

I really like this approach a lot, the ability to test your application outside of the Rails stack is huge, and viewing Rails as a method to deliver my application will change the way I write code. Anyway here is [the code that triggered the Tweet](https://gist.github.com/1305650) (see below). **It's not polished, it doesn't execute, it's not 100% right, but boy it's got me thinking**. It uses Services to represent Use Cases of the application, and roles containing domain logic that is mixed in contextually, extracted into a Gem, delivered via Rails. If you not a fan of the Gem idea, keep it name spaced under your lib directory.

**Update:** A clarification on putting your application in lib. I certainly don't mean to put it directly in lib. Name space it, lib/your_application; e.g. *lib/pay_roll*.

Now that I have let the code out of the bag, expect some more posts on this topic in the future where I work through and describe this code (and related ideas) in more detail.

[View the example on GitHub Gist](https://gist.github.com/1305650)
