# Thematic Interactive Map
Joshua Kunst  



Last week all the _rstatsosphere_ see the beatiful 
[Timo Grossenbacher](timogrossenbacher.ch) work.

The last month, yep, the past year I've working on create map
easily with highcharter. When I saw this chart I took as 
challege so:

http://giphy.com/gifs/fangirl-challenge-13Vjd8PZuKxu2Q




```r
# packages ----------------------------------------------------------------
library(dplyr)
library(maptools)
library(highcharter)
library(geojsonio)
library(rmapshaper)
library(readr)
library(viridis)
library(purrr)
```

## Data

The data used fhor this chart is the same using by Timo. But the workflow
was slightly modified:

1. Read the shapefile with `maptools::readShapeSpatial`.
2. Simplify the shapefile (optional step) using `rmapshaper::ms_simplify`.
3. Then transform the map data to geojson using `geojsonio::geojson_list`.




```
## List of 2
##  $ type    : chr "FeatureCollection"
##  $ features:List of 2324
##   .. [list output truncated]
##  - attr(*, "class")= chr "geo_list"
##  - attr(*, "from")= chr "SpatialPolygonsDataFrame"
```

```
## List of 4
##  $ type      : chr "Feature"
##  $ id        : int 6
##  $ properties:List of 3
##   ..$ Secondary_  : chr "Knonau"
##   ..$ BFS_ID      : int 7
##   ..$ rmapshaperid: int 6
##  $ geometry  :List of 2
##   ..$ type       : chr "MultiPolygon"
##   ..$ coordinates:List of 1
##   .. ..$ :List of 1
##   .. .. ..$ :List of 7
##   .. .. .. ..$ : num [1:2] 676327 230767
##   .. .. .. ..$ : num [1:2] 675655 233007
##   .. .. .. ..$ : num [1:2] 676458 233252
##   .. .. .. ..$ : num [1:2] 679285 230257
##   .. .. .. ..$ : num [1:2] 679590 229402
##   .. .. .. ..$ : num [1:2] 678887 229152
##   .. .. .. ..$ : num [1:2] 676327 230767
```

## Map

Create the raw map is straightforward. The _main_ challege was replicate the 
relief feature of the orignal map. This took _some days_ to figure
how add the backgound image. I almost loose the hope but you know, new year,
so I tried a little more and it was possible :):

1. First I searched a way to transform the _tif_ image to geojson. I wrote
a mail to get the answer to  this but the reply said I was wrong XD. NEXT!
2. I tried with use `divBackgroundImage` but with this the image use all the
container... so... NEXT.
3. Finally surfing in the web I met `plotBackgroundImage` argument in highcharts
which is uesd to put and image only in plot container (inside de axis) and 
it works nicely. It was necessary hack the image using the `preserveAspectRatio`
(html world) to center the image but nothing magical. 




```r
urlimage <- "https://raw.githubusercontent.com/jbkunst/r-posts/master/061-beautiful-thematic-maps-with-ggplot2-highcharter-version/02-relief-georef-clipped-resampled.jpg"

highchart(type = "map") %>% 
  # data part
  hc_add_series(mapData = map, data = data, type = "map",
                joinBy = c("BFS_ID", "bfs_id"), value = "value",
                borderWidth = 0) %>% 
  hc_colorAxis(dataClasses = color_classes(brks, colors)) %>% 
  # functionality
  hc_tooltip(headerFormat = "",
             pointFormat = "{point.name}: {point.value}",
             valueDecimals = 2) %>% 
  hc_legend(align = "right", verticalAlign = "bottom", layout = "vertical",
            floating = TRUE) %>%
  hc_mapNavigation(enabled = FALSE) %>% # if TRUE to zoom the relief image dont zoom.
  # info
  hc_title(text = "Switzerland's regional demographics") %>% 
  hc_subtitle(text = "Average age in Swiss municipalities, 2015") %>% 
  hc_credits(enabled = TRUE,
             text = "Map CC-BY-SA; Author: Joshua Kunst (@jbkunst) based mostly on Timo Grossenbacher (@grssnbchr) work, Geometries: ThemaKart, BFS; Data: BFS, 2016; Relief: swisstopo, 2016") %>% 
  # style
  hc_chart(plotBackgroundImage = urlimage,
           backgroundColor = "transparent",
           events = list(
             load = JS("function(){ $(\"image\")[0].setAttribute('preserveAspectRatio', 'xMidYMid') }")
           ))
```

<!--html_preserve--><div id="htmlwidget-2a357b6da1dd5bd6d124" style="width:100%;height:500px;" class="highchart html-widget"></div>

Same as the original/ggplot2 version! 

The are some details like:

1. The image/relief need to be accesible in web. I don't know how
to add dependencies as images. I tried econding the image but didn't work.
2. I could not do the legend same as the original. So I used `dataClasses`
instead of `stops` in `hc_colorAxis`. 

---
title: "readme.R"
author: "joshua.kunst"
date: "Tue Jan 03 16:00:02 2017"
---