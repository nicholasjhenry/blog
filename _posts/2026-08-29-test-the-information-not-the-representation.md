---
layout: quote
title: Test the information, not the representation
quote: |
  Tests should be written in terms of the information passed between objects,
  not of how that information is represented.
attribution: Steve Freeman and Nat Pryce, Growing Object-Oriented Software, Guided by Tests
---
This is the line I reach for when a test starts asserting on encoded JSON keys
instead of the values the caller actually passed in. Change the encoding and the
test breaks, though nothing about the behaviour did.
