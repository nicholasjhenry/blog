---
title: Knowledge Sharing - Leading and Lagging Indicators
published_url: https://medium.com/living-documentation-in-elixir/knowledge-sharing-leading-and-lagging-indicators-635aff30ca73
tags:
  - elixir
  - living-documentation
  - presentation
---

_This series of blog posts is an extract from the script for my talk presented at [ElixirConf 2021](https://www.elixirconf.com/#nicholas-henry)._


![ElixirConf 2021](./assets/images/medium.png)

1. [Onboarding new developers with Living Documentation](https://medium.com/living-documentation-in-elixir/preventing-brain-freeze-onboarding-new-developers-with-living-documentation-7e8d01d24e5)
2. **Knowledge Sharing: Leading and Lagging Indicators**
3. Living Documentation in Elixir (to be published)
4. Leverage your source code (to be published)
5. Augment your source code (to be published)
6. Curate your source code (to be published)
7. Getting Started (to be published)

---

# Knowledge Sharing - Leading and Lagging Indicators

*Where do you find the motivation and time to invest in knowledge sharing?*

![Atomic Habits by James Clear](./assets/images/atomic-habits.jpg)

I want to frame this question in the concept of habit creation. In the book [Atomic Habits authored
by James Clear](https://jamesclear.com/atomic-habits), he describes the challenge of habit creation
in terms of **leading and lagging indicators**; he states:

> Your outcomes are a lagging measure of your habits. Your net worth is a lagging measure of your
> financial habits. Your weight is a lagging measure of your eating habits. Your clutter is a
> lagging measure of your cleaning habits.

Lagging indicators are the results of your habit over time. Leading indicators is the frequency and
quality of practicing your habit. These are the indicators you can measure immediately.

For software, I would assert that **your team's velocity is a lagging measure of your software
development practices.** Or terms of onboarding: **your team's onboarding experience is a lagging
measure of your knowledge-sharing practices.**

But what are the leading indicators for good software development practices and, more specifically,
knowledge sharing habits? Let's take a look at a practice that I hope we are all familiar with.

## Test-Driven Development as design pressure

Test-Driven Development is a software development practice that, for some, kills the fear of getting
started, allows others to refactor a module without completely breaking the application, and for most
of us, sleep soundly at night.

But let's look at another facet of TDD, design pressure. [JB Rainsberger describes Design Pressure](https://www.youtube.com/watch?v=VDfX44fZoMc) as:

> ...the guidance or "pressure" that test-driven development places on your software design”

[Kelly Sutton a software developer from Gusto](https://kellysutton.com/2017/04/18/design-pressure.html) describes it this way:

> Design Pressure "is the little voice in the back of your head made manifest by a crappy test with
> too much setup.”

The gains you get from “listening” to your tests make your software more cohesive and reduce
coupling. These are your **leading indicators**. Investing in onboarding and knowledge sharing is
challenging in that we may never benefit from the lagging indicator. Or we may only benefit from the
**lagging indicator** in six months when we return to a module to add a new feature and understand
the context and the design decisions we have made. So if TDD can help us with the velocity of a
project, is there a practice that can help with onboarding but still has a leading indicator that
TDD gives us.

## Readme-Driven Development as a motivating practice for knowledge-sharing

[Readme-Driven Development was coined by Tom Preston-Werner](https://tom.preston-werner.com/2010/08/23/readme-driven-development.html), a founder of GitHub, about ten years ago,
and it has always fascinated me. Tom states:

> Writing a Readme is absolutely essential to writing good software. Until you’ve written about your
> software, you have no idea what you’ll be coding.

I propose we expand on what Readme-Driven Development might look like beyond a README Markdown file
in the base of our source code repository. README-Driven Development might describe the problem we
are solving in a moduledoc attribute of the context module. Tom goes on to state:

> Write your Readme first.
>
> First. As in, before you write any code or tests or behaviors or stories or ANYTHING.

Although I haven’t practiced this consistently throughout my career, when I have practiced it, it
hasn’t always been “first,” but I can say it has a stimulating effect on the design of the code I
write. And you may discover some of these benefits too. You may find that your names improve, and
the structure of the code changes as well.

Readme-Driven Development has the same result as "rubber ducking", a technique described in the book
[The Pragmatic Programmer](https://pragprog.com/titles/tpp20/the-pragmatic-programmer-20th-anniversary-edition/).
Rubber ducking is describe as:

> A very simple but particularly useful technique for finding the cause of a problem is simply to
> explain it to someone else. The other person should look over your shoulder at the screen, and nod
> his or her head constantly (like a rubber duck bobbing up and down in a bathtub). They do not need
> to say a word; the simple act of explaining, step by step, what the code is supposed to do often
> causes the problem to leap off the screen and announce itself.

## Better names, Better design - Your leading indicator for knowledge Sharing

Readme-Driven Development can help names leap off the screen and announce themselves. The design
will leap off the screen and announce itself. And this is the leading indicator for the habit of
knowledge sharing. Documentation will place a design pressure on the code you write, much like TDD
does. It’s a different type of design pressure; it injects humanity into your code and your project
to improve async knowledge sharing. *How might we benefit from Readme-Driven Development in Elixir?*

---

Next: Living Documentation in Elixir (to be published)