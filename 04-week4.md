# Performing spatial operations in R

By now you have come a long way in terms of taking your spatial data, and visualising it using maps, and being able to present the values of a variable using thematic maps. You have had some practice in taking data which has a spatial component, and joining it to a shapefile, using the common column, in order to be able to visually demonstrate variation on something, such as the crime rate, across space. 

I hope that you are finding this to be really exciting stuff, and an opportunity to get yourselves accustomed to spatial data. If there is anything you are unsure about, or want to catch up on, please do not hesitate to revisit older material, and ask us questions about it. We build on each week acquiring knowledge umulatively, so don't let yourself get stuck anywhere down the line. But, if you're ready, today we will go a step further, and get your hands dirty with **spatial manipulation of your data**. 

Thus far, our data manipulation exercises were such that you might be familiar with, from your earlier exposures to data analysis. Linking datasets using a common column, calculating a new variable (new column) from values of existing variables, these are all tasks which you can perform on spatial or non-spatial data. However today we will explore some exercises in data manipulation which are specific to *spatial* data analysis. After this session you can truly say you are masters of spatial data manipulation. So let's get started with that!

The main objectives for this session are that by the end you will have:

- used **geocoding** methods to translate postcodes into geographic coordinates 
- made interactive point map with leaflet
- met a new format of spatial shape file called **geojson**
- subset points that are within a certain area using a **spatial operation**
- created new polygons by generating **buffers** around points
- counted the number of points that fall within a polygon (known as **points in polygon**)

These are all very useful tools for the spatial crime analyst, and we will hope to demonstrate this by working through an example project, where you would make use of all of these tools. 

Let's consider the assumption that licenced premises which serve alcohol are associated with increased crimes. We might have some hypotheses about why this may be. 

One theory might be that some of these serve as *crime attractors*. 

> Crime attractors are particular places, areas, neighbourhoods, districts which create well-known criminal opportunities to which strongly motivated, intending criminal offenders are attracted because of the known opportunities for particular types of crime. Examples might include bar districts; prostitution areas; drug markets; large shopping malls, particularly those near major public transit exchanges; large, insecure parking lots in business or commercial areas. The intending offender goes to rough bars looking for fights or other kinds of 'action'. 

On the other hand, it is possible that these areas are *crime generators*. 

> Crime generators are particular areas to which large numbers of people are attracted for reasons unrelated to any particular level of criminal motivation they might have or to any particular crime they might end up committing. Typical examples might include shopping precincts; entertainment districts; office concentrations; or sports stadiums. 

(If you are interested further in crime attractors vs crime generators I recommend a read of [Brantingham, P., & Brantingham, P. (1995). Criminality of place. European journal on criminal policy and research, 3(3), 5-26.](https://link.springer.com/content/pdf/10.1007/BF02242925.pdf))


It's possible that some licensed premises attract crimes, due to their reputation. However it is also possible that some of them are simply located in areas that are busy, attracts lots of people for lots of reasons, and crimes occurr as a result of an abundance of opportunities instead. 

In any case, what we want to do is to examine whether certain outlets have more crimes near them than others. We can do this using open data, some R code, and the spatial operations discussed above. So let's get to it!


## Getting some (more) data

Manchester City Council have an [Open Data Catalogue](http://open.manchester.gov.uk/open/homepage/3/manchester_open_data_catalogue) on their website, which you can use to browse through what sorts of data they release to the public. There are a some more and some less interesting data sets made available here. It's not quite as impressive as the open data from some of the cities in the US such as [New York](https://opendata.cityofnewyork.us/) or [Dallas ](https://www.dallasopendata.com/) but we'll take it. 


One interesting data set, especially for our questions about the different alcohol outlets is the [Licensed Premises](http://www.manchester.gov.uk/open/downloads/file/169/licensed_premises) data set. This details all the currently active licenced premises in Manchester. You can see there is a link to download now. 


As always, there are a few ways you can download this data set. On the manual side of things, you can simply right click on the download link from the website, save it to your computer, and read it in from there, by specifying the file path. Remember, if you save it in your *working directory*, then you just need to specify the file name, as the working directory folder is where R will first look for this file. If however you've saved this elsewhere, you will need to work out the file path. 


*Note* my favourite shortcut to finding the file path is to simply run the `file.choose()` function, and use the popup window to navigate to the file. When you open this file through the popup window, if you're not assigning this to an object, it will simply print out the filepath to your console window. Like so: 



![](img/file_choose_path.png)



You can then copy and paste this path to whatever fuction you are assigning it to, to read in your data. 



### Reading data in from the web


But, programmers are lazy, and the whole point of using code-based interfaces is that we get to avoid doing unneccessary work, like point-and-click downloading of files. And when data exists online in a suitable format, we can tell R to read the data in from the web directly, and cut out the middle man (that being ourseves in our pointing-and-clicking activity). 


How can we do this? Well think about what we do when we read in a file. We say, hello R, i would like to create a new object please and I will call this new object `my_data`. We do this by typing the name we are giving the object and the assignment function `<-`. Right? Then on the right hand side of the assignment function, there is the value that we are assigning the variable. So it could be a bit of text (such as when you're creating a `shp_name` object and you pass it the string `"path to my file"`), or it could be some function, for example when you read a csv file with the `read.csv()` function. 


So if we're reading a csv, we also need to specity *where* to read the csv from. Where should R look to find this data? This is where normally you are putting in the path to your file, right? Something like: 



```r
my_data <- read.csv("path to my file here")
```


Well what if your data does not live on your laptop or PC? Well, if there is a way that R can still access this data just by following a path, then this approach will still work! So how can we apply this to getting the Licensed Premises data from the web? 


You know when you right click on the link, and select "Save As..." or whatever you click on to save? You could, also select "Copy Link Address". This just copies the webpage where this data is stored. Give it a go! Copy the address, and then paste it into your browser. It will take you to a blank page where a forced download of the data will begin. So what if you pasted this into the `read.csv()` function? 



```r
my_data <- read.csv("www.data.com/the_data_i_want")
```



Well in this case, the my_data object would be assigned the value returned from the read.csv() function reading in the file from the url you provided. File path is no mysterious thing, file path is simply the *path* to the *file* you want to read. If this is a website, then so be it. 


So without dragging this on any further, let's read in the licensed premises data directly from the web: 



```r
lic_prem <- read.csv("http://www.manchester.gov.uk/open/download/downloads/id/169/licensed_premises.csv")
```


You can always check if this worked by looking to your global environment on the righ hand side and seeing if this 'lic_prem' object has appeared. If it has, you should see it has 65535 observations (rows), and 36 variables (columns). 


Let's have a look at what this data set looks like. You can use the `View()` function for this: 



```r
View(lic_prem)
```


We can see there are some interesting and perhaps less interesting columns in there. There are quite a lot of venues in this list as well. Let's think about subsetting them.  Let's say we're interested in city centre manchester. We can see that there is a column for postcodes. We know (from our local domain knowledge) That city centre postcodes are M1-M4. So let's start by subsetting the data to include these. 


### Subsetting using pattern matching

We could use spatial operations here, and geocode all the postcodes at this point, then use a spatial file of city centre to select only the points contained in this area. The only reason we're not doing this is because the geocode function takes a bit of time to geocode each address. It would only be about 10 - 15 minutes, but we don't want to leave you sitting around in the lab for this long, so instead we will try to subset the data using pattern matching in text. In particular we will be using the `grepl()` function. This function takes a **pattern** and looks for it in some text. If it finds the pattern, it returns TRUE, and if it does not, it returns FALSE. So you have to pass two parameters to the `grepl()` function, one of them being the pattern that you want it to look for, and the other being the object in which to search. 

So for example, if we have an object that is some text, and we want to find if it contains the letter "a", we would pass those inside the grepl() function, which would tell us TRUE (yes it's there) or FALSE (no it's not there):




```r
some_text <- "this is some text that has some letter 'a's"

grepl("a", some_text)
```

```
## [1] TRUE
```


You can see this returns TRUE, because there is at least one occurrence of the letter a. If there wasn't, we'd get FALSE: 



```r
some_text <- "this is some text tht hs some letters"

grepl("a", some_text)
```

```
## [1] FALSE
```


So we can use this, to select all the cases where we find the pattern "M1 " in the postcode. *NOTICE* the space in our search pattern. It's not "M1" it's "M1 ". Can you guess why?


Well, M1 will be found in M1 but also in M13, which is the University of Manchester's postcode, and not the part of city centre in which we are interested. 


So let's subset our data by creating a new object `city_centre_prems`, and using the piping (`%>%`) and `filter()` functions from the `dplyr` package:



```r
#remember to load dplyr package if you haven't already: 
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
#then create the city_centre_prems object:
city_centre_prems <- lic_prem %>%
  filter(grepl("M1 ", POSTCODE) )
```


Now we only have 353 observations (see your global environment), which is a much more manageable number. 


### Geocoding from an address


Great OK so we have this list of licensed premises, and we have their address, which is clearly *some* sort of spatial information, but how would you put this on a map? 


Any ideas? 



We can use the `geocode()` function from the `ggmap` package to turn our addresses into mappable coordinates. `geocode()` geocodes a location (find latitude and longitude) using either (1) the [Data Science Toolkit](http://www.datasciencetoolkit.org/about) or (2) Google Maps. Note that when using Google you are agreeing to the Google Maps API Terms of Service at [https://developers.google.com/maps/terms](https://developers.google.com/maps/terms) (this link is also loaded when you load the ggmap package). One of the conditions of using Google API to geocode your points is that you *have to* display them using a Google basemap for example! Also, to use the Google Maps API you need to get an API key, which we won't be messing around with today. Instead, we will use the Data Science Toolkit approach. 


We can, at the most basic, geocode the postcode. This will put all the establisments to the centroid of the postcode. Postcodes are used in the United Kingdom as alphanumeric codes, that were devised by Royal Mail. A full postcode is known as a "postcode unit" and designates an area with a number of addresses or a single major delivery point. [You can search the Royal Mail for information on post codes here.](https://www.royalmail.com/business/search/google/POSTCODE). 

Here is a map of the postcode areas in Greater Manchester: 

![](img/750px-M_postcode_area_map.svg.png)


Now the centroid of the post code area represents the central point of the shapefile. For example, here you can see some polygons with their centroids illustrated by points: 


![](img/6R9sn.png)



This is not quite as precise as geocoding the actual address, and we will return to this in the homework, but let's just stick with this approach for now.



So `geocode()` will help us get the coordinates for the relevant post code centroid. First though, we have to specify in the address *where* our postcode is. Just like when you mail a postcard (people still do this, right?), you have to specify what country you want it to go to first, and then specify the postcode and address. So we will create a new variable (column) in our dataframe that pastes together the postcode with a suffix to specify our country, in this case ", UK". To do this, we use the `paste()` function. `paste()` just *pastes* together two or more text values, separating them by whatever separator you want. For example, if I wanted to have people's name displayed in different names I could use paste in this way: 


```r
firstname <- "Kurt"
lastname <- "Cobain"

#this way will paste lastname and firstname, and separate them by a comma
paste(lastname, firstname, sep = ",")
```

```
## [1] "Cobain,Kurt"
```

```r
#this way will paste firstname then lastname, and separate them by a space
paste(firstname, lastname, sep = " ")
```

```
## [1] "Kurt Cobain"
```


So in the same vein, we will now create a new column, call it `postcodes2` and use the paste function to put together the postcode with the ", UK" suffix. We want them to be separated by *nothing* so we use `sep = ""`. (note, if you are separating by nothing, you could use the `paste0()` function, you can read about this uising the help function if you want)


```r
city_centre_prems$postcodes2 <- paste(city_centre_prems$POSTCODE, ", UK", sep="")
```


Now we can use this new column to geocode our addresses. I mentioned that the `geocode()` function is part of the `ggmap` package. This means we have to load up this package to use this function: 



```r
library(ggmap)
```

```
## Loading required package: ggplot2
```

```
## Google's Terms of Service: https://cloud.google.com/maps-platform/terms/.
```

```
## Please cite ggmap if you use it! See citation("ggmap") for details.
```


Now we can use `geocode()`. To use this, you have to specify **what** you want to geocode (in this case our newly created column of postcode2) and also the method. I mentioned above you can use Google or the Data Science Toolkit. Google puts a restriction on the number of geocodes you can perform in a day, so I normally use dsk, but both have advantages and limitations, so read up on this if you're interested. But for now, I'm sticking to dsk. 

So let's create a new column, call it `postcode_coords`, use the geocode function to populate it with values: 



```r
city_centre_prems$postcode_coords <- geocode(city_centre_prems$postcodes2, source = 'dsk')
```


Be patient, this will take a while, each postcode has to be referenced against their database and the relevant  coordinates extracted. For each point you will see a note appear in red, and while R is working you will see the red stop sign on the top right corner of the Console window: 


![](img/r_is_working.png)


Also think about how incredibly fast and easy this actually is, if you consider a potential alternative where you have to manualy find some coordinates for each address. That sounds pretty awful, doesn't it? Compared to that, setting the `geocode()` function running, and stepping away to make a cup of tea is really a pretty excellend alternative, no? 




**Note** there might be a chance that you're getting an error telling you that you are over the geocode limit allowed by Google. This appears to be [a bug that has been flagged](https://github.com/dkahle/ggmap/issues/150). There is a workaround, to pass some phoney credentials as part of the `geocode()` function. [This stackoverflow discussion](https://stackoverflow.com/questions/42282492/ggmap-dsk-rate-limit/42604188#42604188) is where I found this. So if you're experiencing this issue, try running the below: 




```r
city_centre_prems$postcode_coords <- geocode(city_centre_prems$postcodes2, client = "123", signature = "123", output = 'latlon', source = 'dsk')
```


Hopefully now you should be getting your points individually gecoded. 

Right so hopefully that is done now, and you can have a look at your data again to see what this new column looks like. Remember you can use the `View()` function to make your data appear in this screen.


```r
View(city_centre_prems)
```


You will see that we have some coordinates. Woohoo! Let's "flatten" them out, so we can use them to map. While some approaches to mapping can deal with the coordinates as one variable, when we use leaflet, we are expected to provide separate latitude and longitude columns. So let's create them here by extracting first the longitude, then the latitude fmor the coords object.  



```r
city_centre_prems$longitude <- city_centre_prems$postcode_coords$lon
city_centre_prems$latitude <- city_centre_prems$postcode_coords$lat
```


And now we have a column called `longitude` for longitude and a column called `latitude` for latitude. Neat!

## Making interactive maps with leaflet 


Thus far we have explored a few approaches to making maps. We made great use of the *tmaps* package for example in the past few weeks. 


As we saw in earlier sessions, [Leaflet](http://leafletjs.com/) is one way to easily make some neat maps. It is the leading open-source JavaScript library for mobile-friendly interactive maps. It is very most popular, used by websites ranging from The New York Times and The Washington Post to GitHub and Flickr, as well as GIS specialists like OpenStreetMap, Mapbox, and CartoDB, some of who's names you'll recognise from the various basemaps we played with in previous labs.


In this section of the lab we will learn how to make really flashy looking maps using leaflet. 

If you haven't already, you will need to have installed the following packages to follow along:


```r
install.packages("leaflet") #for mapping
install.packages("RColorBrewer") #for getting nice colours for your maps
```


Once you have them installed, load them up with the `library()` function:

### Making a map

To make a map, just load the leaflet library:


```r
library(leaflet)
```

You then create a map with this simple bit of code:


```r
m <- leaflet() %>%
  addTiles()  
```


And just print it:

```r
m  
```

<!--html_preserve--><div id="htmlwidget-d7aa6088ef35ce61d35b" style="width:672px;height:480px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-d7aa6088ef35ce61d35b">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addTiles","args":["//{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",null,null,{"minZoom":0,"maxZoom":18,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":1,"detectRetina":false,"attribution":"&copy; <a href=\"http://openstreetmap.org\">OpenStreetMap<\/a> contributors, <a href=\"http://creativecommons.org/licenses/by-sa/2.0/\">CC-BY-SA<\/a>"}]}]},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

This should all be familiar from earlier. 

### Adding some content:

You might of course want to add some content to your map. 

## Adding points manually:

You can add a point manually:


```r
m <- leaflet() %>%
  addTiles()  %>% 
  addMarkers(lng=-2.230899, lat=53.464987, popup="You are here")
m  
```

<!--html_preserve--><div id="htmlwidget-3544915aeee572a6ff8f" style="width:672px;height:480px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-3544915aeee572a6ff8f">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addTiles","args":["//{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",null,null,{"minZoom":0,"maxZoom":18,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":1,"detectRetina":false,"attribution":"&copy; <a href=\"http://openstreetmap.org\">OpenStreetMap<\/a> contributors, <a href=\"http://creativecommons.org/licenses/by-sa/2.0/\">CC-BY-SA<\/a>"}]},{"method":"addMarkers","args":[53.464987,-2.230899,null,null,null,{"interactive":true,"draggable":false,"keyboard":true,"title":"","alt":"","zIndexOffset":0,"opacity":1,"riseOnHover":false,"riseOffset":250},"You are here",null,null,null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null]}],"limits":{"lat":[53.464987,53.464987],"lng":[-2.230899,-2.230899]}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->


Or many points manually, with some popup text as well:



```r
latitudes = c(53.464987, 53.472726, 53.466649) 
longitudes = c(-2.230899, -2.245481, -2.243421) 
popups = c("You are here", "Here is another point", "Here is another point") 
df = data.frame(latitudes, longitudes, popups)      

m <- leaflet(data = df) %>%
  addTiles()  %>%  
  addMarkers(lng=~longitudes, lat=~latitudes, popup=~popups)
m  
```

<!--html_preserve--><div id="htmlwidget-124e3adb8897e9657a81" style="width:672px;height:480px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-124e3adb8897e9657a81">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addTiles","args":["//{s}.tile.openstreetmap.org/{z}/{x}/{y}.png",null,null,{"minZoom":0,"maxZoom":18,"tileSize":256,"subdomains":"abc","errorTileUrl":"","tms":false,"noWrap":false,"zoomOffset":0,"zoomReverse":false,"opacity":1,"zIndex":1,"detectRetina":false,"attribution":"&copy; <a href=\"http://openstreetmap.org\">OpenStreetMap<\/a> contributors, <a href=\"http://creativecommons.org/licenses/by-sa/2.0/\">CC-BY-SA<\/a>"}]},{"method":"addMarkers","args":[[53.464987,53.472726,53.466649],[-2.230899,-2.245481,-2.243421],null,null,null,{"interactive":true,"draggable":false,"keyboard":true,"title":"","alt":"","zIndexOffset":0,"opacity":1,"riseOnHover":false,"riseOffset":250},["You are here","Here is another point","Here is another point"],null,null,null,null,{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null]}],"limits":{"lat":[53.464987,53.472726],"lng":[-2.245481,-2.230899]}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->


### Adding data from elsewhere

Last time around we added crime data to our map.  In this case, we want to be mapping our licensed premises in the city centre, right? So let's do this: 


```r
m <- leaflet(data = city_centre_prems) %>%
  addProviderTiles("Stamen.Toner") %>% 
  addMarkers(lng=~longitude, lat=~latitude, popup=~as.character(PREMISESNAME), label = ~as.character(PREMISESNAME))
m  
```

<!--html_preserve--><div id="htmlwidget-e2893c7dfa7384acb062" style="width:672px;height:480px;" class="leaflet html-widget"></div>
<script type="application/json" data-for="htmlwidget-e2893c7dfa7384acb062">{"x":{"options":{"crs":{"crsClass":"L.CRS.EPSG3857","code":null,"proj4def":null,"projectedBounds":null,"options":{}}},"calls":[{"method":"addProviderTiles","args":["Stamen.Toner",null,null,{"errorTileUrl":"","noWrap":false,"detectRetina":false}]},{"method":"addMarkers","args":[[53.474370458,53.4725909815,53.4765333604,53.4768159354,53.475166705,53.4815963342,53.4788605873,53.4781101784,53.4774978656,53.4766647971,53.4811770037,53.4737792024,53.478003376,53.4802067352,53.4787199075,53.4730549899,53.4786767987,53.4799840269,53.4808977733,53.4762273049,53.475166705,53.4770727238,53.4747210135,54.75844,53.4749489908,53.469873466,54.75844,53.4788736018,54.75844,53.4746796612,53.4766310222,53.4766987556,53.4816757951,53.4763252769,53.4744315889,53.4765149947,53.47598673,53.47598673,53.478076131,53.4766987556,53.4766987556,53.4744315889,53.4758137931,53.4701964907,53.4774898386,53.4786767987,53.4834421839,53.4770097385,53.476101983,53.4737604139,54.75844,53.4793310199,53.4706812346,53.4801194589,53.4800681496,53.4776340599,53.4785918114,53.4804953088,53.471723286,53.4779089571,53.4781753139,53.4747665082,53.4746796612,53.4788605873,53.4744285616,53.4765333604,53.4777234871,53.4808977733,53.4765149947,53.4786419225,53.4770097385,53.4736928908,53.4774898386,53.4766310222,53.4766647971,53.4737078117,53.47598673,53.47486343,53.4707834083,53.4744315889,53.4770552208,53.48233578,53.4823393869,53.4781450998,53.4779387659,53.4807208054,53.47452938,53.4770200947,53.4712202711,53.473955139,53.4761089211,53.48170348,53.4707834083,53.4762282563,53.4737725639,53.4795267369,53.4781101784,53.4766310222,53.4763569048,53.4766647971,53.4746778111,53.4772166531,53.47452938,53.4738436839,53.4707834083,53.473955139,54.75844,53.4795267369,53.47384063,53.4747665082,53.4759638344,54.75844,53.4816757951,53.4763252769,54.75844,53.4774898386,53.4752654693,53.4744315889,53.47414881,53.4766647971,53.4775150882,53.4766310222,54.75844,53.4812933886,53.4701964907,53.4756509208,53.4770097385,53.478003376,53.481028023,53.4760950885,53.4737078117,53.4779387659,53.4766310222,53.48233578,53.4712202711,53.478076131,53.473955139,53.4834663587,53.4732507536,53.4698656617,53.4706812346,53.4776340599,53.4777063958,54.75844,54.75844,53.4763804279,53.4816420913,53.4823710739,53.4781450998,53.4788736018,53.4766617303,53.4745105743,53.4706812346,53.4766310222,53.4788736018,53.478076131,53.4747665082,53.478003376,53.4785918114,53.4780649788,53.4776579509,53.4765333604,53.4752361604,53.4768159354,53.4798277711,53.4737033756,53.4835834385,53.4706812346,53.4786419225,54.75844,53.4786767987,54.75844,53.4745105743,53.4706812346,53.4817992388,53.4770097385,53.4807208054,53.4770727238,53.4735877675,53.4737078117,53.4774898386,53.4835834385,53.4808780249,53.4808270352,53.4768296531,53.4767960374,53.4721674867,53.4774898386,53.4779738718,53.4766987556,53.4769863727,53.4766987556,53.4777687503,53.4801194589,53.4733090192,53.4783573569,53.4725909815,53.481397362,53.4744285616,53.481397362,54.75844,53.4774898386,53.4777063958,53.4739178454,53.4721674867,53.4766310222,53.4816420913,53.4768354779,53.479579422,54.75844,53.4808538564,54.75844,53.4735877675,54.75844,53.4744315889,53.4825707995,53.4820367951,53.4718058879,53.4726462088,53.4802825001,53.4744285616,53.4777687503,53.4698656617,53.4801709874,53.479040875,53.4721674867,53.4747665082,53.4777234871,53.4788736018,53.4770097385,53.479370294,53.4706812346,53.479579422,53.47598673,53.4700272637,53.4822007033,53.4777234871,53.4721674867,53.4738436839,54.75844,53.4799417055,53.4761089211,53.4823041655,54.75844,53.4725909815,53.4770097385,53.4779387659,53.4732507536,53.4772083376,53.4781753139,54.75844,53.4788736018,53.4738749059,53.4725909815,53.4770552208,53.4765195562,53.4808780249,53.4770200947,53.4766617303,54.75844,53.4802067352,53.4766987556,53.4744315889,53.4808780249,53.4769863727,53.478003376,53.4753208311,53.4807572312,53.481397362,54.75844,53.472083408,53.4801470922,53.4815963342,53.473955139,53.4754996194,53.4783291001,54.75844,54.75844,53.4766987556,53.4739125506,53.4804900873,53.475234661,53.4772083376,53.4825707995,53.47414881,53.4829526738,53.47805212,53.4828789893,53.4820367951,54.75844,53.4822007033,53.4788736018,53.4811544285,53.4797468361,53.4816420913,53.4814830546,53.4811207964,53.4811544285,53.4815588051,54.75844,54.75844,53.4756509208,53.47193835,53.4743732466,53.4815963342,53.4779387659,53.4825900994,53.4737078117,53.4772532665,53.472083408,53.4813268412,54.75844,53.4766310222,53.4749971223,54.75844,53.4737725639,53.4766310222,53.4807844333,53.4823041655,53.4807844333,53.4765195562,53.4819494695,53.4811544285,53.4739178454,53.4779738718,53.4744285616,53.4721674867,53.4817992388,53.4786419225,53.4812411931,53.4817992388,53.4792818463,53.4777291574,53.4813268412,53.4706812346,53.473463073,54.75844,53.4743732466,53.4808780249,53.4813268412,53.472083408,53.4779738718,54.75844,53.4744285616,53.4732507536,53.475166705,53.4774473232,53.4726462088,53.478076131,53.4768296531,53.481028023,53.4834739812,53.4776786274],[-2.24316649264,-2.23848544526,-2.23597600199,-2.23853927176,-2.23610403438,-2.2303978591,-2.2319201334,-2.23411615819,-2.2346402054,-2.23769457233,-2.2333945927,-2.23767844783,-2.23812399235,-2.23765408811,-2.23943905156,-2.24020574205,-2.23851956965,-2.23204681948,-2.23367939557,-2.24069078437,-2.23610403438,-2.23597899442,-2.24318356474,-2.69531,-2.2370973254,-2.23530627263,-2.69531,-2.23900289945,-2.69531,-2.24582024251,-2.23659436338,-2.23415359562,-2.23574833707,-2.24114339865,-2.24841050387,-2.23617179357,-2.23509899396,-2.23509899396,-2.23770245979,-2.23415359562,-2.23415359562,-2.24841050387,-2.24068844443,-2.23560938906,-2.23869374204,-2.23851956965,-2.24251002037,-2.23600878278,-2.23131741709,-2.24694501199,-2.69531,-2.23499692274,-2.23595860688,-2.22705938724,-2.23500099586,-2.23849865279,-2.23604770081,-2.23262227446,-2.24083103798,-2.24038384723,-2.23755232094,-2.23390186915,-2.24582024251,-2.2319201334,-2.23661224889,-2.23597600199,-2.23418937542,-2.23367939557,-2.23617179357,-2.23797687267,-2.23600878278,-2.23586983809,-2.23869374204,-2.23659436338,-2.23769457233,-2.24189701481,-2.23509899396,-2.23943241082,-2.2342867701,-2.24841050387,-2.23573779459,-2.23383800547,-2.24107200583,-2.23917963731,-2.2389675077,-2.23223168226,-2.23563338806,-2.2353156705,-2.23612733201,-2.23960811215,-2.23695310525,-2.23077520373,-2.2342867701,-2.23570307401,-2.24099331751,-2.23144153263,-2.23411615819,-2.23659436338,-2.23882300221,-2.23769457233,-2.2423244372,-2.24044021039,-2.23563338806,-2.23689528343,-2.2342867701,-2.23960811215,-2.69531,-2.23144153263,-2.23388171024,-2.23390186915,-2.23758517541,-2.69531,-2.23574833707,-2.24114339865,-2.69531,-2.23869374204,-2.23616485574,-2.24841050387,-2.24164336846,-2.23769457233,-2.23955282152,-2.23659436338,-2.69531,-2.22899466878,-2.23560938906,-2.24121491764,-2.23600878278,-2.23812399235,-2.2314195623,-2.23023244197,-2.24189701481,-2.2389675077,-2.23659436338,-2.23383800547,-2.23612733201,-2.23770245979,-2.23960811215,-2.23028742164,-2.24120130734,-2.23470357454,-2.23595860688,-2.23849865279,-2.2382880903,-2.69531,-2.69531,-2.24054096403,-2.23460278881,-2.23872113992,-2.23917963731,-2.23900289945,-2.23923157088,-2.24056052613,-2.23595860688,-2.23659436338,-2.23900289945,-2.23770245979,-2.23390186915,-2.23812399235,-2.23604770081,-2.2387873855,-2.24003583848,-2.23597600199,-2.24183036494,-2.23853927176,-2.23380914536,-2.23963682949,-2.24377680974,-2.23595860688,-2.23797687267,-2.69531,-2.23851956965,-2.69531,-2.24056052613,-2.23595860688,-2.23238827754,-2.23600878278,-2.23223168226,-2.23597899442,-2.24346337231,-2.24189701481,-2.23869374204,-2.24377680974,-2.2299569296,-2.23307619665,-2.23615847191,-2.22564024439,-2.23898028835,-2.23869374204,-2.23024261213,-2.23415359562,-2.23872105439,-2.23415359562,-2.23857475394,-2.22705938724,-2.23904697176,-2.23182697301,-2.23848544526,-2.23562622987,-2.23661224889,-2.23562622987,-2.69531,-2.23869374204,-2.2382880903,-2.24905541965,-2.23898028835,-2.23659436338,-2.23460278881,-2.242231248,-2.23667104468,-2.69531,-2.23315169498,-2.69531,-2.24346337231,-2.69531,-2.24841050387,-2.23317617938,-2.23502694548,-2.23998775014,-2.24231286417,-2.23571047329,-2.23661224889,-2.23857475394,-2.23470357454,-2.23754839817,-2.22697828072,-2.23898028835,-2.23390186915,-2.23418937542,-2.23900289945,-2.23600878278,-2.2286527797,-2.23595860688,-2.23667104468,-2.23509899396,-2.23480993107,-2.23395782864,-2.23418937542,-2.23898028835,-2.23689528343,-2.69531,-2.23069029385,-2.23695310525,-2.23160735636,-2.69531,-2.23848544526,-2.23600878278,-2.2389675077,-2.24120130734,-2.2355879535,-2.23755232094,-2.69531,-2.23900289945,-2.24372116826,-2.23848544526,-2.23573779459,-2.23840199101,-2.2299569296,-2.2353156705,-2.23923157088,-2.69531,-2.23765408811,-2.23415359562,-2.24841050387,-2.2299569296,-2.23872105439,-2.23812399235,-2.23996237625,-2.23199075748,-2.23562622987,-2.69531,-2.24056186781,-2.23139969861,-2.2303978591,-2.23960811215,-2.2404606411,-2.22305644647,-2.69531,-2.69531,-2.23415359562,-2.24289265446,-2.2306781984,-2.22415527102,-2.2355879535,-2.23317617938,-2.24164336846,-2.24451167332,-2.24071618041,-2.23186669384,-2.23502694548,-2.69531,-2.23395782864,-2.23900289945,-2.23113391313,-2.2383748721,-2.23460278881,-2.23318528556,-2.22992810163,-2.23113391313,-2.23121146353,-2.69531,-2.69531,-2.24121491764,-2.23667373363,-2.24619517107,-2.2303978591,-2.2389675077,-2.23249809561,-2.24189701481,-2.23099216066,-2.24056186781,-2.23491752877,-2.69531,-2.23659436338,-2.24000575466,-2.69531,-2.24099331751,-2.23659436338,-2.23187034395,-2.23160735636,-2.23187034395,-2.23840199101,-2.22907355496,-2.23113391313,-2.24905541965,-2.23024261213,-2.23661224889,-2.23898028835,-2.23238827754,-2.23797687267,-2.2327469177,-2.23238827754,-2.23713654969,-2.23587715238,-2.23491752877,-2.23595860688,-2.23843006389,-2.69531,-2.24619517107,-2.2299569296,-2.23491752877,-2.24056186781,-2.23024261213,-2.69531,-2.23661224889,-2.24120130734,-2.23610403438,-2.23747290963,-2.24231286417,-2.23770245979,-2.23615847191,-2.2314195623,-2.23099580898,-2.219948793],null,null,null,{"interactive":true,"draggable":false,"keyboard":true,"title":"","alt":"","zIndexOffset":0,"opacity":1,"riseOnHover":false,"riseOffset":250},["Rain Bar","Spar","Oscars Bar","Bangkok Bar","Fifth Avenue Nightclub","Cuba Cafe","Malmaison Hotel","Madisons","Churchills","Mechanics Centre Ltd","Mother Macs","Browns","Pan Asia","Marks & Spencer Simply Food","Seven Oaks","Font Bar","The Little Yang Sing","Spar","Wetherspoons","Genting Club Manchester","Downtown","Goose","Peveril of The Peak","Yates Wine Lodge","O'Sheas","The Footage","Monroes","New Emperor","The Bay Horse","Britons Protection","New York New York","View","Co-op","The Paramount","Home","Vanilla","Olive Delicatessen","New Samsi Japanese Restaurant","Tokyo Season","Sackville Theatre","Waterside Theatre","Comedy Store","Giorgio One Ltd","Khimji's Off Licence","Ibis","Hunan Restaurant","Marks and Spencer","Baa Bar","Star And Garter","City Road Inn","The Thompsons Arms","Missoula","Pizza Co","The Jolly Angler","Kro Piccadilly","Grey Horse","Britannia Hotel","Piccadilly Tavern","The Courtyard","Arora International Hotel","Bella Roma","Retro Bar & Basement","Jury's Inn","Waldorf Hotel","Bouzouki By Night","Manto","Taurus Cafe Bar & Restaurant","Gardens Hotel","GAY","Royal Orchid","Rembrandt Hotel","The Factory","Circus Tavern","Napoleons","Cruz 101","Sixxis","Tribeca","Palace Theatre","Sandbar","Baa Bar","Company","Night & Day Cafe Bar","Tesco Metro","Grosvenor Casino","China City","The Roadhouse","Ibis Hotel","Via","Manchester Metropolitan Students Union","The Grand Central","The New Union","Crown and Anchor","The Zoo/The Pub","Eden Bar & Grill","Pumpkin","B Lounge","Essential","Foo Foos","Charlies Night Club","Alter Ego","Temple Of Convenience","New Novotel Hotel","East2East","Lass O'Gowrie","IXIV Nine Four","Cornerhouse Cafe Bar","Mayfield Bar","Manchester Abode Hotel","Manchester Conference Centre","Renold Tavern & Cafe Bar","Manhattan Showbar","Carluccio's","Bella Italia","Turtle Bay","Marks & Spencer Simply Food","Old Monkey","The Han Dynasty","Lola Lo","Ritz","Satans Hollow","Yang Sing Restaurant","Safad Takeaway &  Restaurant","Gorilla","Fresh","Abduls","McDonalds","Habesha","The New Hong Kong Restaurant","Copacabana","The Bulls Head Hotel","Sugar Buddha","Great Wall Restaurant","AXM","Dry Bar","Union Shop","Siam Orchid","Cornerhouse Cinema","Bem Brasil","Blackdog Ballroom","Grand Daddys","Babylon Kebab House","Janam Fast Food","Red Chilli","Dragon City","T J's Fast Foods","Premier Inn Manchester City Centre","Mace","BHS","Pearl City Cantonese Restaurant","Teppanyaki Japanese Restaurant","Buffet City","Sainsbury's","On the Eighth Day Shop","Bloom Street Convenience Store","Hang Won Hong","New Bei Jing","Spar","Wing Fat Supermarket","Bar Roque/ Wave Cafe Bar","China Buffet","Kwok Man Restaurant","Villagio Restaurante & Pizzeria","Bannatyne Health Club","Peking Court","Thistle Manchester","The Salisbury","Harvey Nichols Restaurant","On the Eighth Day Restaurant","Vina Noodle Bar","Happy Seasons","Woo Sang Chinese Supermarket","Burger King","Felicini","Popolinos Pizza","Carnival Organics","Istanbul Express","Kan Zaman Restaurant","Krispy Fish and Chips","The Whim Wham Eatery","Dog Bowl","Beautiful British Butty","Harvey Nichols","Lammars","Subway","McTucky's Fried Chicken","Booze Direct","Nandos","Subway","Subway","Coyotes","Ashoka Indian Cuisine","Dixy Chicken","Fu's Chinese Restaurant Cafe","City Servants","Revolution Bar","Double Tree by Hilton","Dancehouse Theatre","Barburrito","Middle Kingdom","Pizza Express","Mardi Gras (Ncp Car Park)","Go Chicken","Portland St Off Licence","Obsessions","Blue Ginger","New Village Take Away","Crafty Pig","Don Giovannis Restaurant","Mercure Manchester Piccadilly","The Place Apartment Hotel","Buffet Metro","Cafe Ritazza","Cafe Loco","The Place Apartment Hotel Ground Floor","Revolution","The Soup Kitchen","Moholive","Tinmus General Store","R Local Store","Rice Bar","Speedy Peppers","Wasabi Sushi","Trof Cafe Bar","Grill on New York Street","Undercroft Car Park","Zouk Tea Bar & Grill","St Petersburg Restaurant","Kiki","Long Legs","Big Tops","Golftorium","Il Padrino","Quality Save","Sackville Lounge","Dominos Pizza","Noho","Velvet & Velvet Hotel","Dokie Dokie","Projekt 42","Spar","North Star","Baby Platinum","Hula","Pasty Shop","Greggs","Iconic Bar","Red N Hot Chinese Restaurant","Sound Control","Eagle Bar","Coffee Lounge","K2 Karaoke Night Club","I am Pho","Premier Express","Holiday Inn Express","The Molly House","Taj Indian Restaurant","Drip Coffee","Bar Pop","Fab Cafe","Swadesh Manchester","Tesco","Kitsch","Sakura","Lola's Cocktail Lounge","Macdonald Town House Hotel","Yuzu","Pizza Express","Africa Cavern","Zizzi","Bread Box","Thalassa","Premier Inn","Port Street Beer House","Archie's Shake Bar","Tesco","Smithfield Wine","Teacup on Thomas Street","The Faraday Gallery","Genghis Khan","Rainbow Snooker Club","Shack Bar And Grill","Yo! Sushi","Richmond Tea Rooms","The Soup Kitchen","Ritz","Spar","Koh Samui","Bakerie at the Hive","Ghosts in the Machine","Eversheds","Number Five","Pacific Restaurant","The Whiskey Jar","The Alchemist NYS","Leo's Fish Bar","The Manchester Arts Project","Basement Complex","Kosmonaut","Hatters Hostel Ltd","Tops Restaurant","Houldsworth School of Performing Arts","Tai Wu Cantonese Restaurant","Tesco","Element 19","Asmara Bella","Try Thai Restaurant & Cafe","Slice Pizza Co","The Arch","Motel One","Wasabi","Morrisons","Papa Johns Pizza","Krunchy Fried Chicken","Changos Burrito Bar","Spar","Java Bar Espresso","NCP","Cottonopolis Food & Liquor","Rosy Lee Tearoom","Eleska Bar & Restaurant","Efes Restaurant & Tavern","Booze Manchester","El Capo","Rebellion Bar","Pic-a-Deli Cafe","Spar","Cheeky Coffee Co","Guilty By Association","Vina Bar","Allotment","Kosmonaut","Giovanni's Deli","Mango Tree","Travelodge","Archie's Burgers & Shakes","Oxford Kiosk","TGI Fridays","Stone Spar","Drip Coffee","Nando's","Oishi-Q","Waitrose","Turtle Bay Restaurant & Bar","The Joshua Brooks","Thirsty Scholar","The Garratt","Pop Up Cinema","International Anthony Burgess Foundation","New Bei Jing","H2O","Tariff and Dale","Spanish Cupboard Limited","Cloudwater Brewery"],null,null,null,["Rain Bar","Spar","Oscars Bar","Bangkok Bar","Fifth Avenue Nightclub","Cuba Cafe","Malmaison Hotel","Madisons","Churchills","Mechanics Centre Ltd","Mother Macs","Browns","Pan Asia","Marks &amp; Spencer Simply Food","Seven Oaks","Font Bar","The Little Yang Sing","Spar","Wetherspoons","Genting Club Manchester","Downtown","Goose","Peveril of The Peak","Yates Wine Lodge","O'Sheas","The Footage","Monroes","New Emperor","The Bay Horse","Britons Protection","New York New York","View","Co-op","The Paramount","Home","Vanilla","Olive Delicatessen","New Samsi Japanese Restaurant","Tokyo Season","Sackville Theatre","Waterside Theatre","Comedy Store","Giorgio One Ltd","Khimji's Off Licence","Ibis","Hunan Restaurant","Marks and Spencer","Baa Bar","Star And Garter","City Road Inn","The Thompsons Arms","Missoula","Pizza Co","The Jolly Angler","Kro Piccadilly","Grey Horse","Britannia Hotel","Piccadilly Tavern","The Courtyard","Arora International Hotel","Bella Roma","Retro Bar &amp; Basement","Jury's Inn","Waldorf Hotel","Bouzouki By Night","Manto","Taurus Cafe Bar &amp; Restaurant","Gardens Hotel","GAY","Royal Orchid","Rembrandt Hotel","The Factory","Circus Tavern","Napoleons","Cruz 101","Sixxis","Tribeca","Palace Theatre","Sandbar","Baa Bar","Company","Night &amp; Day Cafe Bar","Tesco Metro","Grosvenor Casino","China City","The Roadhouse","Ibis Hotel","Via","Manchester Metropolitan Students Union","The Grand Central","The New Union","Crown and Anchor","The Zoo/The Pub","Eden Bar &amp; Grill","Pumpkin","B Lounge","Essential","Foo Foos","Charlies Night Club","Alter Ego","Temple Of Convenience","New Novotel Hotel","East2East","Lass O'Gowrie","IXIV Nine Four","Cornerhouse Cafe Bar","Mayfield Bar","Manchester Abode Hotel","Manchester Conference Centre","Renold Tavern &amp; Cafe Bar","Manhattan Showbar","Carluccio's","Bella Italia","Turtle Bay","Marks &amp; Spencer Simply Food","Old Monkey","The Han Dynasty","Lola Lo","Ritz","Satans Hollow","Yang Sing Restaurant","Safad Takeaway &amp;  Restaurant","Gorilla","Fresh","Abduls","McDonalds","Habesha","The New Hong Kong Restaurant","Copacabana","The Bulls Head Hotel","Sugar Buddha","Great Wall Restaurant","AXM","Dry Bar","Union Shop","Siam Orchid","Cornerhouse Cinema","Bem Brasil","Blackdog Ballroom","Grand Daddys","Babylon Kebab House","Janam Fast Food","Red Chilli","Dragon City","T J's Fast Foods","Premier Inn Manchester City Centre","Mace","BHS","Pearl City Cantonese Restaurant","Teppanyaki Japanese Restaurant","Buffet City","Sainsbury's","On the Eighth Day Shop","Bloom Street Convenience Store","Hang Won Hong","New Bei Jing","Spar","Wing Fat Supermarket","Bar Roque/ Wave Cafe Bar","China Buffet","Kwok Man Restaurant","Villagio Restaurante &amp; Pizzeria","Bannatyne Health Club","Peking Court","Thistle Manchester","The Salisbury","Harvey Nichols Restaurant","On the Eighth Day Restaurant","Vina Noodle Bar","Happy Seasons","Woo Sang Chinese Supermarket","Burger King","Felicini","Popolinos Pizza","Carnival Organics","Istanbul Express","Kan Zaman Restaurant","Krispy Fish and Chips","The Whim Wham Eatery","Dog Bowl","Beautiful British Butty","Harvey Nichols","Lammars","Subway","McTucky's Fried Chicken","Booze Direct","Nandos","Subway","Subway","Coyotes","Ashoka Indian Cuisine","Dixy Chicken","Fu's Chinese Restaurant Cafe","City Servants","Revolution Bar","Double Tree by Hilton","Dancehouse Theatre","Barburrito","Middle Kingdom","Pizza Express","Mardi Gras (Ncp Car Park)","Go Chicken","Portland St Off Licence","Obsessions","Blue Ginger","New Village Take Away","Crafty Pig","Don Giovannis Restaurant","Mercure Manchester Piccadilly","The Place Apartment Hotel","Buffet Metro","Cafe Ritazza","Cafe Loco","The Place Apartment Hotel Ground Floor","Revolution","The Soup Kitchen","Moholive","Tinmus General Store","R Local Store","Rice Bar","Speedy Peppers","Wasabi Sushi","Trof Cafe Bar","Grill on New York Street","Undercroft Car Park","Zouk Tea Bar &amp; Grill","St Petersburg Restaurant","Kiki","Long Legs","Big Tops","Golftorium","Il Padrino","Quality Save","Sackville Lounge","Dominos Pizza","Noho","Velvet &amp; Velvet Hotel","Dokie Dokie","Projekt 42","Spar","North Star","Baby Platinum","Hula","Pasty Shop","Greggs","Iconic Bar","Red N Hot Chinese Restaurant","Sound Control","Eagle Bar","Coffee Lounge","K2 Karaoke Night Club","I am Pho","Premier Express","Holiday Inn Express","The Molly House","Taj Indian Restaurant","Drip Coffee","Bar Pop","Fab Cafe","Swadesh Manchester","Tesco","Kitsch","Sakura","Lola's Cocktail Lounge","Macdonald Town House Hotel","Yuzu","Pizza Express","Africa Cavern","Zizzi","Bread Box","Thalassa","Premier Inn","Port Street Beer House","Archie's Shake Bar","Tesco","Smithfield Wine","Teacup on Thomas Street","The Faraday Gallery","Genghis Khan","Rainbow Snooker Club","Shack Bar And Grill","Yo! Sushi","Richmond Tea Rooms","The Soup Kitchen","Ritz","Spar","Koh Samui","Bakerie at the Hive","Ghosts in the Machine","Eversheds","Number Five","Pacific Restaurant","The Whiskey Jar","The Alchemist NYS","Leo's Fish Bar","The Manchester Arts Project","Basement Complex","Kosmonaut","Hatters Hostel Ltd","Tops Restaurant","Houldsworth School of Performing Arts","Tai Wu Cantonese Restaurant","Tesco","Element 19","Asmara Bella","Try Thai Restaurant &amp; Cafe","Slice Pizza Co","The Arch","Motel One","Wasabi","Morrisons","Papa Johns Pizza","Krunchy Fried Chicken","Changos Burrito Bar","Spar","Java Bar Espresso","NCP","Cottonopolis Food &amp; Liquor","Rosy Lee Tearoom","Eleska Bar &amp; Restaurant","Efes Restaurant &amp; Tavern","Booze Manchester","El Capo","Rebellion Bar","Pic-a-Deli Cafe","Spar","Cheeky Coffee Co","Guilty By Association","Vina Bar","Allotment","Kosmonaut","Giovanni's Deli","Mango Tree","Travelodge","Archie's Burgers &amp; Shakes","Oxford Kiosk","TGI Fridays","Stone Spar","Drip Coffee","Nando's","Oishi-Q","Waitrose","Turtle Bay Restaurant &amp; Bar","The Joshua Brooks","Thirsty Scholar","The Garratt","Pop Up Cinema","International Anthony Burgess Foundation","New Bei Jing","H2O","Tariff and Dale","Spanish Cupboard Limited","Cloudwater Brewery"],{"interactive":false,"permanent":false,"direction":"auto","opacity":1,"offset":[0,0],"textsize":"10px","textOnly":false,"className":"","sticky":true},null]}],"limits":{"lat":[53.4698656617,54.75844],"lng":[-2.69531,-2.219948793]}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->


Should be looking familiar as well.  Now let's say you wanted to save this map. You can do this by clicking on the export button at the top of the plot viewer, and choose the *Save as Webpage* option saving this as a .html file: 

![](img/save_as_wp.png)

Then you can open this file with any type of web browser (safari, firefox, chrome) and share your map that way. You can send this to your friends not on this course, and make them jealous of your fancy map making skills.


One thing you might have noticed is that we still have some points that are not in Manchester. This should illustrate that the pattern matching approach is really just a work-around. Instead, what we really should be doing to subset our data spatially is to use spatial operations. So now we'll learn how to do some of these in the next section. 


## Spatial operations

Spatial operations are a vital part of geocomputation. Spatial objects can be modified in a multitude of ways based on their location and shape. For a comprehensive overview of spatial operations in R I would recommend the relevant chatper [Chapter 4: Spatial Operations](https://geocompr.robinlovelace.net/spatial-operations.html) from the project of Robin Lovelace and Jakub Nowosad, [Geocomputation with R](https://geocompr.robinlovelace.net/spatial-operations.html). 


> Spatial operations differ from non-spatial operations in some ways. To illustrate the point, imagine you are researching road safety. Spatial joins can be used to find road speed limits related with administrative zones, even when no zone ID is provided. But this raises the question: should the road completely fall inside a zone for its values to be joined? Or is simply crossing or being within a certain distance sufficent? When posing such questions it becomes apparent that spatial operations differ substantially from attribute operations on data frames: the type of spatial relationship between objects must be considered. 

- [(Lovelace & Nowosad, 2018)](https://geocompr.robinlovelace.net/spatial-operations.html)


So you can see we can do exciting spatial operations with our spatial data, which we cannot with the non-spatial stuff. 


For our spatial operations we will be using functions that belong to the `sf` package. So make sure you have this loaded up: 


```r
library(sf)
```

```
## Linking to GEOS 3.6.1, GDAL 2.2.3, PROJ 4.9.3
```

### Coordinate reference systems revisited

One important note before we begin to do this brings us back to some of the learning from the second session on map projections and coordinate reference systems, like we discussed in the lecture today. We spoke about all the ways of flattening out the earth, and ways of making sense what that means for the maps, and also how to be able to point to specific locations within these. The latter refers to the **Coordinate Reference System** or CRS the most common ones we will use are **WGS 84** and **British National Grid**. 


So why are we talking about this? 


***It is important to note that spatial operations that use two spatial objects rely on both objects having the same coordinate reference system***


If we are looking to carry out operations that involve two different spatial objects, they need to have the same CRS!!! Funky weird things happen when this condition is not met, so beware!


So how do we know what CRS our spatial objects are? Well the `sf` package contains a handy function called `st_crs()` which let's us check. All you need to pass into the brackets of this function is the name of the object you want to know the CRS of. 


So let's check what is the CRS of our licenced premises:



```r
st_crs(city_centre_prems)
```

```
## Coordinate Reference System: NA
```



You can see that we get the CRS returned as `NA`. Can you think of why? Have we made this into a spatial object? Or is this merely a dataframe with a latitude and longitude column? The answer is really in the question here. 


So we need to convert this to a sf object, or a spatial object, and make sure that R knows that the latitude and the longitude columns are, in fact, coordinates. 

In the `st_as_sf()` function we specify what we are transforming (the name of our dataframe), the column names that have the coordinates in them (longitude and latitude), the CRS we are using (4326 is the code for WGS 84, which is the CRS that uses latitude and longitude coordinates (remember BNG uses Easting and Northing)), and finally *agr*, the attribute-geometry-relationship, specifies for each non-geometry attribute column how it relates to the geometry, and can have one of following values: "constant", "aggregate", "identity". "constant" is used for attributes that are constant throughout the geometry (e.g. land use), "aggregate" where the attribute is an aggregate value over the geometry (e.g. population density or population count), "identity" when the attributes uniquely identifies the geometry of particular "thing", such as a building ID or a city name. The default value, NA_agr_, implies we don't know.


```r
cc_spatial = st_as_sf(city_centre_prems, coords = c("longitude", "latitude"), 
                 crs = 4326, agr = "constant")
```


Now let's check the CRS of this spatial version of our licensed premises: 



```r
st_crs(cc_spatial)
```

```
## Coordinate Reference System:
##   EPSG: 4326 
##   proj4string: "+proj=longlat +datum=WGS84 +no_defs"
```


We can now see that we have this coordinate system as WGS 84. We need to then make sure that any other spatial object with which we want to perform spatial operations is also in the same CRS. 


### Meet a new format of shapefile: geojson

**GeoJSON** is an open standard format designed for representing simple geographical features, along with their non-spatial attributes. It is based on JSON, the JavaScript Object Notation. It is a format for encoding a variety of geographic data structures.

Geometries are shapes. All simple geometries in GeoJSON consist of a type and a collection of coordinates. The features include points (therefore addresses and locations), line strings (therefore streets, highways and boundaries), polygons (countries, provinces, tracts of land), and multi-part collections of these types. GeoJSON features need not represent entities of the physical world only; mobile routing and navigation apps, for example, might describe their service coverage using GeoJSON.

To tinker with GeoJSON and see how it relates to geographical features, try [geojson.io](geojson.io), a tool that shows code and visual representation in two panes.


Let's read in a geoJSON spatial file, again from the web. This particular geojson represents the wards of Greater Manchester. 



```r
manchester_ward <- st_read("https://raw.githubusercontent.com/RUMgroup/Spatial-data-in-R/master/rumgroup/data/wards.geojson")
```

```
## Reading layer `wards' from data source `https://raw.githubusercontent.com/RUMgroup/Spatial-data-in-R/master/rumgroup/data/wards.geojson' using driver `GeoJSON'
## Simple feature collection with 215 features and 12 fields
## geometry type:  POLYGON
## dimension:      XY
## bbox:           xmin: 351664 ymin: 381168.6 xmax: 406087.5 ymax: 421039.8
## epsg (SRID):    27700
## proj4string:    +proj=tmerc +lat_0=49 +lon_0=-2 +k=0.9996012717 +x_0=400000 +y_0=-100000 +ellps=airy +towgs84=446.448,-125.157,542.06,0.15,0.247,0.842,-20.489 +units=m +no_defs
```


Let's select only the city centre ward, using the `filter()` function from dplyr



```r
city_centre <- manchester_ward %>%
  filter(wd16nm == "City Centre")
```


Let's see how this looks, using the `plot()` function: 



```r
plot(st_geometry(city_centre))
```

<img src="04-week4_files/figure-html/unnamed-chunk-28-1.png" width="672" />




Now we could use this to make sure that our points included in `cc_spatial` are in fact only licensed premises in the city centre. This will be your first spatial operation. Excited? Let's do this!


### Subset points to those within a polygon


So we have our polygon, our spatial file of the city centre ward. We now want to subset our point data, the cc_spatial data, which has points representing licensed premises. 


First things first, we check whether they have the same crs. 



```r
st_crs(city_centre) == st_crs(cc_spatial)
```

```
## [1] FALSE
```



Uh oh! They do not! So what can we do? Well we already know that cc_spatial is in WGS 84, because we made it so a little bit earlier. What about this new city_centre polygon?



```r
st_crs(city_centre) 
```

```
## Coordinate Reference System:
##   EPSG: 27700 
##   proj4string: "+proj=tmerc +lat_0=49 +lon_0=-2 +k=0.9996012717 +x_0=400000 +y_0=-100000 +ellps=airy +towgs84=446.448,-125.157,542.06,0.15,0.247,0.842,-20.489 +units=m +no_defs"
```


Aha, the key is in the `27700`. This code in fact stands for.... British National Grid...! 

So what can we do? We can **transform** our spatial object. Yepp, we can convert between CRS. 


So let's do this now. To do this, we can use the `st_transform()` function. 


```r
cc_WGS84 <- st_transform(city_centre, 4326)
```

Let's check that it worked: 


```r
st_crs(cc_WGS84) 
```

```
## Coordinate Reference System:
##   EPSG: 4326 
##   proj4string: "+proj=longlat +datum=WGS84 +no_defs"
```



Looking good. Triple double check: 


```r
st_crs(cc_WGS84) == st_crs(cc_spatial)
```

```
## [1] TRUE
```


YAY!


Now we can move on to our spatial operation, where we select only those points within the city centre polygon. To do this, we can use the st_intersects() function: 



```r
# intersection
cc_intersects <- st_intersects(cc_WGS84, cc_spatial)
```

```
## although coordinates are longitude/latitude, st_intersects assumes that they are planar
```

```r
# subsetting
cc_intersects <- cc_spatial[unlist(cc_intersects),]
```


have a look at this new `cc_intersects` object in your environment. How many observations does it have? Is this now fewer than the previous `cc_spatial` object? Why do you think this is? 



(hint: you're removing everything that is outside the city centre polygon)


We can plot this too to have a look: 



```r
# plot
plot(st_geometry(cc_WGS84), border="#aaaaaa")
plot(st_geometry(cc_intersects), col = "red", add=T)
```

<img src="04-week4_files/figure-html/unnamed-chunk-35-1.png" width="672" />


COOL, we have successfully performed our first spatial operation, we managed to subset our points data set to include only those points which are inside the polgon for city centre. See how this was much easier, and more reliable than the hacky workaround using pattern matching? Yay!


### Spatial operations two: building buffers


Right, but what we want to do really to go back to our original question. We want to know about crime in and around out areas of interest, in this case our licensed premises. But how can we count this? 


Well first we will need crime data. Let's use the same data set from last week. I'm not going over the detail of how to read this in, if you forgot, go back to the notes from last week. 



```r
crimes <- read.csv("data/2017-11-greater-manchester-street.csv")
```


Now let's make sure again that R is aware that this is a spatial set of points, and that the columns 


```r
crimes_spatial = st_as_sf(crimes, coords = c("Longitude", "Latitude"), 
                 crs = 4326, agr = "constant")
```


Notice that in this case the columns are spelled with upper case "L". You should always familiarise yourself with your data set to make sure you are using the relevant column names. You can see just the column names using the `names()` function like so :



```r
names(crimes)
```

```
##  [1] "Crime.ID"              "Month"                
##  [3] "Reported.by"           "Falls.within"         
##  [5] "Longitude"             "Latitude"             
##  [7] "Location"              "LSOA.code"            
##  [9] "LSOA.name"             "Crime.type"           
## [11] "Last.outcome.category" "Context"
```



Or you can have a look at the first 6 lines of your dataframe with the `head()` function: 



```r
head(crimes)
```

```
##                                                           Crime.ID   Month
## 1                                                                  2017-11
## 2 f892dce3e7a4c45fe4f8f09f24d6a494f2b49783a976afba3d6f51e01f72521f 2017-11
## 3 f48d18a3e88eaeba043807cd95642dccb2a2e007021de06e55956c3ee7bad0a4 2017-11
## 4                                                                  2017-11
## 5 bee6bb133a8ae0b35cf19d2d994ad2bdc3852d1d0d560ce8510359dd02d2e503 2017-11
## 6 7f5838988383f952ec1445fb266cd731d5fa32831c3cc27c234f2a3696dfc2d7 2017-11
##                 Reported.by              Falls.within Longitude Latitude
## 1 Greater Manchester Police Greater Manchester Police -2.462774 53.62210
## 2 Greater Manchester Police Greater Manchester Police -2.462774 53.62210
## 3 Greater Manchester Police Greater Manchester Police -2.462774 53.62210
## 4 Greater Manchester Police Greater Manchester Police -2.464422 53.61250
## 5 Greater Manchester Police Greater Manchester Police -2.444807 53.61151
## 6 Greater Manchester Police Greater Manchester Police -2.444043 53.62939
##                  Location LSOA.code                  LSOA.name
## 1   On or near Scout Road E01012628 Blackburn with Darwen 018D
## 2   On or near Scout Road E01012628 Blackburn with Darwen 018D
## 3   On or near Scout Road E01012628 Blackburn with Darwen 018D
## 4 On or near Parking Area E01004768                Bolton 001A
## 5 On or near Belmont Road E01004768                Bolton 001A
## 6    On or near West Walk E01004803                Bolton 001B
##                     Crime.type
## 1        Anti-social behaviour
## 2    Criminal damage and arson
## 3    Criminal damage and arson
## 4        Anti-social behaviour
## 5 Violence and sexual offences
## 6                  Other theft
##                           Last.outcome.category Context
## 1                                                    NA
## 2 Investigation complete; no suspect identified      NA
## 3                   Unable to prosecute suspect      NA
## 4                                                    NA
## 5                           Under investigation      NA
## 6 Investigation complete; no suspect identified      NA
```


Or you can view, with the `View()` function. 


Now, we have our points that are crimes, right? Well... How do we connect them to our points that are licensed premises? 


One approach is to build a buffer around our licensed premises, and say that we will count all the crimes which fall within a specific radius of this licensed premise. What should this radius be? Well this is where your domain knowledge as criminologist comes in. How far away would you consdier a crime to still be related to this pub? 400 meters? 500 meters? 900 meters? 1 km? What do you think? This is again one of them *it depends* questions. Whatever buffer you choose you should justify, and make sure that you can defend when someone might ask about it, as the further your reach obviously the more crimes you will include, and these might alter your results. 


So, let's say we are interested in all crimes that occur within 400 meters of each licensed premise. We chose 400m here as this is the recommended distance for accessible bus stop guidance, so basically as far as people should walk to get to a bus stop ([TfL, 2008](http://content.tfl.gov.uk/accessibile-bus-stop-design-guidance.pdf)). So in this case, we want to take our points, which represent the licensed premises, and build buffers of 400 meters around them. 


You can do with the `st_buffer()` function: 



```r
prem_buffer <- st_buffer(cc_intersects, 1)
```

```
## Warning in st_buffer.sfc(st_geometry(x), dist, nQuadSegs, endCapStyle =
## endCapStyle, : st_buffer does not correctly buffer longitude/latitude data
```

```
## dist is assumed to be in decimal degrees (arc_degrees).
```


You should get a warning here, like I did above. This message indicates that sf assumes a distance value is given in degrees. This is because we have lat/long data (WSG 48)


One quick fix to avoid this message, is to convert to BNG:



```r
prem_BNG <- st_transform(cc_intersects, 27700)
```


Now we can try again, with meters



```r
prem_buffer <- st_buffer(prem_BNG, 400)
```


Let's see how that looks: 


```r
plot(st_geometry(prem_buffer))
plot(st_geometry(prem_BNG), add = T)
```

<img src="04-week4_files/figure-html/unnamed-chunk-43-1.png" width="672" />


That should look nice and squiggly. But also it looks like there is *quite* a lot of overlap here. Should we maybe consider smaller buffers? Let's look at 100 meter buffers: 




```r
prem_buffer_100 <- st_buffer(prem_BNG, 100)
plot(st_geometry(prem_buffer_100))
plot(st_geometry(prem_BNG), add = T)
```

<img src="04-week4_files/figure-html/unnamed-chunk-44-1.png" width="672" />


Still quite a bit of overlap, but this is possibly down to all the licensed premises being very densely close together in the city centre. 

Well now let's have a look at our crimes. I think it might make sense (again using domain knowledge) to restrict the analysis to violent crime. So let's do this: 




```r
violent_spatial <- crimes_spatial %>%
  filter(Crime.type=="Violence and sexual offences")
```


Now, remember the CRS is WGS 48 here, so we will need to convert our buffer layer back to this: 



```r
buffer_WGS84 <- st_transform(prem_buffer_100, 4326)
```

Now let's just have a look:


```r
plot(st_geometry(buffer_WGS84))
plot(st_geometry(violent_spatial), add = T)
```

<img src="04-week4_files/figure-html/unnamed-chunk-47-1.png" width="672" />



OKAY, so some crimes fall inside some buffers, others not so much. Well, let's get to our last spatial operation of the day, the famous points in polygon, to get to answering which licensed premises have the most violent crimes near them. 


### Points in Polygon

When you have a polygon layer and a point layer - and want to know how many or which of the points fall within the bounds of each polygon, you can use this method of analysis. In computational geometry, the point-in-polygon (PIP) problem asks whether a given point in the plane lies inside, outside, or on the boundary of a polygon. As you can see, this is quite relevant to our problem, wanting to count how many crimes (points) fall within 100 meters of our licensed premises (our buffer polygons). 



```r
crimes_per_prem <- violent_spatial %>% 
  st_join(buffer_WGS84, ., left = FALSE) %>% 
  count(PREMISESNAME)
```

```
## although coordinates are longitude/latitude, st_intersects assumes that they are planar
```


You now have a new dataframe, `crimes_per_prem` which has a column for the name of the premises, a column for the number of violend crimes that fall within the buffer, and a column for the geometry. 


Take a moment to look at this table. Use the View() function. Which premises have the most violent crimes? Are you surprised? 


Now as a final step, let's plot this, going back to leaflet. We can shade by the number of crimes within the buffer, and include a little popup label with the name of the establishment: 



```r
pal <- colorBin("RdPu", domain = crimes_per_prem$n, bins = 5, pretty = TRUE)
leaflet(crimes_per_prem) %>% 
  addTiles() %>% 
  addPolygons(fillColor = ~pal(n), fillOpacity = 0.8,
              weight = 1, opacity = 1, color = "black",
              label = ~as.character(PREMISESNAME)) %>% 
  addLegend(pal = pal, values = ~n, opacity = 0.7, 
            title = 'Violend crimes', position = "bottomleft") 
```

<!--html_preserve--><div id="htmlwidget-3087965e6ce53659eed4" style="width:672px;height:480px;" class="leaflet html-widget"></div>


It's not the neatest of maps, with all these overlaps, but we can talk about prettifying maps another day. You've done enough today. 

## Recap


Today we learned to: 

- use **geocoding** methods to translate postcodes into geographic coordinates 
- make interactive point map with leaflet
- about a new format of spatial shape file called **geojson**
- subset points that are within a certain area using a **spatial operation**
- create new polygons by generating **buffers** around points
- count the number of points that fall within a polygon (known as **points in polygon**)



## Homework

In the homework you will now apply your learning to a new problem. To do so, please complete the following tasks: 

### **Geocode the addresses:** 
Use the geocode function to geocode, but this time pass the actual address values, rather than just the post code. 

You can geocode from addresses. For example, if we want to geocode the location of [Shoryu ramen](https://www.shoryuramen.com/stores/76-manchester), we could use the address to do so:


```r
geocode("1 Piccadilly, Manchester M1 1RG", source = "dsk")
```

```
## Source : http://www.datasciencetoolkit.org/maps/api/geocode/json?address=1+Piccadilly,+Manchester+M1+1RG
```

```
## # A tibble: 1 x 2
##     lon   lat
##   <dbl> <dbl>
## 1 -2.24  53.5
```

You can see that this is slightly different than the postcode centroid: 


```r
geocode("M1 1RG", source = 'dsk')
```

```
## Source : http://www.datasciencetoolkit.org/maps/api/geocode/json?address=M1+1RG
```

```
## # A tibble: 1 x 2
##     lon   lat
##   <dbl> <dbl>
## 1 -2.24  53.5
```


- **1) Discuss why you think the geocode for the address is different than the geocode for the postcode**
- **2) Geocode the address for all the city centre licensed premises**
- **3) Did this work OK? Do you have any geocoded to weird locations? Why do you think this might be?**
- **4) Fix (or at describe how you would fix) any issues with geocoding that you identified above**


Make sure you think a little bit about this. Not only do you have to consider what variable to pass to the `geocode()` function, but also whether that's enough information to get the relevant coordinates. 






