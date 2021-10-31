---
title: Preventing Brain Freeze - Living Documentation in Elixir
published_url:
tags:
  - elixir
  - living-documentation
  - presentation
---

# Living Documentation in Elixir

Living Documentation consists of a number of patterns that helps us leverage, augment and curate
knowledge to share with new developers.

There are close to 100 different techniques and practices that Martraire writes about in his book on
the subject, but we'll look at a handful of these and how they can be applied to our Elixir
projects.

I think it would be rather boring to describe an in depth catalog of these patterns, it would be
much more fun to discuss these patterns and techniques in the context of an Elixir project and how
they improve the knowledge sharing in the onboarding process.

## An example application

To make the application of Living Documentation more concrete, I’ve created a [simple umbrella
project for a wealth management platform](https://www.github.com/nicholasjhenry/pockets_platform).

Under the 'apps' directory I have stubbed out multiple child applications with names you would
expect to see in a platform targeting a financial domain.

We'll be focused mostly on a specific module that calculates the rate of return for a client's
financial portfolio.