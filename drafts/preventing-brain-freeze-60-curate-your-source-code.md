---
title: Peventing Brain Freeze - Curate your source code for onboarding
published_url:
---

# Curate your source code for onboarding

Within your project there is a massive amount of knowledge stored in the source code. But not all
knowledge is equally important or necessary to share. This is where our third and final strategy
comes into play, curation. Curation allows us select the relevant knowledge in our domain and share
that with our team members.

## Guided Tour and Site-Seeing Maps

A curation pattern we'll review is the **Guided Tour and Site-Seeing Maps**. Onboarding a new
developer to a project can be like visiting a new city. A Guided Tour or a Site-seeing Map can be
invaluable to explore the must-see attractions and sometimes introduce you to some interesting
unknowns.

Although, known for it application in data science, LiveBook, Elixir's interactive and collaborative
code notebook, is a tool we can use to create Guided Tours and Site-seeing Maps. It's even pointed
out in the LiveBook README under the section custom runtimes:

> when executing Elixir code, you can either start a fresh Elixir instance, connect to an existing
> node, or run it inside an existing Elixir project, with access to all of its modules and
> dependencies. **This means Livebook can be a great tool to provide live documentation for existing
> projects.**

Let's take a look at connecting LiveBook for a Guided Tour. After starting up LiveBook at the
command line, we navigate to the `tour` directory of our umbrella project, here we select the guided
tour for financial ratios.

This is a simple example, based on our Rate of Return calculation, (and I have literally copy and
pasted the documentation), but unlike our static documentation we can execute the examples.

Although we’ve selected our notebook, we then need to select a runtime to evaluate our examples. The
runtime we will select is our umbrella project as a mix standalone node.

Once connected, we can evaluate our examples.

We could expand this example further to provide a complete tour of the financial portfolio
management application with the exiting implementation.

I particularly like this example, as LiveBook has support for Katex out of the box, the MathML
language that we wired up in ExDoc to render the formula.

## A Story from the field

One team I worked with included a manual verification step as part of their review process. Manual verification steps are great for the UI, but can be a little repetitive for backend only features. Often it included 15 different lines of code to copy into IEX. How cool would it be to provide a markdown file that the reviewer could load up into LiveBook? Could these verification markdown files even be committed to the project’s repo?