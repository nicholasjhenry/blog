---
title: Preventing Brain Freeze - Knowledge Sharing - Leading and Lagging Indicators
published_url:
tags:
  - elixir
  - living-documentation
  - presentation
---

# Knowledge Sharing - Leading and Lagging Indicators

I would like to frame this question in the concept of habit creation. In the book, Atomic Habits
authored by James Clear, he describes the challenge of habit creation in terms of leading and
lagging indicators, he states:

> Your outcomes are a lagging measure of your habits. Your net worth is a lagging measure of your
> financial habits. Your weight is a lagging measure of your eating habits. Your clutter is a
> lagging measure of your cleaning habits.

Lagging indicators are the results of your habit over time. Leading indicators is the frequency and
quality of practicing your habit. These are the indicators you can measure immediately.

For software, I would assert that **your teams velocity is a lagging measure of your software
development practices.** Or terms of onboarding: **your teams onboarding experience is a lagging
measure of your knowledge sharing practices.**

But what are the leading indicators for good software development practices and more specifically,
knowledge sharing habits? Let's take a look at a practice that I hope we are all familiar with.

## Test-Driven Development as design pressure

Test-Driven Development is a software development practice that for some, kills the fear of getting
started, allows other to refactor a module without completely breaking the application, and for most
of us, sleep soundly at night.

But let's look at another facet of TDD, design pressure.

JB Rainsberger describes Design Pressure as:

> ...the guidance or "pressure" that test-driven development places on your software design”

Kelly Sutton a software developer from Gusto describes it this way:

> Design Pressure "is the little voice in the back of your head made manifest by a crappy test with
> too much setup.”

The gains you get from "listening" to your tests, making your software more cohesive and reducing
coupling. These are your leading indicators. Investing in onboarding and knowledge sharing is
difficult in that we may never benefit from the lagging indicator. Or we may only benefit from the
lagging indicator in six months time when we return back to a module to add a new feature and
understand the context and the design decisions we have made. So if TDD can help us with the
velocity of a project, is there a practice that can help with onboarding but still has a leading
indicator that TDD gives us.

## Readme-Driven Development may be one of these practices

Readme-Driven Development was coined by Tom Preston-Werner, a founder of GitHub, about 10 years ago
and it has always fascinated me.

Tom states:

> Writing a Readme is absolutely essential to writing good software. Until you’ve written about your
> software, you have no idea what you’ll be coding. - [Tom
> Preston-Werner](https://tom.preston-werner.com/2010/08/23/readme-driven-development.html)

I propose we expand on what Readme-Driven Development might look like beyond a README Markdown file
in the base of our source code repository. README-Driven Development might describe the problem we
are solving in a moduledoc attribute of the context module.

Tom goes on to state:

> Write your Readme first.
>
> First. As in, before you write any code or tests or behaviors or stories or ANYTHING.

Although, I haven't practiced this consistently throughout my career, when I have practiced it, it
hasn't always been "first", but I can say it has an interesting affect on the design of the code I
write.  And you may discover some of these benefits too.  You may discover that your names improve
and the structure of the code changes as well.

Readme-Driven Development has the same result as "rubber ducking", a technique describe in the book
The Pragmatic Programmers.

Rubber ducking is describe as:

> A very simple but particularly useful technique for finding the cause of a problem is simply to
> explain it to someone else. The other person should look over your shoulder at the screen, and nod
> his or her head constantly (like a rubber duck bobbing up and down in a bathtub). They do not need
> to say a word; the simple act of explaining, step by step, what the code is supposed to do often
> causes the problem to leap off the screen and announce itself. -- The Pragmatic Programmers

## Better names, Better design - Your leading indicator for knowledge Sharing

Readme-Driven Development can help names leap off the screen and announce themselves.

The design will leap off the screen and announce itself.

And this is the leading indicator for the habit of knowledge sharing. Documentation will place a
design pressure on the code you write much like TDD does. It's a different type of design pressure,
it injects humanity into your code and your project to improve async knowledge sharing.