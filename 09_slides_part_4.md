---
title: "R and spatial data: intermediate 2"
author: "Radoslaw Panczak"
date: 2019-02-26
output: 
  ioslides_presentation: 
    highlight: pygments
    keep_md: yes
    widescreen: yes
---



## Mapping vs. analysis


## Spatial dependence - theory

![](./images/spdep.PNG)

Source: Lloyd, 2011


## Spatial dependence - practice

![](09_slides_part_4_files/figure-html/unnamed-chunk-2-1.png)<!-- -->

## Neighbours - theory

![](./images/queen.PNG)

Source: Lloyd, 2011


## Neighbours - practice

![](09_slides_part_4_files/figure-html/unnamed-chunk-3-1.png)<!-- -->


## Global autocorrelation 

![](./images/global.gif)


Image: [arcgis](https://pro.arcgis.com/en/pro-app/tool-reference/spatial-statistics/cluster-and-outlier-analysis-anselin-local-moran-s.htm)

Moran, P (1950) Notes on Continuous Stochastic Phenomena. *Biometrika*, 37(1-2): 17-23.


## Local autocorrelation 

![](./images/lisa.png)

Image: [arcgis](https://pro.arcgis.com/en/pro-app/tool-reference/spatial-statistics/cluster-and-outlier-analysis-anselin-local-moran-s.htm)

Anselin, L (1995) "Local Indicators of Spatial Association—LISA," Geographical Analysis, 27(2): 93–115.


## What we are going to cover 

- Learn about alternative way of representing spatial data in R
- Explore spatial correlation
- Search for clusters and hotspots 
- Determine significance of results
