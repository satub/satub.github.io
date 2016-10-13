---
layout: post
title: The Return of the Regex
---

#### A Question of Flavors: JavaScript vs Ruby

Once upon October, while composing regex for matching and replacing all occurrences of 'be' in one string with Javascript (JS), I was stunned to find out that the simple expression that had worked in Ruby did not produce the same result in JS. This was almost as baffling to me as typing `[]===[]` into a JS console and receiving `false`. The day of reckoning was upon me:

![I has a flavor kitten](https://s-media-cache-ak0.pinimg.com/564x/ae/7f/90/ae7f90ad1dd5e433d6a464610bcf8c71.jpg)  

>As usual in the software world, different regular expression engines are not fully compatible with each other. The syntax and behavior of a particular engine is called a regular expression flavor.    

_Quote from [www.regular-expressions.info](http://www.regular-expressions.info/tutorial.html)_

So, what are these differences? Is regex with Ruby sweet and with JS savory? Let's dig in and find out!  

Unlike in Ruby, in JS an expression such as `/a/` looks only for the first occurrence unless you define a modifier: g. First, let's head out to [rubular](http://rubular.com/r/lJlmUzd7gc). As you can see, all a's have been matched. Now, copy the text and check out [Regexpal](http://www.regexpal.com/). Check off the g modifier from the flags and see how `/a/` will only match the first a. Reactivate the modifier and it looks like ruby again! A word of caution, though, in case that you are now thinking that you can use the modifier to make all JS methods work like Ruby methods; as we'll see, both have methods that totally disregard global matching regardless of the defaults or added modifiers.    

If you have gotten used to matching [the beginning](http://rubular.com/r/Jiw1Rcg5qF) and end of a string with `\A` and `\Z`, you can forget all about that in JS. There are _no_ such anchors; you must use ^ and $ instead. In Ruby, [the caret](http://rubular.com/r/TaJkBOz3Bs) and the dollar always match before and after newlines; you cannot change this. However, in JS you can make them match either the borders of a string or newlines by switching the m modifier on or off. Try inserting this: `^(I\s)` to Regexpal and play with the m modifier. Another little quirk of the m (stands for multiline) modifier is that in Ruby, it can be used to force . to match newline. In JS, . _never_ matches newline. If you really want to match _any_ character, including newlines, you'll have to use a combination of a character-class and non-character-class-shorthand notations, eg `([\w]|[\W])` or `([\s]|[\S])`.

Both positive and negative look-arounds work in JS and Ruby. You can find [the English](http://rubular.com/r/6u8yKqwTed) and [the US](http://rubular.com/r/Zi6JPThmwc) kitten in both flavors with these: `o(?=u)` & `o(?!u)`.
However, look-behinds such as [`(?<=a)s`](http://rubular.com/r/nSkVKYTBgG) do not work at all in JS.   

Creating regexp in Ruby and JS doesn't look that different. In Ruby:    
  * `re = /pattern/`   
  * `re = r%{pattern}`   
  * `re = Regexp.new("pattern", "modifier"(optional))`    <== this one needs quotationmarks around the expressions   

And in JS:   
  * `var re = /pattern/`    
  * `var re = new RegExp("pattern", "modifiers"(optional))`  <== this one needs quotation marks around the expressions   

Now that we have expressions ready, let's use some methods!

#### Methods, Methods, Methods

As mentioned above, there is no global g modifier in the actual Ruby regexp. Instead, the method used decides whether the method finds or edits all occurrences in the given string, for example: sub matches and substitutes only one occurrence, whereas gsub substitutes all of them. Scan returns an array of all matches, and match only finds one match. If you try to add the g modifier into your ruby regex, you will get a syntax error:   
`/a/g.match(blaab2)[0]     
SyntaxError: (irb):22: unknown regexp option - g`   
In JS, you can choose between one and global substring replacement with the g modifier when using the replace() method.

Replacing and substituting with a backreference is very similar...

Ruby:    

  `re = %r{((I\slove\s)((\w+\W*)*))}`    
  `blaab = "And so yada yada yada. I love walking on the beach! In addition...."`
  `blaab.sub(re, "I love cats! Who cares about \\3")`    
  ==>
  `"And so yada yada yada. I love cats! Who cares about walking on the beach! In addition...."`    

JS:    

  `var re = /((I\slove\s)((\w+\W*)*))/`    
  `var blaab = "And so yada yada yada. I love walking on the beach! In addition...."`    
  `var theTruth = blaab.replace(re, "I love cats! Who cares about $3")`    
  ===>   
  `"And so yada yada yada. I love cats! Who cares about walking on the beach! In addition...."`


Checking for matches also works similarly:

`"I has a flavor".scan(/u/).empty? => true` (ruby)    
`/u/.match("I has a flavor").nil? => true` (ruby)    
`/u/.test("I has a flavor") => false` (js)   
`"I has a flavor".match(/u/) => null` (js)        

But what if there _are_ matches? How do we get our hands on those?

In ruby: `md = regex.match(string)` returns a MatchData object. You can extract the following information from it:   
`md[0]` First of the matched strings `m.to_a` returns all matches in an array.        
`md.captures` An array of capture groups if such were present in the regexp.    
`md.string` Returns the original string.   
`md.begin(0)` returns the position of the first letter of the match in the original string.    

Match is not global in ruby! In JS you can turn match() global with g and it will return matches in an array of strings:     
`"I has a flavor".match(/a/g)   
===>   
["a", "a", "a"]`   
 As a default - aka w/o the g-modifier -  it returns only the first match:    
 `"I has a flavor".match(/a/)   
===>  
["a"]`   

 In Ruby, to get and array of matched strings, you can use scan instead.

<!-- A final peculiarity will have to go to JS: its RegExp Object has a `lastIndex` property that calling exec() on the RegExp will change. Check this out:   
`var myRegex = /a/`   
`var myMatch = myRegex.exec("I has a flavor").index` will return 3 on the first run, but will change on consecutive runs => 6 => 11 => `Uncaught TypeError: Cannot read property 'index' of null(…)`     -->

<!-- Huh? Are we back to that `[]===[]` returns `false` now?    -->


<!-- ![I can't even](https://cdn.meme.am/instances/500x/60560544.jpg) -->
<!-- ![this is why we can't have nice things](http://i2.kym-cdn.com/photos/images/newsfeed/000/041/084/why_we_cant_have_nice_things.jpg) -->
<!-- ![one does not simply explain javascript](https://i.imgflip.com/1c9943.jpg) -->



##### Resources:

[Rubular (test Regex in Ruby flavor)](http://rubular.com/)      
[Regexpal (test Regex in JS flavor)](http://www.regexpal.com/)    
<!-- Ruby gsub (and sub)  
Ruby match, matchData Object  
Ruby scan  
JS replace  
JS test  
JS exec  
JS match    -->
[Regex and languages @ regular-expressions.info](http://www.regular-expressions.info/tools.html)   
