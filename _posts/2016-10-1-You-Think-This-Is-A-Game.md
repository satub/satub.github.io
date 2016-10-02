---
layout: post
title: You Think This Is a Game?
---

#### Coding My First Rails App: Interception      


The idea on this app came to me when I was working through a coding exercise for post creation and editing on a fictional message board. The editing ability for each user was decided, as usual, by the user roles so that the regular users could only edit posts they where the authors of, whereas moderators and admins could edit any post. While testing the functionality of the edits, trying to get the tests pass, I started to allocate user roles according to different fictional universes, having the more powerful or obnoxious characters edit their enemies' posts until I had a whole load of back-and-forth pickering in one post. What if there existed  battle of dominion over posts? What if instead of conquering provinces on a map, battles would be settled on a message board?

This idea is not actually that far off. After all, internet is a crazy place and in a world where even a presidential candidate can rise or fall in the polls affected by their Twitter-wars, celebrities status is counted by their number of followers, and way too many people dream of becoming the next member of the twitterati, aren't we already choosing our winners and losers by those who control all the posts? And thus arose this little conquer-the-map game, Interception, where the locations to capture are represented by posts and successful or failed attempts to edit them.   


#### How it works    

After creating a player account, the user can create a game by choosing a name and number of locations on the map. The locations are generated automatically with random defense value and content. In the next step, the player can either create new characters for the game or choose from existing ones. Once a game has been setup, other players can joinin. Hitting the launch button will divide the locations between the players and randomly assign a starting turn.

The rules for the game actual game are simple: Whoever controls all the locations at the same time is the winner. Each character has a default number of troops in the beginning of the game. Takeover attempts are turn-based; the character of choice attacks a location with user-submittable number of troops. If the number of troops sent is higher than the defense of the target, the takeover will be successful. A new defense value is decided by number of troops remaining after the attack. Only the player in control of a given location knows what the current defense value is.     


#### Notes on the Coding Process    

There are 7 models in this app, 3 of which are many_to_many join tables: player, character, game, location, game_player, game_character, and character_location. The database structure and associations through foreign keys can be depicted like this:
<img src="http://i.imgur.com/CsnKqXA.jpg" style="width: 600px" alt="database table relations">

The game_players table allows players to have multiple games. The game_characters table allows the players to reuse their characters; the available troops for that character to use in takeovers is defined separately for each game through this table. Character_locations table tracks down the whole takeover history for each location.


The signup and session is handled via the Devise gem. Instead of the default user-namespace, these devise routes were set to point to players. To be able to add the player alias into the DB upon signup, an override was generated with a separate registrations controller:  

 <pre><code class="x-long">
 class Players::RegistrationsController < Devise::RegistrationsController

   private

   def sign_up_params
     params.require(:player).permit(:alias, :email, :password, :password_confirmation)
   end

   def account_update_params
     params.require(:player).permit(:alias, :email, :password, :password_confirmation, :current_password)
   end
 end
 </code></pre>

 In addition to the default controllers, the app has controllers for characters, games, and locations, as well as a static controller for the welcome and about pages. Dynamic views are provided for characters, games, and locations. The games and locations controllers have the most actions, and these models have also the most custom methods for handling the gameplay and location takeover actions.       


#### Something New, Something Learned  

This project was the first where I used Postgres database instead of sqlite. Apart from the initialization, the  differences in using it are minimal as the ActiveRecord methods mask the database flavors. However, the default ordering of records when running Model.all is different: instead of always returning the instances sorted by their index, the sorting was affected by any table updates. This caused some initial confusion when building views for all the locations in a given game as they got shuffled every time the content was updated.     

Displaying flash-messages correctly required a few extra steps. A flash message does not survive through a redirect, as it's generally used only to show errors when re-rendering a form. However, it also survives when navigating to another page, eg upon clicking the link to the homepage instead of trying to refill the form that gave the errors. To avoid these lingering flash-errors, I created a housekeeping method in the application controller and a before action call that clears the flash before any action is taken. For the redirects I needed the message to persist, I wrote an override in the appropriate controllers.

_application_controller.rb_
<pre><code class="long">class ApplicationController < ActionController::Base
  protect_from_forgery with: :exception
  before_action :housekeeping

  def housekeeping
    flash.clear
  end  
end
</code></pre>

_games_controller.rb_
<pre><code class="long">class GamesController < ApplicationController
  before_action :housekeeping, except: :status

  def status
    if @game.game_over?
      @game.update(status: "finished", winner: @game.winner_id, turn: nil)
    end
    flash.keep(:message)
    redirect_to game_locations_path(current_game)
  end
</code></pre>

_locations_controller.rb_
<pre><code class="long">
class LocationsController < ApplicationController
  before_action :housekeeping, except: :index
</code></pre>


#### Difficulties   

The domain structure and DB table relations are relatively complex. The original schema included a maps table wrapping the game locations into one unit for possible reuse in different games. There was also an extra table for grouping players' choice of characters in a given game into team. An additional Turn model was included for managing the turns while allowing for the possibility for one player to intercept commands from other players (hence the app name). All the aforementioned tables were removed in a complete domain rewrite almost halfway through the coding process, as the structure was getting way too complex to describe and manage, especially considering the timeframe of this project.  

Another obstacle was to get the validations working correctly for an associated model. The app has a custom nested attribute writer for the character_locations table through the locations model. This table logs down all takeover attempts and takes in two user-submittable attributes: the new message they are attempting to post on the chosen location and the number of troops they are playing in attempt to take over. The validations are simple: the new message cannot be empty and the number of troops must be an integer. However, since the validations were run via model associated to the form, and not by the form model itself, the controller was not aware of validation errors on the associated model and the update function on the location model would still run even when it should have been blocked. In the end, this problem was solved via a private method in the controller, checking for the validity of the params for the nested attributes before running the update action for the location.


#### Dream Features   

As always, the list of dream features is long. The takeover interception functionality is obviously on the list, as that is what gave this app its name. Certain neccessary housekeeping functions have not been implemented yet, for example: deciding when the game has been forfeit due to inactivity of one or all players. Also, the ability to invite other players into a game instead of waiting them to join would increase the playability. And maybe, just maybe, automatic character and message generation based on chosen fandom, eg Lannisters vs Starks from The Game of Thrones...


##### Links

[Project repo on Github](https://github.com/satub/interception-game-app)
