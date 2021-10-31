---
title: Peventing Brain Freeze - Getting started with Living Documentation in Elixir
published_url:
tags:
  - elixir
  - living-documentation
  - presentation
---

## Getting started with Living Documentation in Elixir

Now that we've learnt the concepts and some patterns of living documentation applied to Elixir. How do we get started applying this to your project?

My recommendation is always to start with small experiments, **never** large documentation efforts. You want to avoid the trappings of traditional documentation such as stalling feature development with these types of initiatives.

Small experiments give you the opportunity to try some of these ideas without risk and help generate interest from your other team members.

Here are a couple experiments you can try:

First, with the next feature you add to the code base, try README-Driven Development. As a reminder this doesn't necessary mean a README markdown file located in the base of your project. Start with overview documentation for the feature located in a module using the `@moduledoc` attribute. Start with describing the problem domain and see how this improves naming of concepts and the overall design of the solution domain. Maybe even try some `iex` examples that can be verified by the test suite. These examples can be used as an entry point for new developers to experiment with the application.

Second, add `ex_doc` as a dependency to your application and try to generate the documentation. If some modules and functions have already been documented you may receive warnings as the documents are generated. This is a great place to start in making documenting a first class member of your project. From there, starting navigating the documentation and determine how easier it is reason about. Do the names in core domain concepts make sense? Can I restructure modules to make it easier for new developers to navigate?

Finally, you could start your with experimentation with LiveBook. If you have a complex problem domain, try to build an interactive tutorial to explain it. This can result in an amazing onboarding asset.

## Onboarding is knowledge sharing

Remember onboarding is knowledge sharing, practice knowledge sharing with Living Documentation to prevent the next ice cream headache for your new developers.

If you have any questions, please ping me on the conference or social media platforms. Thank you so much for your time today and take care.