---
layout: post
title: Are we Framework? Or are we Engine?
---

#### Going off the Rails    


![](http://i.imgur.com/QHT8ZdT.png)
[_Monorail cat at Noobshire station in AQW_](http://aqwwiki.wikidot.com/noobshire)

All the time spent on learning [Ruby on Rails](http://rubyonrails.org/), in addition to my off-the-rails time with playing [RenPy](https://www.renpy.org) games has made me wonder: what is meant by a framework, and how does it differ from software engines? Is the difference between engines and frameworks purely semantic, as these terms seem to be used interchangeably? In fact, if you look up [game engines in Wikipedia](https://en.wikipedia.org/wiki/Game_engine), you'll see this definition:

> A game engine is a software framework designed for the creation and development of video games.

However, if you'd take a dive into several programmer's forums, you'll find people disagreeing on this. The rationale behind this it that a framework, like Rails, doesn't provide you content in the sense building the actual web application. Instead, it will provide you with project structure and generator tools by tying together libraries that are optimized for creating the structure where you can fit in your webapp, aka the engine. So, you can fit and build an engine into a framework, but you shouldn't fit or build a framework into an engine. Here's an illustrative image from game-developing-POV [Gamedev Forums at Reddit](https://www.reddit.com/r/gamedev/comments/39qomu/game_library_vs_framework_vs_engine_common_terms/):


![](http://i.imgur.com/8c124Rg.png)   



#### So, what would an example of an engine be like, then? Well, RenPy is one.


RenPy is a visual novel (VN) engine developed with Python. It allows building VN games via simple scripting language. Unlike with Rails, where you have to know Ruby well to build an actual distributable application, generating VNs with RenPy doesn't require one to know Python at all. The graphical user interface guides you through a quick setup and you'll be off writing your own VN game in less than two hours (I timed myself, it took 90 minutes from when I launched the quickstart guide to when I had my first 1-choice-game up). The user uses labels to tag menus, branching points, graphics, and sounds. The engine takes care of rendering the 2D views, playing the music at tagged points and providing simple game menus.


However, to create more in depth gameplay, heavy gaming logic such as RPG fighting functionality, card game logic, user input processing, statpoint checkups, etc, you will have to study Python. Using a software engine without knowing what's happening behind the scenes makes you no more of a programmer than driving a car makes you a mechanic. In this sense, a framework like Rails, gives you at least a little protection from the impostor syndrome, as it doesn't build your code for you. Also, staring at my 90-minute-game I knew painstakingly well that I had no idea how I had produced it. Well, the answer was, I hadn't. RenPy had.




![](http://i.imgur.com/x3uAv6r.png)
RenPy Project Launcher GUI
<br><br>

RenPy script for The Decision:
<pre><code class="long">
# The game starts here.
image bg station = "atlantic_ave_small.jpg"
image bg deadend = "atlantic_ave_tunnel_large.jpg"

image stick pondering = "stick_neutral_squish.png"
image stick agony = "stick_confused_squish.png"

define r = Character('Rat', color="#c8ffc8")
define m = Character('Me', color="#c8c8ff")

label start:
    play music "sub.wav" loop
    scene bg station
    show stick pondering

    "One Day, On the Atlantic Avenue Station..."

    m "Um... should I..."
    m "Should...I really take the D train?"

    show stick agony

    "Silence."
    "I feel so afraid, but finally..."

    show stick pondering

menu:
    "Sure, it can't be \"that\" bad.":
        jump dtrain

    "Hell no! I'll try the R instead":
         jump rtrain

label dtrain:
    m "Let me just take those stairs up"
    jump adversities

label rtrain:
    m "I wonder why it's labeled \"DNR\" (Do Not Resuscitate)"
    jump pitch_black

label adversities:
    play music "rocks.wav" loop
    scene bg deadend
    with dissolve
    show stick agony

    m "This might take awhile..."
    ".:. Bad Ending."

    return

label pitch_black:
    play music "drip.ogg" loop
    scene black
    with dissolve

    "--- years later ---"
    m "I should have taken the bus."
    m "Which way was up again?"
    play sound "rat.ogg"
    r "Squeek!"

    ".:. There is no End."

    return
</code></pre>
<br><br>


![](https://i.imgur.com/p8A8Sgw.png)
Decision Point. Menu design provided by RenPy, background image from [Wikipedia](https://en.wikipedia.org/wiki/Atlantic_Avenue%E2%80%93Barclays_Center_(New_York_City_Subway)), stick figure by [pixabay.com](https://pixabay.com/en/stickman-thinking-worry-confused-310590/)


<br><br>


##### More on Framework vs Engine vs Library:<br>    
[Game framework vs Game engine](http://gamedev.stackexchange.com/questions/31772/what-is-a-game-framework-versus-a-game-engine)   
[GameDev Glossary: Library vs Framework vs Engine](http://www.gamefromscratch.com/post/2015/06/13/GameDev-Glossary-Library-Vs-Framework-Vs-Engine.aspx)   
[Differences between framework, engine, API and libraries](https://www.quora.com/What-are-the-differences-between-framework-engine-API-and-libraries)  
[Difference between an engine and a framework, Stackoverflow](http://stackoverflow.com/questions/5068992/whats-the-difference-between-an-engine-and-a-framework)   
