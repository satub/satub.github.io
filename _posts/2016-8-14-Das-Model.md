---
layout: post
title: Das Model
---

#### Dr. Computerlove  or: How I Failed in Updating Tables and Learned about ActiveModel::Dirty

Sometimes the road to new discoveries can get convoluted. You barely remember why you were looking for the question to 42 to begin with. Or you stare at a new ruby class on your screen like it were the Atlantis. Cool. But the thing is, you never set out to find the Lost City, the Answer to Everything, or even the Softest Toilet Paper Ever - you were just trying to update some tables. Which, of course, failed repeatedly.

So, here I was, in the middle of doing the bonus part of the [Flatiron Bank / sinatra secure password lab](https://github.com/satub/sinatra-secure-password-lab-wdf-000), trying to insert a new balance on to the account with ActiveRecord::Base .update. This is the piece of code I ran in pry:    


<pre><code class="long">patch "/deposit" do
  # session[:withdraw_done] = false
  deposit = params[:deposit].to_f
  new_balance = User.find(session[:user_id]).balance + deposit
  @user = User.update(session[:user_id], :balance => new_balance)
  => binding.pry
  # session[:deposit_succesful] = true
  redirect '/account'
  end
</code></pre>      

But it didn't quite go according to plan. You done goofed:

![](https://i.imgur.com/WJ9NENd.png)   

Didn't I just successfully update the users table? After all, the instance variable @user showed up with the updated balance. Yet, selecting the same user data from the database with .find showed that the balance was still the same!    

As if this wasn't enough, trying to compare the variables on the screen resulted into even weirder output:

![](https://i.imgur.com/SRdS0e6.png)

Clearly, this was leading to nowhere, fast!    

![](http://www.reactiongifs.us/wp-content/uploads/2015/12/interesting_reaction_nightmare_before_christmas.gif)    

The thing is, [the class method .update apparently always returns the resulting object, __REGARDLESS__ of whether it was updated into the table or not!](http://apidock.com/rails/v2.3.8/ActiveRecord/Base/update/class)  How inconvenient. There must have been a data validation that wasn't passing, but I found myself totally clueless as how to solve this issue.   


I chose to attack the problem with the somewhat dangerous instance method #update_attribute(name, value) instead (do not try this at work!). This method by-passes validations; this is certainly __not__ safe for real applications, but good enough for a bonus coding task on a Friday afternoon; at this point I was mainly concerned about testing the '/patch' and '/delete/' routes in the application controller, just to get _something_ working (see <a href="#afternote">Afternote</a>). However, as happy as I was with finally succeeding in using the 'patch' route and the record having been finally saved into the database, I was even more intrigued by this line in the method description:    

[`update_attribute :Updates all the attributes that are dirty in this object.`](http://api.rubyonrails.org/classes/ActiveRecord/Persistence.html#method-i-update_attribute)

What is a dirty attribute? Simply put, when an attribute has been changed, but not saved, the attribute is considered dirty. Further googling landed me to this rather sweaty looking package: [ActiveModel::Dirty](http://api.rubyonrails.org/classes/ActiveModel/Dirty.html)  A Dirty Model? I could practically hear the neurons cracking in my head as audiovisual memory circuit overload ensued:

<img src="http://i.imgur.com/9HnTSuf.gif" style="width: 350px;">    

Now let's get to this best thing since toilet paper. `ActiveModel::Dirty` will allow you to track the changes in any object just like ActiveRecord tracks changes in objects mapped to databases. Illustrious! I just had to try this! Let us enter into the mind of an electronic music fan with less-than-perfect memory. The glorious, but simplified ElectronicMusic class could be:    


<pre><code class="long">require 'active_model'
  class ElectronicMusic
    include ActiveModel::Dirty  

    attr_reader :title, :artist, :tracks
    define_attribute_methods :title, :artist, :tracks

    def initialize(title, artist)
      @tracks = []
      @title = title
      @artist = artist
    end

    def artist=(artist)
      artist_will_change! unless artist == @artist
      @artist = artist
    end

    def title=(title)
      title_will_change! unless title == @title
      @title = title
    end

    def tracks=(track_array)
      tracks_will_change! unless @tracks == track_array
      @tracks = track_array
    end

    def add_track(track)
      tracks_will_change! unless @tracks.include?(track)
      @tracks << track
      @tracks.uniq!
    end

    def delete_track(track)
      tracks_will_change! unless !@tracks.include?(track)
      @tracks.delete_if{|included_track| included_track == track}
    end

    def save
      #save the object where ever you want to save it, eg file or class variable, then invoke ActiveModel::Dirty method:
      changes_applied
    end

    def reload!
      clear_changes_information
    end

    def rollback!
      restore_attributes
    end

  end</code></pre>      


Line `define_attribute_methods :title, :artist, :tracks`  lists all the attributes you want to track. In each method that might change those attributes, you'll have to declare attribute_will_change! e.g. `tracks_will_change!` is declared for tracks= setter method, #add_track(track), as well as #delete_track(track) methods in the above code. Remember that all change-tracking methods are depending on getter and setter methods for each attribute of interest!

Now we shall play with this. :D Let's create a record for _Die Mensch-Maschine_ by _Kraftwerk_. Since we just created it, nobody has had time to tamper with the record:   

<pre><code class="long">

electronica = ElectronicMusic.new("Die Mensch-Maschine","Kraftwerk")
electronica.changed?  => false
</code></pre>

Adding tracks, querying after the changes, and saving those changes:   

<pre><code class="long">
electronica.tracks = ["Das Model", "Neonlicht"] => ["Das Model", "Neonlicht"]
electronica.changed? => true
electronica.changes => {"tracks"=>[[], ["Das Model", "Neonlicht"]]}
electronica.save => {}
electronica.changed? => false</code></pre>

Adding yet another track, and checking for changes:   

<pre><code class="long">

electronica.add_track("Computerliebe") => nil
electronica.changes => {"tracks"=>[["Das Model", "Neonlicht"], ["Das Model", "Neonlicht", "Computerliebe"]]}
</code></pre>

Oops! _Computerliebe_ is from another album! Our less-than-perfect memory mixed this up as there was a UK single released with both _Computer Love_ and _The Model_ on it. No worries! We can take care of this with rollback:

<pre><code class="long">electronica.rollback! => ["tracks"]
electronica => #<ElectronicMusic:0x007fe850ba4808
 @artist="Kraftwerk",
 @changed_attributes={},
 @previously_changed={"tracks"=>[[], ["Das Model", "Neonlicht"]]},
 @title="Die Mensch-Maschine",
 @tracks=["Das Model", "Neonlicht"]></code></pre>

Awesome! We rolled back to the previous save point, the `@changed_attributes` hash is empty! `@previously_changed` hash shows the previously saved changes and the wrong track is gone. What if we would have saved before the rollback? Then the rollback doesn't remove the wrong track any more, but we can use the #delete_track(track) method. This results in the same `@tracks` array, but different change history:   

<pre><code class="long">electronica.add_track("Computerliebe") => nil
electronica.save
electronica.delete_track("Computerliebe") => ["Das Model", "Neonlicht"]
electronica.changes => {"tracks"=>[["Das Model", "Neonlicht", "Computerliebe"], ["Das Model", "Neonlicht"]]}
electronica => #<ElectronicMusic:0x007f9cc9b9f998
 @artist="Kraftwerk",
 @changed_attributes={"tracks"=>["Das Model", "Neonlicht", "Computerliebe"]},
 @previously_changed={"tracks"=>[["Das Model", "Neonlicht"], ["Das Model", "Neonlicht", "Computerliebe"]]},
 @title="Die Mensch-Maschine",
 @tracks=["Das Model", "Neonlicht"]></code></pre>   

How cool is that! Now I have half a mind to go and add ActiveModel::Dirty to a bunch of previous object relationship labs. Unfortunately, our time is limited, so I shall dispel such evil thoughts by watching some German television from the 80s.....

[Watch Das Model by Kraftwerk on ZDF (youtube)](https://www.youtube.com/watch?v=84YCcDY4coU)    


#### Resources   

[ActiveModel Basics](http://guides.rubyonrails.org/active_model_basics.html)      


***************


#### Afternote: <a name="afternote"></a>

I did, in the end, get the balance changes working by adding password authentication also to the updating. The key to understanding what was happening after I got the nudge to the right direction (thank you, Fidel!) was running `@user.errors` in pry, which returned the actual failing part of my .update call:  

`#<ActiveModel::Errors:0x007fe92d809d50
....
 @messages={:password=>["can't be blank"]}>`
