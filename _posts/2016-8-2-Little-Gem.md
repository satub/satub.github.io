---
layout: post
title: Little Gem
---

#### Coding My First Gem

This blog post describes musings on first gem. A gem! Sounds luxurious, right?


![diamonds are a girl's best friend](https://upload.wikimedia.org/wikipedia/commons/b/b6/Gentlemen_Prefer_Blondes_Movie_Trailer_Screenshot_(34).jpg)
Marilyn Monroe in Gentlemen Prefer Blondes -- source: Wikipedia


Rubies are a girl's best friend? The verdict is still out on that.   

For this project, I wanted to manipulate data that is not already provided by ubiquitous apps. I also wanted to use a website that would not be riddled with ads. Governmental, organizational, or any other sort of public sector websites are usually good for that. So, I thought it would be cool to see what data I could scrape from the IAEA.org (International Atomic Energy Agency) website. I found the [Power Reactor Information System](https://www.iaea.org/pris/) site.   



![Nuclear Power Reactors, here I come!](http://worldmeets.us/images/homer.simpson.nuke.plant.png)      
_Nuclear Power Reactors, here I come!_


##### First Hurdle

The basic logic for this command line gem would be to let the user choose a country they are interested in to see all the nuclear reactors within that country and then choose a reactor within that country to show more details. Sounds simple enough. However, the PRIS site actually works as a jquery/javascript interface to a database, so each country's homepage links the name of each respective reactor in that country via a javascript action instead of a href-link. Good for presenting data from a database -- not so good for a newbie coder who wants to scrape.    

Luckily, the PRIS homepage lists the reactor names together with reactor ID numbers that provide straight access to reactor data via the reactor pages. The addresses to these pages are simply constructed by adding the ID number to the end of the url-address, eg: 'https://www.iaea.org/PRIS/CountryStatistics/ReactorDetails.aspx?current=1056' where '1056' is the ID for the chosen reactor. Workaround found, time to scrape!



##### Building classes

You can see all the source code here: [https://github.com/satub/nuclear-power-reactors-cli-gem](https://github.com/satub/nuclear-power-reactors-cli-gem)

I chose to build 5 different classes to handle all the methods and data.

* Country and Reactor are self-explanatory models. Each Country has many Reactors. They both utilize mass assignment upon initialization to grant more flexibility, should the website change.

* NPRCLI class provides the run sequence for the user interface, plus several helper methods for interacting with the main class, NuclearPowerReactors, based on user input.

* NPRScraper class is responsible for scraping data from the website and the one to break first should any changes to the website occur.

* NuclearPowerReactors is the main class, that calls on NPRScraper to get data and then feeds that data to the Reactor and Country classes for object creation. It is also responsible for fetching data from the countries and reactors and formatting it for display, so you can found it is rather riddled by little helper methods.

##### Second Hurdle

Upon gaining access to all this cool data via scraping, trying to limit the functionality I wanted to code was really hard. In the end I managed to leave all the fancy methods, such as search mode, for future improvements, or else I'd never finish a first, working version of the code. Keep it to basics and make sure that those work.

##### Third Hurdle

...Speaking of basics, I built the file structure for this project by hand. This led to some complications at the time of bundling it all into a gem. I had to create a dummy gem project to find out what files I was still missing and then manually copy and edit those into the project. Not recommended. Also, the default bundler gem initialization creates an executable called bin/console. I'm still looking into why mine doesn't work. The solution lies somewhere between my unconventional gem building process or multiple ruby installations on my system. I'm suspecting the latter ... the console seems to be looking for gems somewhere else and hence won't start.

##### The TODO-list

Refactor the code. Add a search mode. Scrape owner and operator data on the reactors. Add wiki-links to the reactor model names. Clean double ruby installations from my computer and get that bin/console working. [Feel euphoric.](https://open.spotify.com/track/5oBaUoKtnEl3c7oiI1A3Xl)
