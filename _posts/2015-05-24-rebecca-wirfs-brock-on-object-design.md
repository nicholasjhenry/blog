---
title: Rebecca Wirfs-Brock on Object Design
canonical_url: https://blog.firsthand.ca/2015/05/rebecca-wirfs-brock-on-object-design.html
---
> Building an object-oriented application means inventing appropriate machinery. We represent real-world information, processes, interactions, relationships, even errors, by inventing objects that don't exist in the real world. We give life and intelligence to inanimate things. We take difficult-to-comprehend real-world objects and split them into simpler, more manageable software ones. We invent new objects. Each has a specific role to play in the application. Our measure of success lies in how clearly we invent a software reality that satisfies our application's requirements -- and not in how closely it remembers the real world.

-- Rebecca Wirfs-Brock, Object Design

I pulled this quote from Avdi Grimm's excellent keynote presentation, [The Soul of Software](https://www.youtube.com/watch?v=IgbHzFb1hGw) as a reminder this very important idea on object-oriented design: that we invent new objects that don't have a corresponding object in the real world. If we limit our domain objects to those that are represented in the real world, we miss opportunities to **encapsulate variation in behaviour** within our application.
