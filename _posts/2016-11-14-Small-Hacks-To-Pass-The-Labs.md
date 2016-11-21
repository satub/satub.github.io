---
layout: post
title: Small Hacks To Pass The Labs (And Having Fun With It)
---

This blog is a summary of little code snippets and other solutions that I've used to solve small, more-or-less-unrelated problems that arose while I was working my way through the Flatiron School Web Dev Fellowship curriculum.   


##### Colormyspec and The Missing .rspec-file   

Have you ever cloned a github repo with accompanied rspec-tests but no .rspec-file? Have you done it repeatedly? Do you prefer the test output on your terminal to be colorful and showing both passed as well as failed tests instead of just a black blob of text with dots representing success? Have you run the commands to create .rspec-file with two lines: `--format documentation` and `--color` several times a day? Were you sick of it at some point?   

![](https://67.media.tumblr.com/tumblr_m23iepqJnr1qbheumo1_500.gif)   

<p  style="text-align: center; font-style: italic;"> This is not repetitive at all! (Screenshot from The Shining.)</p>  

My solution (this is for mac OS):    


  1.  Create a file named .rspec with the default options in an accessible location in your home/code directory.    

  2.  Choose a shorthand command you'd like to use to access and copy that file: for example, mine is `colormyspec`. This will be the alias that you will add to your .bash_profile.    

  3.  In your home directory, open up the file `.bash_profile` in your any text editor, locate a section for aliases (if there are none, just add a comment line `#Aliases` to the file and type in what you need in a new line after that) and add the following:    
  `alias YOUR_CHOSEN_ALIAS='cp YOUR_PATH_TO_THE_DEFAULT_RSPEC_FILE/.rspec .'`  
  The alias in my .bash_profile looks like this:    
  `alias colormyspec='cp ~/.atom/code/.rspec .'`    

This alias simply copies (cp) the default file (.rspec) from the folder /.atom/code under my home directory (~) into whatever directory I am in (.) when I type the alias (colormyspec) in the terminal. And problem solved! No more creating from scratch .rspec-files for repos that do not have them.



##### Timeout and Mocha-tests That Would Not Run   

Little did I know when I was editing my .bash_profile to take care of missing .rspec files, that I would be using the same pattern to solve another problem a month later as I was acquainting myself with JavaScript. This new problem with running mocha-tests from command line was far more _insidious_, though. Whenever I would clone a new repo, the whole test suite would run only once; any subsequent attempts would lead into timing out on the first test and inability to run no further. Well, that's cute (sort of):


<img src="http://wheresthejump.com/wp-content/uploads/2015/06/insidious-1.jpg" style="width: 350px;">   
 <p  style="text-align: center; font-style: italic;"> Why don't these tests run...Move!! Do something!! (Screenshot from Insidious.)</p>  


 It turned out that mocha-tests have a default timeout of two seconds; if the tests would take longer than that, they would exit without further trials. Why would the first then run? I still haven't found an explanation to that, but at least I found a way to work around the timeout issue as this problem was only occurring in repos where no mocha.opts file existed. So, similarly to my solution with the missing .rspec while, I created a default mocha.opts file with only one line of code in it:
`--timeout 10000` which allows for 10 seconds before exiting tests and added a newline to my .bash_profile:    

`alias timeout='cp ~/.atom/code/mocha.opts ./test/mocha.opts'`    

Now I would simply run `timeout` on my command line on repos where no mocha.opts were defined and gone was the problem of timing out on the first test!



##### A Moment of WTF: ["Jason", "Voorhees"] === ["Jason", "Voorhees"] => False    

As far as horror movies go, we all know that the sequels are rarely as good as the original and that the villain hardly ever is as terrifying the second time as (s)he was in the first feature. Be that as may, you would still think that ["Jason", "Voorhees"] === ["Jason", "Voorhees"] would return true, but it won't. Two arrays are never equal as any new array is always a new object. If you want to compare the contents of two arrays, you could describe your own elegant function that compares the arrays element by element...or you could cheat by asking JSON to string up the objects -so to speak- and compare their innards for you:

  `JSON.stringify(["Jason", "Voorhees"]) === JSON.stringify(["Jason", "Voorhees"] ) => true`  

![Jason Voorhees](https://upload.wikimedia.org/wikipedia/en/0/05/Jasonf.jpg)   
<p  style="text-align: center; font-style: italic;"> Jason from Friday the 13th franchise </p>




##### Adding JQuery to Mocha-tests  

Recently, I ran into [a coding exercise](https://github.com/learn-co-students/react-container-components-lab-wdf-000) where one is required to create a container component with React and set its state according to a result obtained by running a GET -request to the NYT api. The instructions recommended using [fetch()](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) for executing the request, but did not limit the solution to only that. After some fast failures with fetch, I decided to go with the good-old-low-level $.ajax request. It was easy enough to add a JS-library such as the JQuery required to run ajax to my index.html file: I simply added  


`<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.1.1/jquery.min.js"></script>`   

inside the `<head></head>` tags and the webpage functionality was there!  

However, the mocha.tests that were set to test my code's functionality, were done with fetch in mind, and anything api-related ended up in errors declaring:    

` ReferenceError: $ is not defined`   

Mocha-test does not simply read your script tags in the html!!   

<img src="http://apennedpoint.com/wp-content/uploads/2013/06/HAL7-e1370317928395-541x401.png" style="width: 350px;">  

<p  style="text-align: center; font-style: italic;"> (HAL from 2001: A Space Odyssey)</p>  

Well, I could've played it nice and go back to configuring fetch, but since the exercise instructions had given me the free license to use whatever method I wanted for the api-requests, I decided to hack the test instead. So, in the test-folder, open the root.js file that looks like this:   
<pre><code class="x-long">const path = require('path');
const jsdom = require('jsdom').jsdom;
const fs = require('fs');
const { fetch } = require('whatwg-fetch');

const html = fs.readFileSync(path.resolve(__dirname, '..', 'index.html'));
const exposedProperties = ['window', 'navigator', 'document'];

global.expect = require('expect');
global.document = jsdom(html);
global.window = document.defaultView;

Object.keys(document.defaultView).forEach((property) => {
  if (typeof global[property] === 'undefined') {
    exposedProperties.push(property);
    global[property] = document.defaultView[property];
  }
});

global.navigator = {
  userAgent: 'node.js'
};

</code></pre>  
Add the following lines to the file:

`import _$ from 'jquery';`  and
`const $ = _$(global.window);`   

=>
<pre><code class="x-long">...
import _$ from 'jquery';
const fs = require('fs');
const { fetch } = require('whatwg-fetch');

const html = fs.readFileSync(path.resolve(__dirname, '..', 'index.html'));
const exposedProperties = ['window', 'navigator', 'document'];

global.expect = require('expect');
global.document = jsdom(html);
global.window = document.defaultView;
const $ = _$(global.window);
.....
</code></pre>

And now mocha knows what $ is and my JQuery ajax request passes the tests as well! Woot!   


##### Customizing Mocha-reporters   

After surviving what was ~~Election Week 2016~~ this Horror Movie Marathon, let's move into a sweeter territory. While running mocha-tests, I kept on wondering, where did the nyan-cat representing your test success come from?

<img src="https://mochajs.org/images/reporter-nyan.png" style="width: 550px;">    

<p  style="text-align: center;"> Nyan cat reporter @ mochajs.org </p>

Nyan cat is only one of the reporters that come with Mocha.js. You can define the reporter you want to be used in the package.json file. For example, in the containers components repo (link here):

<pre><code class="x-long">{
  "name": "react-container-components",
  "version": "0.1.0",
  "description": "Introduction to using container components to keep and update state",
  "engines": {
    "node": "6.x",
    "npm": "3.x"
  },
  "main": "index.js",
  "scripts": {
    "debug": "node-debug --hidden node_modules _mocha --watch",
    "debug-ide": "./bin/debug-ide",
    "copy-git-hooks": "./bin/copy-hooks",
    "test": "mocha -R mocha-multi --require test/root.js --reporter-options nyan=-,json=.results.json",
    "bundle": "browserify index.js -t babelify --exclude react/addons --exclude react/lib/ExecutionEnvironment --exclude react/lib/ReactContext -o bundle.js",
    "postinstall": "npm run copy-git-hooks"
  },
  "repository": {
    "type": "git",
    "url": "git+ssh://git@github.com/learn-co-curriculum/react-container-components.git"
  },
  "keywords": [
    "javascript",
    "flatiron",
    "learn"
  ],
  "author": "The Flatiron School",
  "license": "SEE LICENSE IN LICENSE.md",
  "bugs": {
    "url": "https://github.com/learn-co-curriculum/react-container-components/issues"
  },
  "homepage": "https://github.com/learn-co-curriculum/react-container-components#readme",
  "devDependencies": {
    "babel-core": "^6.11.4",
    "babel-preset-es2015": "^6.9.0",
    "babel-preset-react": "^6.11.1",
    "babelify": "^7.3.0",
    "browserify": "^13.1.0",
    "enzyme": "^2.4.1",
    "expect": "^1.20.2",
    "jsdom": "^9.4.1",
    "mocha": "^3.0.0",
    "mocha-multi": "^0.9.0",
    "node-inspector": "^0.12.8",
    "react-addons-test-utils": "^15.3.0"
  },
  "dependencies": {
    "node-fetch": "^1.6.3",
    "react": "^15.3.0",
    "react-dom": "^15.3.0",
    "whatwg-fetch": "^1.0.0"
  }
}

</code></pre>


Look for the scripts - key and under key `test` :   

`"test": "mocha -R mocha-multi --require test/root.js --reporter-options nyan=-,json=.results.json"`    

You can switch between different preset reporters by changing the --reporter-options parameter. For example,  

`--reporter-options landing=-, ...`     

will change the nyan cat into an airplane on a landing strip. More options in [Mocha.js documentation](https://mochajs.org/#reporters).

If you so wish, you can also edit the reporters in the node_modules/mocha/reporters directory, and add your own reporters. By playing around with the number of times the test trajectories are printed (`Nyan.prototype.draw`), before adding the cat itself, the trajectory symbols (`Nyan.prototype.appendRainbow`) and nyan cat layout (`Nyan.prototype.drawNyanCat`) in the nyan.js file, I added even more bazazz to my nyan and saved it into a separate bazazzNyan.js file. Now I can use this reporter in any lab by editing the full reporter path into the --reporter options in package.json:


![Nyan cat with an extra long raindow trajectory](http://i.imgur.com/3kCwKkV.png)
_Nyan Cat Pooping An Extra Long Rainbow. Now Try and Trump That! Rainbows FTW!_      


##### Resources

[Mocha](http://mochajs.org/)    
[Make an Alias in Bash Shell](https://coolestguidesontheplanet.com/make-an-alias-in-bash-shell-in-os-x-terminal/)      
