---
title: Preventing Brain Freeze - Augment your source code for onboarding
published_url:
tags:
  - elixir
  - living-documentation
  - presentation
---

# Augment your source code for onboarding

Often source code doesn't tell the full story and there may even be an impedance mis-match between representations in the problem and solution domains inhibiting onboarding. This is where **Knowledge Augmentation** patterns can help us.

We can **Augment Code** to clarify our code bases. One of my favourite examples of this is financial calculations. Let's take an example of the Rate of Return. If we visit the Investopedia, an online resource for finance, they define rate of return as:

> A rate of return (RoR) is the net gain or loss of an investment over a specified time period, expressed as a percentage of the investment’s initial cost. When calculating the rate of return, you are determining the percentage change from the beginning of the period until the end. -- Investopedia

While the formula that looks like this in the problem domain…

TODO: Add formula

…An implementation in Elixir may look like this in our solution domain when using the Decimal module.

```
  def calculate(current_value, initial_value) do
    current_value
    |> Decimal.sub(initial_value)
    |> Decimal.div(initial_value)
    |> Decimal.mult(100)
  end
```

### ExDoc + Katex

Our definition in the problem domain looks very different to our implementation in the solution
domain. Can we do better for our new developers?

Fortunately, when working with ExDoc we can add additional functionality to help us communicate with
our new developers. ExDoc after all is just HTML, and it open to extensions by using additional
tools such as JavaScript libraries.

In the case of financial calculations, we can utilize Katex, a math typesetting JavaScript library
to render formulas.

### Exdoc configuration

Let’s take a look at how to extend ExDoc with Katex. We perform this confirmation in our Mix project
module.

1. (Click) For nested application configuration I like to extract that out into a dedicated
   function, hence the ‘docs’ function.

2. (Click) The ExDoc `before_closing_head_tag` configuration key allows us to insert any HTML before
   the closing head, this is where we reference the Katex Javascript library.

3. (Click) Again, I've extracted an intention revealing function describing the markup we are going
   to include before the closing head tag.

### Documentation Example

Now that we have ExDoc setup with Katex, let's see how this can be used in documentation for the
Rate of Return module.

1. (Click) In the module doc we have included the definition of Rate of Return from the
   Investopedia.

In the Living Documentation pattern language, this is an example of **Embedded Learning**, putting
more knowledge into the code helps new developers learn while working on it.

The second pattern this demonstrates is using **Ready-made documentation**.

Often for problem domains, the knowledge is already documented and we just need to link to it from
our source code.

Living documentation focuses on writing less documentation and utilizing existing documentation.

The final pattern this demonstrates is **Stable Documentation**. Stable documentation has a great
ROI, invest in this documentation because it is unlikely to change. In our example here, I think we
can agree that the Rate of Return calculation is not going to change.

6. (Click) Inside the module attribute, ‘formula’, we include the Katex markup.

7. (Click) And we’ve included the module attribute between the double dollar signs. This is an
   indicator by the Katex library to render the MathML.

8. (Click) Working code, based on the example for Investopedia is also included, verified by our
   test suite.

### Generated documentation with Katex

With our rendered documentation using `mix docs`, we are able to communicate the problem domain, in
this case the formula, while our Elixir code is used to implement the solution domain.