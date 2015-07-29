---
layout: post
title:  "A New Nation Votes"
date:   2015-07-28 17:09:50
categories: about NNV R
---
<body>
<p>It's been a busy summer so you'll have to bear with me here if this post runs a bit long. </p>

<p>I'm fortunate enough to have some professional contact with <a href= "www.twitter.com/mollyhardy">Molly Hardy</a>, the Digital Humanities Curator at the American Antiquarian Society, and expressed interest in working with some of the AAS' vast holdings. She in turn graciously introduced me to the New Nation Votes datasets and provided insight into their history, depth, and limitations.</p> 

<p>Although, I'm by no means a scholar of early US history, I was captivated. <a href= "http://www.neh.gov/humanities/2008/januaryfebruary/feature/the-orphan-scholar">Phil Lampi </a> has devoted the better part of his life to this dataset and his diligent work in cataloguing and organizing the collection. The AAS is in the process of producing an amazing set of voting records that provides a unique insight into the history of the Early Republic. While I'm lucky to be playing with this data, it really deserves a much deeper dive than I can provide. </p>

<p> For example, I've struggled with changing town names and had to limit my scope to New England and New York.  More on that in a moment. First let's talk about how I did what I did. </p>

<p> I approached this project with the goal of emulating the work that has been done with other GIS programs by both graduate and undergraduate students at WPI. Keeping that in mind, I originally hoped to plot the points using ArcGIS (primarily because it is free) and intended to port them over to angular/d3.js for a nice visualization. </p>

<p> Luckily, I stumbled upon Lincoln Mullen's amazing resource <a href= "http://lincolnmullen.com/projects/dh-r/">"<i>Digital History Methods in R</i></a>. If you're looking for an introduction to R with a digital history/DH bent, I cannot recommend this book enough. I use R in my current position as an analyst and familiarized myself with the textual analyses CRAN packages last winter/spring so the progression to using R for its GIS capabilities was an easy transition. Prof. Mullen provides a strong foundation in using R including data manipulation, munging, and a great section on plotting. My favorite part about his book, however, is that he frames it all within the realm of history. Furthermore, his CRAN packages provide another strong resource for those looking for large, relatively clean historic datasets. I've been using his <a href = "https://cran.r-project.org/web/packages/USAboundaries/USAboundaries.pdf">'USAboundaries'<a> library to map historic county lines. </p>

<p>To begin I headed over to the <a href = "http://elections.lib.tufts.edu/"> NNV site <a> and downloaded <a href = "http://dl.tufts.edu/election_datasets"> the specific files I wanted to work with <a> -- in this case ME, NH, VT, MA, RI, CT, and NY. I placed each of those files in a folder. </p>

<p> 
I used the following packages in this exercise:

'''R
library(ggmap)
library(dplyr)
library(tidyr)
library(rgdal)
library(ggplot2)
library(USAboundaries)
library(stringr)
'''
Please note that USAboundaries needs to be installed using <a href = "https://github.com/hadley/devtools"> devtools <a>, however, this is a fairly straightforward process.
</p>

<h2>Cleaning and Geocoding</h2>

<p>
With all packages installed, we can begin reading data into R. First set the working directory to your folder with the NNV data and save it as wd. Next, list each file in the folder and read each file in with a 'tab' deliminating columns. Finally, combine each file based on column headings.

'''R
wd <- setwd("####")
load_data <- function(path) { 
files <- list.files( path = wd)
tables <- lapply(files, read.csv, sep = "\t")
do.call(rbind, tables)
}

mapdata <- load_data(wd)
'''

Because the column headings are consistent across each of our data sets, R has no problem combining them into one master file. That being said, I always perform 'head(mapdata)' or 'str(mapdata)' just to check my findings. With less clean data you may need to go back and edit a heading or falsely insert a column.
</p>

<p> Now that we have our masterfile read in (renamed to 'mapdata'), we need to isolate the unique towns and cities and begin geocoding. To do this we first need to concatinate our town column and state column. This way we'll avoid confusing Google; rather than looking up 'Mexico' and ending up with the 'lat' and 'lon' of Mexico City, we'll specify 'Mexico, Maine' and hopefully get somewhere in Maine.
</p>

'''R
towns <- str_c(mapdata$Town, mapdata$State, sep= ", ")
'''
<p>Typing 'towns' will give you a list of every town and state within the master file. This is too much. We only want the unique town names. </p>

'''R
allT <- (str_trim(unique(sort(towns), stringsAsFactors=FALSE)))
'''

<p>'allT' will give us a sorted list. Now we are getting somewhere and are almost ready to begin geocoding. However, closely looking at the list reveals that the first nine variables are false entries. Removing those will give us a clean data set to work with:</p>

'''R
allTowns <- allT[9:2372]
'''

<p>With that, the fun begins. Lincoln Mullen's textbook provides an excellent introduction to incorporating Google's geocoding API into R using the 'ggmap' package and I make use of this as well. The free version of this service limits you to 2500 downloads a day. In order to work with the API you must first convert your vector to a data frame. This will allow you to give it an index and ultimately append coordinates to it. Once converted to a data frame, we'll geocode the data. Please note that this will take some time so go make a cup of coffee. After all data is geocoded, we then bind both data frames together and rename it to 'allLocations' </p>

'''R
firstTowns <- data.frame (towns = c (allTowns),stringsAsFactors = FALSE)
geocodedTowns <- geocode(firstTowns$towns)
firstTowns <- cbind(firstTowns, geocodedTowns)
allLocations <- firstTowns
'''

<p>Done? If you're feeling lazy, I've provided a messy data set on my <a href = "https://github.com/danieljohnevans/NNV-Geocoding"> github.<a> This is for every town, city and location. Please feel free to fork, update, etc. I will be beyond elated if I receive any interest in collaboration or receive any contributions!</p>

<p>I've also provided a cleaner data set for the New England and NY data set we're currently working with <a href = "https://github.com/danieljohnevans/NNV-Geocoding"> here. <a> </p>

<h2>Merging</h2>

<p>Still with me? For those of you playing along at home, we have a quick bit of column renaming to do before we can plot out points. This will help us out further on down the road when we merge our data back with the masterfile 'mapdata'. Additionally, we'll want to merge our 'allLocations' data frame back to our 'mapdata' master file so that we'll be easily able to call on those files later. Finally, if you haven't already, now would be a great time to write out your various files to .CSVs in the event of a computer or program crash you don't want to wait through all of that geocoding again do you?</p>

'''R

##write.csv(firstTowns, file= "####")


##merge allLocations to mapdata

str(mapdata)
mapdata <- mutate(mapdata, 
    location = str_c(mapdata$Town, mapdata$City))
mapdata <-  mutate(mapdata, 
    location = str_c(mapdata$location, mapdata$State, sep= ", "))

str(allLocations)
names(allLocations) <- c("x", "location", "lon", "lat")
str(allLocations)

mapdata_merged <- left_join(mapdata, allLocations, by = c("location" = "location"))
str(mapdata_merged)

## Writes master file:
##write.csv(mapdata_merged, file= "####")
'''

<h2>Mapping</h2>

<p>Now that your coordinates are saved, you can easily import them at your leisure without going through the multitude of data munging steps. With that said, it's finally time to see how things look on a map. A quick note on plotting, the CRAN package 'ggplot2' is the most efficient charting tool in R. Charts and graphs in R are finicky. The best thing to do is to keep working at it. I really can't recommend anything other than to just keep grinding away at them. In the case of maps, I'll recommend taking a look at Lincoln Mullen's write up on GIS maps. My code is below:</p>

'''R
USA <- c("Connecticut","Maine", "Massachusetts", "New Hampshire", 
"New York", "Rhode Island", "Vermont")  
map <- us_boundaries(as.Date("1825-03-15"), type = "county", state = USA)
usMap <- ggplot() +  geom_polygon(data=map, aes(x=long, y=lat, group=group))
usMap +
    ggtitle("County Boundaries on March 15, 1825") +
    geom_text(data = allLocations, aes(x = lon, y = lat, label = location), 
        color="gray",
        vjust = -1,
        size = 4) +
    geom_point(data = allLocations, aes(x = lon, y = lat), color= "red") +
    theme(legend.position = "bottom" ) +
    theme_minimal()
'''

<p>If you've followed along thus far, importing all packages and geocoding everything, this is what you'll get:
![NE MAP RAW]({{ site.url }}/assets/Rplot_raw.png)</p>

<p>Most likely this is not what you were hoping to see. I've never plotted anything in R and got it right my first time. That being said, the changing names of US towns have been difficult for me to georectify. In many ways, I've chosen to limit my scope to New England and New York due to this problem. I think working with a smaller data set initally and expanding my scope from there will help to isolate many of the issues I'm facing. Take for instance the MA/ME split in 1820. All Maine towns in the NNV data set rightfully fell under the jurisdiction of Massachusetts prior to 1820. Ergo, many of the errors on the map above are places like 'Denmark, Massachusetts' which became 'Denmark, Maine' a few years later. However, variations in spelling requires further manual checks. 'Chili, New York' verses 'Chile, New York' registers as two different locations for Google's API. Finally, I'm struggling with towns simply no longer existing. Take for example 'Phillipe, New York' which, according to the master file is located in Dutchess County. It appears that it was a part of the <a href = "https://en.wikipedia.org/wiki/Philipse_Patent> Philipse Patent <a> and is probably a misspelling. What was once Philipse, Dutchess County, New York was incorporated into <a href = "http://www.putnamcountyny.com/countyhistorian/boundary-changes/"> Fishkill, New York <a> after the Revolutionary War and I've georectified to make up for the loss. </p>

<p>That being said, the going is slow. We're currently looking at a cleaner data set that plots out to:
![NE MAP Cleaner]({{ site.url }}/assets/Rplot_clean.png)</p>

<p>I'm about halfway through the list and hope to finish by the end of the summer. Feel free to reach out!</p>


</body>
 


[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help