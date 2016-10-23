---
layout: post
title: Go To Jails
---

#### Injecting Javascript to Rails   

The fourth assessement in the Flatiron School Fellowship Program requires adding Javascript into one's previous Rails project in the form of AJAX requests and transforming JSON responses into JS Model Objects (hence the nickname _Jails_). This turned out to be the purrfect oppurrtunity to refactor my Interception Rails Game App into a one page view.

#### First Step  

To prevent redirects from the navigation bar, I first captured all click events on the links there, apart from the sign in and sign out-actions. The signing actions are the only occasions where I want the page to refresh, thus inserting the playerId of the logged in user into the window scope for JS access. In `application.html.erb`:
<pre><code class="long"><% if current_player %>
    <%= javascript_tag do %>
      playerId = <%= current_player.id %>;
      playerAlias = "<%= current_player.alias %>";
    <% end %>
  <% end %>
  </code></pre>

Otherwise, all information fetched from the DB should be rendered to its corresponding div on the page without refreshing.

#### Second Step    

The original page layout had minimal structure; the only named box containers were the navigation bar, error/message box for flash-messages from the controllers and a status box showing current player alias and game status. Everything else was rendered onto the main page space according to each view the controllers directed the player to. To be able to build a one-page-view, I created a simple flex-box design and allocated individual spaces to characters, games, the current active game and forms. Switching the visibility of each box on and off is handled via functions in a separate houseKeeping.js file.

#### Third Step   

Asynchronously with the second step, I added active model serializers and JS Model Objects to be able to manipulate the responses provided by database queries into the views. Since the Interception App has multiple has_many relationships, displaying the nested information correctly required custom serializers for organizing location history (character_locations table), player data, and for location data.  


#### Difficulties   

The original Rails app has a few game controller actions that are comprised of several chained redirects. Following these actions turned out to be quite the challenge in JS. The current solution to this is to "fatten" out the controllers a little bit, having them calling multiple actions inside one method before returning one finished JSON response. In the future, this needs some rethinking and refactoring via callbacks or better understanding of how to follow redirects.      

This assessment also pointed out to me how I need to put a lot more work into understanding the asynchronous nature of JS much better than I do now. Since I'm not completely comfortable with callbacks yet, a lot of my JS functions do not quite follow the separation of concerns yet.

#### To Be Added   

Removing the page rendering and redirecting process broke the previously built flash-message displays. As of now, the Jails version of the app has only the crudest of error handling features within the actual ajax calls, and is in need of proper, separate error handling functions.  


##### Repo Links    

[Original Rails App](https://github.com/satub/interception-game-app)     
[One-page-view with JS - branch](https://github.com/satub/interception-game-app/tree/jails)
