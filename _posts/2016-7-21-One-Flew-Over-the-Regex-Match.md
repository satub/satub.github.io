---
layout: post
title: One Flew Over the Regex Match
---

#### Why Regex?

At first glance, regex can feel overwhelming and intimidating.

![cat buried in a snow drift](https://i.chzbgr.com/full/8822440960/h09650A10/)

[comment]: # First I was like this ![](https://i.imgflip.com/o0nfo.jpg)

[comment]: # Take a few screenshots from MC in different search modes and toss them here?

[comment]: # Then I was like this ![](http://itsfunny.org/wp-content/uploads/2013/01/Funny-cat-faces-with-sayings.jpg)

_Why am I here? Can't I just search with exact words? This syntax is not for cats! Oh, well, let me Google up some guides..._
_So... \S is a non-whitespace character, . is any single character, \w is a word character, yada yada yada and then..._

> `\b[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}\b` is a more complex pattern.

_You don't say!_

![Rosetta's stone](https://upload.wikimedia.org/wikipedia/commons/thumb/4/46/RosettaStoneAsPartOfOriginalStele.svg/314px-RosettaStoneAsPartOfOriginalStele.svg.png)
_Image Source:  [Wikipedia - Rosetta Stone](https://en.wikipedia.org/wiki/Rosetta_Stone)_

_Great, so now I should know how to translate Egyptian hieroglyphs to Demotic script to Ancient Greek!_

* Understanding the basics...
  + character (think of ruby filenames: letter, underscore, number), digit, whitespace, or a non-character
  + match any or match specifically
  + what about or?
* How many chars are matched?
  + The difference between +, ?, \*, {n}, {n,}, {n,m}
* What are optional matches?
  + the power of ?


_Note to self: remember to be mindful when searching for strings that could be substrings for other strings. Unless, of course, you won't mind watching splatter punk instead of comedies (slaughter vs laughter), or are prepared to pay the ticket for driving into a lieutenant parking instead of tenant parking! `/\w{3}\/` matches any three consecutive letters, underscores or numbers in any length of a word; if you want to find words of exactly 3 letters long, use word boundaries or non-word characters such as whitespace characters to specify that! `/\b\w{3}\b/`_


After learning all the above, a sudden impulse of producing philosophical paragraph propagates:

So, how can the computer tell the kitten apart from the snow? For a human this might seem obvious -- stick your hands into the snow nearby the boundary between the brown and white colors and if you get scratched, you've found the cat -- but a computer needs to be told what pattern it should be looking for in different terms. In regex, text can be viewed by alternating patterns of word- and non-word characters. Like clusters of letters and digits separated from one another with spaces and punctuation. Alternating vowels and consonants form stripe patterns into a word, dots and 'at's are the joints of an email address. Maybe the question is not "Why is regex so difficult to understand?" but "How does the human brain execute pattern matching so effortlessly?"

  [comment]: # Optional picture of Sherlock Holmes goes here.

##### What Are Capturing Groups?

To be able to understand backreferences in regex, one has to first understand how capturing groups work. In regex, we capture patterns into groups by surrounding them with parentheses. For example, let's try regex expressions on this classic poem

> Cats sleep anywhere, any table, any chair.      
> Top of piano, window-ledge, in the middle, on the edge.     
> Open drawer, empty shoe, anybody's lap will do.     
> Fitted in a cardboard box, in the cupboard with your frocks.    
> Anywhere! They don't care! Cats sleep anywhere.     

_Eleanor Farjeon (1881 - 1965)_  

`/\b([Cc]ats?)\W/`

 + matches the word Cats with one non-word character following it: "Cats "  
 + captures `Cats` & `Cats`

`/\b(\w*)\!/`

 + matches words that are followed by an exclamation mark, here "Anywhere!" and "Care!"  
 + captures `Anywhere` & `care`  

`/\b\w*(\w)\1\w*\b/`

  + matches words that have two adjacent identical letters: "sleep", "middle", "will", "Fitted", and "sleep"  
  + captures `e`, `d`, `l`, `t`, and `e`  


In the last example, `\1` is a backreference to capture group number one in each individual match. The groups are always numbered from left to right. Correct numbering is crucial for any form of successful backreferencing. To clarify this issue for any regex you might be playing with, you can test both the matched result as well as the captured groups when testing your expressions on Rubular. For example, if you would like to add groups for whatever reason to the last example, eg `/\b(\w*)(\w)\1\w*\b/`, the code would break as `\1` is now referring to group `(\w*)`. To keep your second group in the code, you could either change the reference number to `\2` => `/\b(\w*)(\w)\1\w*\b/` or mark the first group as non-capture group with `(?:)` => `/\b(?:\w*)(\w)\1\w*\b/` Voilà, now `(\w)` is again the first captured group!

Backreferences are extremely helpful in matching for repeated patterns. For example, imagine that you got the sudden urge to find data for all the animals that have tautonymous name (eg _vulpes vulpes_) from a humongous taxonomy datadump. A simple pattern search with `/\b(\w+)\s\1(\s\1)?\b/` will find our friend the lynx, the red fox, the european eagle owl, the western lowland gorilla, and many others without having to get loopy with scanned each letter or worse still, having to type out the exact name for each species for matching.




Time to celebrate! Feeling like a pro: ![](http://i.imgur.com/alu7wsS.png)    
[Try this yourself](http://regexone.com/problem/extracting_url_data?)

* Getting a kick out of backreferences
  + what do these regexes do? `/(.{3}).*\1/`  && `/(.)(.)\2\1/`

* Efficiency?

### Resources

* [Interactive Tutorial to Regex Matching](http://regexone.com/)
* [Test your regex for ruby at Rubular](http://rubular.com/)
* [Regex Reference for Capturing Groups and Backreferences](http://www.regular-expressions.info/refcapture.html)
* [Need a Break? Go Play Golf!](http://regex.alf.nu/)
