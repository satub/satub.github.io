---
layout: post
title: Math - The Final Frontier
---


#### Math.js: Stepping Beyond the Built-in JavaScript Math Library   

Working with arrays in plain JS has really made me appreciate the little helper methods, such as reverse iteration `array.reverse_each {|x| puts x}` or directly accessing the last element in an array `array.last` or `array[-1]`, available in Ruby. In JS, I've often found myself resorting to nested for loops or writing my own methods to be able to achieve something that seems really basic, such as iterating over a multidimensional array. Imagine if I actually had to manipulate that array in several different ways... But hold on! Turns out there's a library called Math.js that extends the built-in JS Math object. So, time to go beyond just typing in Math.PI!   

Math.js can be used both in browser as well as with Node.js. To install run `npm install mathjs` in your terminal and load the library with `var math = require('mathjs');` on top of your .js file. To load it in your index.html file for the browser, just add `<script src="math.js" type="text/javascript"></script>`   between the `<head></head>` tags.   

Now you can use a load of additional math methods in your file, such as *drumroll*

![learning about matrices](http://img.memecdn.com/matrix_o_262494.jpg)   

#### MATRICES!!

Creating a two-dimensional matrix is as easy as this:

`let mat = math.matrix([[1,1],[2,2],[3,3]]);`  => `mat.size() => [3,2]`

A short notation for creating a matrix of just ones, or some other number (here 7):

`mat = math.ones(math.matrix([3,2]));`   `mat = math.multiply(math.ones(3, 2), 7)` =>

<pre><code class="long">
|1 1|     |7 7|
|1 1| and |7 7|
|1 1|     |7 7|</code></pre>

You could also create a nested array with similar notation, giving an array as input:  

`let arr = math.ones([3,2])` => `[[1,1],[1,1],[1,1]]`

So what is the difference, except for object type? See iteration:

`mat.forEach(function (value, index, obj) {
  console.log('value:', math.multiply(value,2), 'index:', index);
});`   

will print out  
<pre><code class="long">value: 14 index: [0, 0]
value: 14 index: [0, 1]
value: 14 index: [1, 0]
value: 14 index: [1, 1]
value: 14 index: [2, 0]
value: 14 index: [2, 1]</code></pre>   

whereas `arr.forEach(function (value, index, obj) {
  console.log('value:', math.multiply(value,2), 'index:', index);
});`   

will print
<pre><code class="long">value: [14, 14] index: 0
value: [14, 14] index: 1
value: [14, 14] index: 2</code></pre>   

So, with a matrix, you can iterate over all elements in the object and their indices are the multidimensional coordinates.

Accessing an element: `mat.get([1,1])` => `7`
Changing and element: `mat.set([1,1],6)`

The above 2 methods may not seem that magical compared to the regular accessing of elements in a nested array, but resizing a matrix is something else! To add a third column of 7s, just run this: `mat.resize([3,3],7)`, and you'll see  

<pre><code class="long">
|7 7 7|
|7 6 7|
|7 7 7|</code></pre>

Resize does not work on arrays.

#### If Matrices Should Not Happen to Rub Your Buddha...   

Math.js has plenty of other handy methods in addition to matrix manipulation you might want to checkout if you want to same some time in adding mathematics to your JS, for example, simple unit conversions can be really be as simple as this:   

`math.eval('2 inch to m').value;` => `0.0508`  

or create a mathematical function:   

`var fun = math.eval('fun(x,y) = x*sqrt(y)')` => `fun(2,100)` => `20`   

Just to get you started. Now go forth and calculate!

##### References, Resources, Research   

* [mathjs.org](http://mathjs.org/index.html)  
* [JS Math Object](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Math)   
* [Math Notepad](http://mathnotepad.com/)  
* [Matrices on Wikipedia](https://en.wikipedia.org/wiki/Matrix_(mathematics))  
