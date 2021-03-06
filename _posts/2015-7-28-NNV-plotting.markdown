---
layout: post
title:  "A New Nation Votes"
date:   2015-07-28 17:09:50
categories: about nnv r node
published: false
---
It's been a busy summer so you'll have to bear with me here if this post runs a bit long.

I'm fortunate enough to have some professional contact with [Molly Hardy](http://www.twitter.com/mollyhardy), the Digital Humanities Curator at the American Antiquarian Society, and expressed interest in working with some of the AAS' vast holdings. She in turn graciously introduced me to a New Nation Votes datasets and provided insight into their history, depth, and limitations.

A New Nation Votes project is coordinated by both the AAS and Tufts University through a grant from the National Endowment for the Humanities. The project seeks to capture and index voting records from 1787 to 1825. It is a mammoth undertaking.

Although, I'm by no means a scholar of early US history, I was captivated. [Phil Lampi](http://www.neh.gov/humanities/2008/januaryfebruary/feature/the-orphan-scholar) has devoted the better part of his life to diligently cataloguing and organizing this collection. The AAS and Tufts in turn are in the process of producing an amazing set of voting records that provide a unique insight into the history of the Early Republic. While I'm lucky to be playing with this data, it really deserves a much deeper dive than I can provide.

For example, I'm struggling with changing town names and had to limit my scope to New England and New York.  More on that in a moment. First let's talk about how I did what I did.

I approached this project with the goal of emulating the work that has been done with other GIS programs by both graduate and undergraduate students at WPI. Keeping that in mind, I originally hoped to plot county level geographic data points using ArcGIS (primarily because it is free) and intended to port them over to angular/d3.js for a nice visualization.

Luckily, I stumbled upon Lincoln Mullen's amazing resource [*Digital History Methods in R*](http://lincolnmullen.com/projects/dh-r/). If you're looking for an introduction to R with a digital history/DH bent, I cannot recommend this book enough. I use R in my current position as an analyst and familiarized myself with the textual analyses CRAN packages last winter/spring so the progression to using R for its GIS capabilities was an easy transition. Prof. Mullen provides a strong foundation in using R including data manipulation, munging, and a great section on plotting. One of my favorite things about his book, however, is that he frames it all within the realm of history. Furthermore, his CRAN packages provide another strong resource for those looking for large, relatively clean historic datasets. I've been using his  [USAboundaries](https://cran.r-project.org/web/packages/USAboundaries/USAboundaries.pdf) library to map historic county lines.

To begin I headed over to the [NNV site](http://elections.lib.tufts.edu/) and [downloaded](http://dl.tufts.edu/election_datasets) the specific files I wanted to work with -- in this case ME, NH, VT, MA, RI, CT, and NY. I placed each of those files in a folder.


I used the following packages in this exercise:


    library(ggmap)   
    library(dplyr)   
    library(tidyr)   
    library(rgdal)   
    library(ggplot2)   
    library(USAboundaries)    
    library(stringr)    


Please note that USAboundaries needs to be installed using [devtools](https://github.com/hadley/devtools), however, this is a fairly straightforward process and following the directions on the github should yield the desired results.

---

<h2>Cleaning and Geocoding</h2>


With all packages installed, we can begin reading data into R. First set the working directory to your folder with the NNV data and save it as wd. Next, list each file in the folder and read each file in with a 'tab' deliminating columns. Finally, combine each file based on column headings.


    wd <- setwd("####")
    load_data <- function(path) {
        files <- list.files( path = wd)
        tables <- lapply(files, read.csv, sep = "\t")
        do.call(rbind, tables)
    }

    mapdata <- load_data(wd)

Because the column headings are consistent across each of our data sets, R has no problem combining them into one master file. That being said, it helps to perform 'head(mapdata)' or 'str(mapdata)' to check double and triple check your work. With less clean data you may need to go back and edit a heading or falsely insert a column.

 Now that we have our masterfile read in (renamed to 'mapdata'), we need to isolate the unique towns and cities and begin geocoding. To do this we first need to concatinate our town column and state column. This way we'll avoid confusing Google; rather than looking up `Mexico` and ending up with the latitude and longitude of Mexico City, we'll specify `Mexico, Maine` and hopefully get somewhere in Maine.

    towns <- str_c(mapdata$Town, mapdata$State, sep= ", ")

Typing `towns` into the R command line will give you a list of every town and state within the master file. This is too much. We only want the unique town names.

    allT <- (str_trim(unique(sort(towns), stringsAsFactors=FALSE)))

`allT` will give us a sorted list. Now we are getting somewhere and are almost ready to begin geocoding. However, closely looking at the list reveals that the first nine variables are false entries. Removing those will give us a clean data set to work with:

    allTowns <- allT[9:2372]

With that, the fun begins. Lincoln Mullen's textbook provides an excellent introduction to incorporating Google's geocoding API into R using the `ggmap` package and I make use of this as well. The free version of this service limits you to 2500 downloads a day. In order to work with the API you must first convert your vector to a data frame. This will allow you to give it an index and ultimately append coordinates to it. Once converted to a data frame, we'll geocode the data. Please note that this will take some time so go make a cup of coffee. After all data is geocoded, we then bind both data frames together and rename it to `allLocations`

    firstTowns <- data.frame (towns = c (allTowns),stringsAsFactors = FALSE)
    geocodedTowns <- geocode(firstTowns$towns)
    firstTowns <- cbind(firstTowns, geocodedTowns)
    allLocations <- firstTowns

If you're feeling lazy, I've provided a messy data set on my [github](https://github.com/danieljohnevans/NNV-Geocoding). This is for every town, city and location. Please feel free to fork, update, etc. I will be beyond elated if I receive any interest in collaboration or contributions.

I've also provided a cleaner data set for the New England and NY data set we're currently working with there as well.

---

<h2>Merging</h2>


Still with me? For those of you playing along at home, we have a quick bit of column renaming to do before we can plot out points. This will help us out further on down the road when we merge our data back with the masterfile 'mapdata'. Additionally, we'll want to merge our `allLocations` data frame back to our mapdata master file so that we'll be easily able to call on those files later. Finally, if you haven't already, now would be a great time to write out your various files to .CSVs in the event of a computer or program crash.

    write.csv(firstTowns, file= "your/file/here")
    str(mapdata)

    ##merge allLocations to mapdata
    mapdata <- mutate(mapdata,
        location = str_c(mapdata$Town, mapdata$City))
    mapdata <-  mutate(mapdata,
        location = str_c(mapdata$location, mapdata$State, sep= ", "))

    str(allLocations)
    names(allLocations) <- c("x", "location", "lon", "lat")
    str(allLocations)

    mapdata_merged <- left_join(mapdata, allLocations, by = c("location" = "location"))

    str(mapdata_merged)

    write.csv(mapdata_merged, file= "your/file/here")

---

<h2>Mapping</h2>


Now that your coordinates are saved, you can easily import them at your leisure without going through the multitude of data munging steps. With that said, it's finally time to see how things look on a map. A quick note on plotting, the CRAN package `ggplot2` is the most efficient charting tool in R. Charts and graphs in R are finicky. I really can't recommend anything other than to just keep grinding away at them. In the case of maps, I recommend taking a look at Lincoln Mullen's write up on GIS maps. My code is below:


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


If you've followed along thus far, importing all packages and geocoding everything, this is what you'll get:
![NE MAP RAW](/assets/Rplot_raw.png){: .center-image .responsive-image }    

Most likely this is not what you were hoping to see. I've never plotted anything in R and got it right my first time. In this case the changing and variable names of US towns are the problem. Admittedly, the data cleanup and georectification part of this process is taking longer than expected. In many ways, I've chosen to limit my scope to New England and New York due to the overwhelming number of towns in a NNV. I think working with a smaller data set initally and expanding my scope outward after deployment will help to isolate many of the issues I'm facing.

Take for instance the MA/ME split in 1820. All Maine towns in the NNV data set rightfully fell under the jurisdiction of Massachusetts prior to 1820. Ergo, many of the initial errors on the map above can be blamed on Google maps getting confused by places like `Denmark, Massachusetts`. Instead it should be looking for `Denmark, Maine`.

Spelling variations pose another problem. These anomolies are more difficult for me to find and are oftentimes only discovered via manual checks. An obvious example of this is `Chili, New York` verses `Chile, New York`. This registers as two radically different locations for Google's API. However, sometimes Google will plot a variable in another county or state and this isn't immediately apparent.

Finally, I'm struggling with towns simply disappearing from the historical record. Take for example `Phillipe, New York` which, according to the master file is located in Dutchess County. It appears that it was a part of the [Philipse Patent](https://en.wikipedia.org/wiki/Philipse_Patent) and is probably a misspelling. What was once Philipse, Dutchess County, New York was incorporated into [Fishkill, New York](http://www.putnamcountyny.com/countyhistorian/boundary-changes/) after the Revolutionary War and I've georectified to make up for the loss.

These investigations take time and the going is slow. A few miscellaneous notes:

<ul>For whatever reason, CT and RI seem to have relatively static names. I've made few, if any, georectifications due to name changes or spelling variations.</ul>
<ul>Most of my time is devoted to trying to figure out Upstate NY's complicated village system. This information has been difficult to find. </ul>
<ul>This contrasts sharply with ME, NE and VT. They seem to have devoted historians/Wikipedians. Town name changes are diligently noted on Wikipedia pages or easily findable on the web.</ul>

My current iteration plots out to:
![NE MAP Cleaner](/assets/Rplot_clean.png){: .center-image .responsive-image }  

---

<h2>Conclusions</h2>

I'm about a third through the list and hope to finish by the end of the summer. My ultimate goal is to reintroduce these georectified locations into the original NNV dataset and plot historic voting records by town and county on an angular/d3 website. I'd like to then compare those numbers against US Census records to examine voter turnout per town per election. I'll pull the OCR for this data using tesseract as I haven't yet seen an API that documents town level census data back that far. That being said, it'd be great to receive feedback on any part of this process so please feel free to reach out.

The complete code used in this project can be found [here](https://github.com/danieljohnevans/electionsNE).

A quick note about the backend of this site - I've been playing around with node.js. Earlier this summer, I redployed this site under the yeoman-jekyllrb framework but have since reverted to my jekyll bootstrap framework. As a task runner, Grunt.js is giving me more problems than it's solving. Every time I try to deploy using it, I receive a litany of error messages. I know I'll return to this in the coming weeks but my initial thought is that the problem may be in grunt and I may need to look at gulp.js instead.

More soon.
