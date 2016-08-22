---
layout: post
title: Once Upon A Time On The Subway
---

#### Creating the Mystery Riders Webapp   

The idea for this MVC sinatra application came to me as I was experiencing repeated delays and service changes during my commute in New York. So, instead of printing out sarcastic flyers and posting them on the stations (why didn't I think of that?),

<img src="http://gothamist.com/attachments/arts_jen/phpAmeQeoAM.jpg" style="width: 400px;"> <br>
_Fake service advisory posted at the G line in 2009. Source: [Gothamist](http://gothamist.com/2009/04/17/subway_alert.php)_   

 I decided to build a webpage for giving anonymous feedback for each of the NYC subway lines. Source code for this project is available at [Github.](https://github.com/satub/mystery-riders)   

The models for the Mystery Riders app are really simple. There are three classes: Passenger, SubwayLine and RideFeedback. Instances of RideFeedback belong to both a passenger as well as a subway line. Passengers can ride (read: _have_ in ActiveRecord's rubyspeak) multiple subway lines and subway lines have multiple passengers through the ride feedbacks.

Controller tasks are divided between just two classes: ApplicationController deals with login, logout and signup routes as all well requesting subway line information. FeedbackController provides routes for writing feedback to a chosen line, as well as editing and deleting it. Basic checks ensure that only the author of any given feedback can edit or delete it, and that you have to signup and login to write any new feedback. Since this app is not used to add or edit any new subway lines (that job belongs to the [MTA](http://www.mta.info/), yo), I chose not to create a separate controller for the subway line views.

Views are divided into three different categories. The passenger views provide simple login and signup forms, and the subway line views give a list of all subways and a view of a single subway together with all the related comments and average rating for that line. Feedback views provide forms for new feedback for a chosen line and quick buttons to edit a single entry or to delete it.

ActiveRecord is set to validate several things: upon signup, the users will have to choose an alias for posting, give an email address, and choose a password. The app will notify the user if their alias for posting is already taken, or if their password or alias is too short. It also checks that the email address format is correct, but it doesn't check if such an address actually exists.

To get even with the subway lines, I added some extra fun: you can't rate and ride higher than 3 (ActiveRecord will let you know) and if you try to mess around with the system by searching for lines that do not exist by typing unknown letters or numbers straight at the address line, you'll get some extra sassy error messages. :D


#### Afternote   

I do realize that most problems in NYC public transport are caused by two things: the failing infrastructure that is in dire need for repair and funding and some passengers behaving in less-than-smart ways. This blog post nor the associated repo were not written to insult the MTA workers who already have a lot to deal with on a daily basis. 
