---
layout: post
title: The SQL and The AR (ActiveRecord)
---

#### Executing Complex Queries with ActiveRecord
<audio controls>
  <source src="http://www.televisiontunes.com/uploads/audio/The%20Young%20and%20The%20Restless%20-%20Full.mp3" type="audio/mp3">
  <source src="https://ia600308.us.archive.org/29/items/NadiasThemeFromTheYoungAndTheRestless/RecordingAug82012101030Am.ogg" type="audio/ogg">
  Your browser does not support this audio element.
</audio>

Learning new ways to communicate with data and computers is always exiting, but not without pitfalls. When the path is not straightforward, typing simple functions feels particularly frustrating if you already have mastered an alternate way to accomplish the task, but are suddenly forced to change your approach. This happened to me while doing [the Model Class Methods lab in the Full Stack Webdev Curriculum](https://github.com/learn-co-students/model-class-methods-lab-wdf-000). I could find the information needed by typing raw SQL queries, but the preset tests required for the answers to be returned as ActiveRecord:Relation-type objects.  

While tedious to compose, SQL syntax is rather easy to follow even when the query encompasses several join tables and grouping or aggregate functions. The logic of building an SQL query is always the same: `SELECT what you want FROM where you want JOINING them with tables ON foreign keys and if you need to GROUP the results state what criteria they are supposed to be HAVING.` For example:

`SELECT * FROM table INNER JOIN other_table ON table.id = other_table.table_id INNER JOIN another_table ON other_table.id = another_table.other_table_id GROUP BY table.id HAVING COUNT(table.column) = N`

Now, I was trying to chain AR queries together to get the same results and felt as if AR refused to listen to my orders.

<img src="http://www.snapstream.com/images/enterprise/youngrestless.gif" style="width: 600px;">

AR query language works by injecting bits of SQL into the query string to be built. To be able to understand how to translate your SQL query into an AR format, you need to know the correct method name for each SQL - insertion method as well as be vary of distincting between AR calculations methods and query methods. The latter return Fixnum (.average, .sum, .minimum, .maximum, and .count methods), Hash(.count method when called after .group method), or Array(.ids and .pluck) objects, and thus have to be placed last in the query chain, even when it would linguistically make no sense.

The most common AR query methods are .where, .where.not, .limit, .order, .group, .having and .joins. These all return AR relation objects that you can use to chain further queries together. The .joins method executes inner joins for you; but remember that its functionality is wholly dependent on you defining the correct associations in your AR models.

The find_by_sql that allows you to write your whole query in raw SQL is an exception from the above queries:
<blockquote>"The results will be returned as an array with columns requested encapsulated as attributes of the model you call this method from. If you call Product.find_by_sql then the results will be returned in a Product object with the attributes you specified in the SQL query.

If you call a complicated SQL query which spans multiple tables the columns specified by the SELECT will be attributes of the model, whether or not they are columns of the corresponding table."</blockquote>

OK, lets get to examples! I created a database on some of the characters in the CBS soap _The Young and The Restless_ -- after all, what else would serve better as a domain with complex has_many associations and relationships than a daytime soap! :D



<img src="http://vignette3.wikia.nocookie.net/theyoungandtherestless/images/5/57/Mariah_jealous.gif/revision/latest?cb=20160118162151" style="width: 350px;">  

Schema for soap:
<pre><code class="x-long">ActiveRecord::Schema.define(version: 20160925160152) do

create_table "character_relationships", force: :cascade do |t|
  t.integer "character_b_id"
  t.integer "character_a_id"
  t.string  "relationship"
  t.string  "status"
end

  create_table "characters", force: :cascade do |t|
    t.string "first_name"
    t.string "last_name"
    t.string "description"
    t.text   "bio"
  end

  create_table "families", force: :cascade do |t|
    t.string "name"
    t.string "business"
  end

  create_table "family_characters", force: :cascade do |t|
    t.integer "family_id"
    t.integer "character_id"
  end

  create_table "relationships", force: :cascade do |t|
    t.string "type"
    t.string "description"
  end

end
</code></pre>

Adding some data:
<pre><code class="x-long">sqlite> select * from families;
id    name        business                                          
----  ----------  --------------------------------------------------
1     Abbott      Jabot Cosmetics                                   
2     Newman      Newman Enterprises                                
3     Winters     The Abbott - Winters Foundation                   
4     Chancellor  Chancellor Industries                             
5     Ashby       Australian Mob            

sqlite> select id,first_name,last_name,description,alive from characters;
id    first_name    last_name     description                                                             alive
----  ------------  ------------  ----------------------------------------------------------------------  ------
1     Jack          Abbott        Head of the Abbott Family since his father John Abbott died             t     
2     Billy         Abbott        The Black Sheep                                                         t     
3     Ashley        Abbott        Head Chemist of Jabot Cosmetics                                         t     
4     Victor        Newman        Head and Patriarch of the Newman Family and Newman Enterprises          t     
5     Nikki         Newman        The Woman who stands by Victor                                          t     
6     Cane          Ashby         Runaway Australian - The Character that was too popular to die          t     
7     Abby          Rayburn       Warm-hearted but spoiled daughter at the Abbott-Newman crossroads       t     
8     Phyllis       Abbott        Fierce Campaign Manager and Webmistress who goes by the nickname Red    t     
9     Nicholas      Newman        The Good Newman                                                         t  


sqlite> select * from family_characters;
id    family_id     character_id
----  ------------  ------------
1     1             1           
2     1             2           
3     4             2           
4     1             3           
5     2             4           
6     2             5           
7     3             6           
8     4             6           
9     5             6           
10    1             7           
11    2             7           
12    1             8           
13    2             8           
14    1             9        

sqlite> select * from character_relationships;
id    character_b_id   character_a_id   relationship     status      
----  ---------------  ---------------  ---------------  ------------
1     4                1                enemies          on          
2     4                8                enemies          on          
3     5                4                married          on          
4     1                8                married          complicated
5     2                8                affair           complicated
6     9                8                married          off         
7     8                9                affair           off         
8     2                1                half-siblings    complicated
9     3                7                daughter         on          
10    4                7                daughter         on          
11    3                4                affair           off         
12    5                1                affair           off         
13    5                1                friends          on          
14    3                1                siblings         on          
15    4                9                son              on          
16    5                9                son              on         
</code></pre>



Let's find all characters that are enemies. First using command line prompt SQL:
  <pre><code class="long">sqlite> SELECT first_name,last_name  from characters INNER JOIN character_relationships on characters.id = character_relationships.character_a_id OR characters.id = character_relationships.character_b_id WHERE character_relationships.relationship = "enemies";
first_name    last_name   
------------  ------------
Jack          Abbott      
Victor        Newman      
Victor        Newman      
Phyllis       Abbott   
</code></pre>

Executing the same SQL in rails console with find_by_sql:
  <pre><code class="long">Character.find_by_sql 'SELECT first_name,last_name  from characters INNER JOIN character_relationships on characters.id = character_relationships.character_a_id OR characters.id = character_relationships.character_b_id WHERE character_relationships.relationship = "enemies"'

   => [#<Character id: nil, first_name: "Jack", last_name: "Abbott">, #<Character id: nil, first_name: "Victor", last_name: "Newman">, #<Character id: nil, first_name: "Victor", last_name: "Newman">, #<Character id: nil, first_name: "Phyllis", last_name: "Abbott">]
</code></pre>
As you can see, this returns an array of model objects.

Next, we'll use an AR query:
<pre><code class="long">Character.select(:first_name,:last_name).joins(:character_relationships).where("character_relationships.relationship = ?","enemies")

 => #<ActiveRecord::Relation [#<Character id: nil, first_name: "Jack", last_name: "Abbott">, #<Character id: nil, first_name: "Phyllis", last_name: "Abbott">]>

 Character.select(:first_name,:last_name).joins(:reverse_character_relationships).where("character_relationships.relationship = ?","enemies")

 => #<ActiveRecord::Relation [#<Character id: nil, first_name: "Victor", last_name: "Newman">, #<Character id: nil, first_name: "Victor", last_name: "Newman">]>
</code></pre>
Since we have defined the has_many and belongs_to relationship between characters as unidirectional, we have to execute the second query to find which person is on the receiving end of the animosity. Since the result type is an AR Relation in the last example, you can use AR Calculations methods on it, and, for example, pluck off the names into an array:

<pre><code class="long">Character.select(:first_name,:last_name).joins(:character_relationships).where("character_relationships.relationship = ?","enemies").pluck(:first_name,:last_name)
 => [["Jack", "Abbott"], ["Phyllis", "Abbott"]]
</code></pre>

Looks good, right? Now, let's throw more tables into the fray and find all the families harboring animosity!



  SQL command prompt:
  <pre><code class="long">sqlite> SELECT families.name FROM families INNER JOIN family_characters ON families.id = family_characters.family_id INNER JOIN characters ON family_characters.character_id = characters.id INNER JOIN character_relationships ON characters.id = character_relationships.character_a_id  OR characters.id = character_b_id WHERE character_relationships.relationship = "enemies" GROUP BY families.name;
  name        
  ------------
  Abbott      
  Newman          
</code></pre>

Using AR query:
  <pre><code class="long">Family.select(:name).joins(:character_relationships).where("character_relationships.relationship = ?","enemies").group("name")
 => #<ActiveRecord::Relation [#<Family id: nil, name: "Abbott">, #<Family id: nil, name: "Newman">]>
 Family.select(:name).joins(:reverse_character_relationships).where("character_relationships.relationship = ?","enemies").group("families.name")
 => #<ActiveRecord::Relation [#<Family id: nil, name: "Newman">]>
</code></pre>

<img src="http://wwwimage.cbsstatic.com/base/files/asset/10/00/28/00/were_great.gif" style="width: 600px;">

Moving on...to some calculations. Remember that you have to call these methods last with AR queries. Here's how to find the character who is a member in most of our listed families:

With RAW SQL:
  <pre><code class="long">sqlite> SELECT characters.first_name, characters.last_name, COUNT(families.name) AS member_of FROM characters INNER JOIN family_characters ON family_characters.character_id = characters.id INNER JOIN families ON families.id = family_characters.family_id GROUP BY characters.first_name ORDER BY member_of DESC LIMIT 1;
first_name    last_name     member_of   
------------  ------------  ------------
Cane          Ashby         3           
</code></pre>

With AR Query and Calculations:
  <pre><code class="long">Character.select(:first_name).joins(:families).group("characters.first_name").order('count_families_name desc').limit(1).count('families.name')

   => {"Cane"=>3}
</code></pre>

And lastly, we shall demonstrate .having by finding all the characters that belong to at least 2 families. Raw SQL:
<pre><code class="long">sqlite> SELECT characters.first_name, characters.last_name, COUNT(families.name) AS member_of FROM characters INNER JOIN family_characters ON family_characters.character_id = characters.id INNER JOIN families ON families.id = family_characters.family_id GROUP BY characters.first_name HAVING member_of > 1;
first_name    last_name     member_of      
------------  ------------  ---------------
Abby          Rayburn       2              
Billy         Abbott        2              
Cane          Ashby         3              
Phyllis       Abbott        2     
</code></pre>
And same in AR:
<pre><code class="long">Character.select(:first_name,:last_name).joins(:families).group("characters.first_name","characters.last_name").having('count_families_name  > 1').count('families.name')
 => {["Abby", "Rayburn"]=>2, ["Billy", "Abbott"]=>2, ["Cane", "Ashby"]=>3, ["Phyllis", "Abbott"]=>2}
</code></pre>

 Weird Grammar, eh? .count returns a Fixnum, which means that you have to group your query by stating .having first:
 whereas in SQL, you are selecting the count so that comes first. In short summary, ActiveRecord method chaining does not completely follow the sequence of SQL queries. You have chain your methods based on the return values.


Sometimes, you'll just have to deal with the new syntax and learn to read it right:

<img src="http://wwwimage.cbsstatic.com/thumbnails/photos/files/asset/10/00/11/40/cbs_yr_10662_clip2_555515_740.gif" style="width: 600px;">



#####  Resources:   

[http://guides.rubyonrails.org/active_record_querying.html](http://guides.rubyonrails.org/active_record_querying.html)
[http://api.rubyonrails.org/classes/ActiveRecord/Querying.html](http://api.rubyonrails.org/classes/ActiveRecord/Querying.html)
[http://api.rubyonrails.org/classes/ActiveRecord/Calculations.html](http://api.rubyonrails.org/classes/ActiveRecord/Calculations.html)
[http://api.rubyonrails.org/classes/ActiveRecord/Relation.html](http://api.rubyonrails.org/classes/ActiveRecord/Relation.html)
[http://apidock.com/rails/v4.0.2/ActiveRecord/QueryMethods/having](http://apidock.com/rails/v4.0.2/ActiveRecord/QueryMethods/having)
[http://www.w3schools.com/sql/sql_having.asp](http://www.w3schools.com/sql/sql_having.asp)
