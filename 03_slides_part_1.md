---
title: "R and spatial data: introduction 1"
author: "Radoslaw Panczak"
date: 2019-02-25
output: 
  ioslides_presentation: 
    highlight: pygments
    keep_md: yes
    widescreen: yes
---



## Why analyse data?

![](./images/donut.jpg)

## Setting things in context

- 🤕 patients admitted to hospital 

- 🚕 traffic in the city 

- 🤳 social media posts 

- ⛈ weather patterns

- 🐒 animal movement 

- 🍒 agricultural output 

- 💻 pixels on images 


## Setting things in context

- 🤕 patients admitted to hospital 🌐

- 🚕 traffic in the city 🌐

- 🤳 social media posts 🌐

- ⛈ weather patterns 🌐

- 🐒 animal movement 🌐

- 🍒 agricultural output 🌐

- 💻 pixels on images 🌐


## Why R spatial?

![](./images/R.jpg)


## Is my data spatial?




```r
listings %>% select(-host_id) %>% slice(1:5)
```

```
## # A tibble: 5 x 8
##      id name     host_name neighbourhood latitude longitude room_type price
##   <dbl> <chr>    <chr>     <chr>            <dbl>     <dbl> <chr>     <dbl>
## 1  9835 Beautif~ Manju     Manningham       -37.8      145. Private ~    60
## 2 10803 Room in~ Lindsay   Moreland         -37.8      145. Private ~    35
## 3 12936 St Kild~ Frank & ~ Port Phillip     -37.9      145. Entire h~   159
## 4 15246 Large p~ Eleni     Darebin          -37.8      145. Private ~    50
## 5 16760 Melbour~ Colin     Port Phillip     -37.9      145. Private ~    69
```

## Is my data spatial?


```r
ggplot(listings, aes(x = longitude, y = latitude)) + geom_point()
```

![](03_slides_part_1_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

## Is my data spatial?




```r
SA2_2016_MELB %>% select(-starts_with("SA4")) %>% slice(1:5)
```

```
## Simple feature collection with 5 features and 2 fields
## geometry type:  MULTIPOLYGON
## dimension:      XY
## bbox:           xmin: 144.3336 ymin: -38.50299 xmax: 145.8784 ymax: -37.1751
## epsg (SRID):    NA
## proj4string:    +proj=longlat +ellps=GRS80 +no_defs
##   SA2_MAIN16        SA2_NAME16                       geometry
## 1  206011105         Brunswick MULTIPOLYGON (((144.9497 -3...
## 2  206011106    Brunswick East MULTIPOLYGON (((144.9734 -3...
## 3  206011107    Brunswick West MULTIPOLYGON (((144.9341 -3...
## 4  206011108            Coburg MULTIPOLYGON (((144.9485 -3...
## 5  206011109 Pascoe Vale South MULTIPOLYGON (((144.9326 -3...
```

## Is my data spatial?


```r
ggplot(SA2_2016_MELB) + geom_sf()
```

![](03_slides_part_1_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

## What is special about spatial (1)?


```r
table(st_geometry_type(SA2_2016_MELB))
```

```
## 
##           GEOMETRY              POINT         LINESTRING 
##                  0                  0                  0 
##            POLYGON         MULTIPOINT    MULTILINESTRING 
##                  0                  0                  0 
##       MULTIPOLYGON GEOMETRYCOLLECTION     CIRCULARSTRING 
##                309                  0                  0 
##      COMPOUNDCURVE       CURVEPOLYGON         MULTICURVE 
##                  0                  0                  0 
##       MULTISURFACE              CURVE            SURFACE 
##                  0                  0                  0 
##  POLYHEDRALSURFACE                TIN           TRIANGLE 
##                  0                  0                  0
```

## Vector data

![](./images/vector.png)

Image Source: NEON, via Data Carpentry


## Raster data

![](./images/raster.png)

Image Source: NEON, via Data Carpentry


## What is special about spatial (2)?


```r
st_crs(SA2_2016_MELB)
```

```
## Coordinate Reference System:
##   No EPSG code
##   proj4string: "+proj=longlat +ellps=GRS80 +no_defs"
```

> - ... allowing "every location on Earth to be specified by a set of numbers, letters or symbols" 

> - ... but "To specify a location on a plane requires a map projection." [Wikipedia](https://en.wikipedia.org/wiki/Geographic_coordinate_system)


## What we are going to cover 

- Read, create and manipulate spatial data
- Work with vector point and polygon data
- Create an interactive map
- Create a static map
- Learn basics of cartography 
