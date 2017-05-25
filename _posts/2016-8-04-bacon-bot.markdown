---
layout: post
title:  "Early Modern Bots"
date:   2016-08-04 17:09:50
categories: sdfb bot
published: false
---

I started at CMU earlier this spring and since joining have been working with the creative minds of the [Six Degrees of Francis Bacon Project](http://www.sixdegreesoffrancisbacon.com/). These folks are mining names from the [Oxford Dictionary of National Biography](http://www.oxforddnb.com/) and creating an early modern social network. It's an amazing project. I recommend checking it out if you haven't already.

Early in the project I spent admittedly too much time clicking on individuals, trying to get a sense of who they were and how they interacted. As someone who doesn't have a background in early modernism I found that I knew most of the big names but few of the smaller ones (embarrassingly I referred to Charles V as a English king in one meeting -- oops). Unfortunately my explorations quickly dead-ended because I only knew a handful of prominent individuals and their social networks tended to be exclusionary.

After sketching out a few possible solutions, I realized that I  wanted to view random relationships of lesser-known figures and do it automatically. I ultimately settled on a twitter bot as twitter is something I check rather frequently and I've been wanting an excuse to build a bot for awhile now. The other requirement for this project was I wanted to dive back into Ruby. My Jekyll site is built with Ruby and the Six Degrees project is built on Rails so I knew it would be a useful exercise.

### The Bot

So the bot by itself is conceptually rather simple. It pulls a random random relationship_id from the Six Degrees of Francis Bacon project. It then looks up the individual_id associated with that relationship and prints out:

1. Information about that relationship while substituting the individual_id for a display name
2. It responds to itself with information about those individuals (birth year, death year, historical significance)

So how are these ids generated?

The bot makes these calls to the database by recognizing a pattern of the url. In the Six Bacon project it follows .com/relationships/# or .com/groups/#. This number will always link back to a unique identifier provided that one exists. Furthermore simply adding .json to nearly any page will provide provide an API version of that page.

I've provided a few examples below to help illustrate my point:

1. The front end of Francis Bacon's relationship to Anne Bacon is [here](http://www.sixdegreesoffrancisbacon.com/relationships/100009300). And the back end version is [here](http://www.sixdegreesoffrancisbacon.com/relationships/100009300.json).

2. The front end of Francis Bacon's profile is [here](http://www.sixdegreesoffrancisbacon.com/people/10000473). The back end is [here](http://www.sixdegreesoffrancisbacon.com/people/10000473.json).


I've provided my code below with some basic documentation for reference:


First install gems and require them. Next plug in your twitter consumer key, consumer secret, access token, access token secret. There are plenty of other guides online and I won't go over all this here. However, if you have questions please reach out.

```
require 'httparty'
require 'bundler/setup'
require 'json'
require 'rubygems'
require 'twitter'

$client = Twitter::REST::Client.new do |config|
    config.consumer_key          = "YOUR_CONSUMER_KEY"
    config.consumer_secret       = "YOUR_CONSUMER_SECRET"
    config.access_token          = "YOUR_ACCESS_TOKEN"
    config.access_token_secret   = "YOUR_ACCESS_TOKEN_SECRET"
end
```
After the setup is completed, generate a random number and plug that number into the relationship url. I should note that while there are gaps in the relationship_ids, an end user could roughly identify the number of 100000000 as the beginning of the relationship_id and 100200719 as the end. Generating a random number in between would provide a positive hit.


Furthermore, we want ruby to recognize that the output is json.

Down the line we'll need to provide a further lookup because the relationship url includes references to individual_ids. To solve this issue, we'll need to provide a parallel lookup. This follows the same format and I've included the code below for simplicity.



```
randos = Hash.new( rand(100000000..100200719) )

class Randomizer
    include HTTParty
    format :json

    def self.find_by_id(id)
        get("http://www.sixdegreesoffrancisbacon.com/relationships/#{id}.json", :query=> {:id => id, :output => "json"})
    end

end

class Personalizer
    include HTTParty
    format :json

    def self.find_by_id(num)
        get("http://www.sixdegreesoffrancisbacon.com/people/#{num}.json", :query=> {:num => num, :output => "json"})
    end

end

```
Next we create hashes that reference various JSON objects. In other words, we're creating key value pair items. If you've worked with dictionaries in python, these are the same thing. If not, we're essentially pulling the json object and morphing into a workable format.  I should note here that if you see Randomizer, I am pulling from the relationship url. If you see Personalizer, I am pulling from the people URL.


The "Go Fish" bit is when you hit a null. go fish

```
rel_id = Hash.new( Randomizer.find_by_id("#{randos[2]}").parsed_response{0}["id"] )
p1 = Hash.new(Randomizer.find_by_id("#{randos[2]}").parsed_response{0}["person1_index"] )
p2 = Hash.new(Randomizer.find_by_id("#{randos[2]}").parsed_response{0}["person2_index"] )

hash = Hash.new("Go Fish")

```

So let's break this down a bit :

```
hash["rel"] = Randomizer.find_by_id("#{randos[2]}").parsed_response{0}["type_certainty_list"]

```
The hash I've created above pulls from the relationship url. It parses the response and looks for the json object "type_certainty_list". This hash produces the certainty number and the relationship type. In the Francis Bacon and Anne Bacon example above, the certainty percentage was 100 and the relationship type was "Parent of".

We've created a few more hashes below. These all mostly from the people information p1 is the first person in the relationship, p2 is the second:


```
## Pulls from person id and outputs person information

hesh = Hash.new("Go Fish")
hesh["person1"] = Personalizer.find_by_id("#{p1[2]}").parsed_response{0}["display_name"]
hesh["person2"] = Personalizer.find_by_id("#{p2[2]}").parsed_response{0}["display_name"]
hesh["hist1"] = Personalizer.find_by_id("#{p1[2]}").parsed_response{0}["historical_significance"]
hesh["hist2"] = Personalizer.find_by_id("#{p2[2]}").parsed_response{0}["historical_significance"]
hesh["by1"] = Personalizer.find_by_id("#{p1[2]}").parsed_response{0}["ext_birth_year"]
hesh["by2"] = Personalizer.find_by_id("#{p2[2]}").parsed_response{0}["ext_birth_year"]
hesh["dy1"] = Personalizer.find_by_id("#{p1[2]}").parsed_response{0}["ext_death_year"]
hesh["dy2"] = Personalizer.find_by_id("#{p2[2]}").parsed_response{0}["ext_death_year"]
hesh["by1_T"] = Personalizer.find_by_id("#{p1[2]}").parsed_response{0}["birth_year_type"]
hesh["by2_T"] = Personalizer.find_by_id("#{p2[2]}").parsed_response{0}["birth_year_type"]
hesh["dy1_T"] = Personalizer.find_by_id("#{p1[2]}").parsed_response{0}["death_year_type"]
hesh["dy2_T"] = Personalizer.find_by_id("#{p2[2]}").parsed_response{0}["death_year_type"]

```

Next we had to substitute date information. [Jessica Otis](https://twitter.com/jotis13) noted that there is quite a bit of uncertainty with our dates and I simply added that information in and swapped out the abbrevations:

```

##six modifiers (“AF” “BF” “CA” “IN” “AF/IN” “BF/IN”)
##which are abbreviations for (“after” “before” “circa” “in” “after or in” and “before or in”).

replacements = [ ["AF", "after"], ["BF", "before"], ["CA", "circa"], ["IN", "in"], ["AF/IN", "after or in"],
    ["BF/IN", "before or in"] ]
replacements.each {|replacement| hesh["by1_T"].gsub!(replacement[0], replacement[1])}

replacements = [ ["AF", "after"], ["BF", "before"], ["CA", "circa"], ["IN", "in"], ["AF/IN", "after or in"],
    ["BF/IN", "before or in"] ]
replacements.each {|replacement| hesh["by2_T"].gsub!(replacement[0], replacement[1])}

replacements = [ ["AF", "after"], ["BF", "before"], ["CA", "circa"], ["IN", "in"], ["AF/IN", "after or in"],
    ["BF/IN", "before or in"] ]
replacements.each {|replacement| hesh["dy1_T"].gsub!(replacement[0], replacement[1])}

replacements = [ ["AF", "after"], ["BF", "before"], ["CA", "circa"], ["IN", "in"], ["AF/IN", "after or in"],
    ["BF/IN", "before or in"] ]
replacements.each {|replacement| hesh["dy2_T"].gsub!(replacement[0], replacement[1])}

```

Next we construct the tweets from the hashes:

```

option = Hash.new( "The @6bacon community is " + hash["rel"][0][1].floor.to_s + "% certain that " + hesh["person1"] + " " +
    hash["rel"][0][2].downcase + " " + hesh["person2"] + ": http://www.sixdegreesoffrancisbacon.com/relationships/#{rel_id[0]}" )

option1 = Hash.new( "The @6bacon community is " + hash["rel"][0][1].floor.to_s + "% certain that " + hesh["person1"] + " was " +
    hash["rel"][0][2].downcase + " " + hesh["person2"] + ": http://www.sixdegreesoffrancisbacon.com/relationships/#{rel_id[0]}" )

option2 = Hash.new( "The @6bacon community is " + hash["rel"][0][1].floor.to_s + "% certain that " + hesh["person1"] + " was a " +
    hash["rel"][0][2].downcase + " " + hesh["person2"] + ": http://www.sixdegreesoffrancisbacon.com/relationships/#{rel_id[0]}" )

option3 = Hash.new( "The @6bacon community is " + hash["rel"][0][1].floor.to_s + "% certain that " + hesh["person1"] + " was an " +
    hash["rel"][0][2].downcase + " " + hesh["person2"] + ": http://www.sixdegreesoffrancisbacon.com/relationships/#{rel_id[0]}" )

h = Hash.new("I am a bot. Please see more at : www.sixdegreesoffrancisbacon.com/")
h["who1"] = ( hesh["person1"] + ", " + hesh["hist1"] + ", was born " + hesh["by1_T"] + " " + hesh["by1"].to_s +
    " and died " + hesh["dy1_T"] + " " + hesh["dy1"].to_s + ": http://www.sixdegreesoffrancisbacon.com/people/#{p1[0]}" )
h["who2"] = ( hesh["person2"] + ", " + hesh["hist2"] + ", was born " + hesh["by2_T"] + " " + hesh["by2"].to_s +
    " and died " + hesh["dy2_T"] + " " + hesh["dy2"].to_s + ": http://www.sixdegreesoffrancisbacon.com/people/#{p2[0]}" )

```

Finally there is some code that is still in progress which is hosted on [my github](https://github.com/danieljohnevans/6bacon_bot).

I should also note that the bot is freely hosted on heroku and updates once an hour.

To do:

- Get the bot to respond to users
- Pull from something other than first relationship.
- Focus on particular demographics / time periods / locations / groups.


### What I've Learned

To be honest I've been surprised with the conversation this bot has generated. I think it has raised important questions about certainty in historical data, open data and what role self-exposing error should play in a project. I'll let more qualified persons than myself discuss those points.

However, I am pleased with the bot and where it is going. I've learned quite a bit and would like to continue adding features to it so long as it continues to spark discussion.

DJE
