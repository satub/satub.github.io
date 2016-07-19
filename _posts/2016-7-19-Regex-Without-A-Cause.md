---
layout: post
title: One Flew Over the Regex Match
---

Work in progress.

* Understanding the basics …
  -- character (think of ruby filenames: letter, underscore, number), digit, whitespace, or a non-character
  -- match any or match specifically
  -- what about or?
* How many chars are matched?
  -- The difference between +, ?, \*, {n}, {n,}, {n,m}
* What are optional matches?
  -- the power of ?
* Are we matching a whole word or just an element of it?
  -- Word boundaries, line starts and ends

* What is grouping?
  -- ()()()
  -- http://i.imgur.com/alu7wsS.png
  -- try it yourself => http://regexone.com/problem/extracting_url_data?
* How to match without grouping?
  -- (?:)
* Getting a kick out of backreferences
  -- progression, numbering
  -- what does this do? (.{3}).\*\\1


Resources

bunch of links here


Interactive Tutorial to Regex Matching: http://regexone.com/
Test your regex for ruby at http://rubular.com/
Regex Reference for Capturing Groups and Backreferences http://www.regular-expressions.info/refcapture.html
