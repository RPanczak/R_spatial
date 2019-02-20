---
title: "R and spatial data: resources"
author: "Radoslaw Panczak"
date: 2019-02-25
output: 
  html_document: 
    highlight: pygments
    keep_md: yes
    number_sections: yes
    theme: united
    toc: yes
    toc_depth: 4
    toc_float: yes
---

# R

## General 

Excellent (and freely available!) *R for Data Science* book comes very handy as a general R resource. Although it doesn't cover spatial data it will guide you through best practices of manipulating and graphing your data. Some chapters and parts  related to our work that can be useful are:

  - setting up and working with RStudio projects  https://r4ds.had.co.nz/workflow-projects.html
  - working with pipes https://r4ds.had.co.nz/transform.html#combining-multiple-operations-with-the-pipe
  - joining data https://r4ds.had.co.nz/relational-data.html


## Spatial 

Learn more about simple features and `sf` library https://r-spatial.github.io/sf/

Handy Simple Features (sf) cheatsheet https://github.com/rstudio/cheatsheets/raw/master/sf.pdf 

Learn more about `tmap` library https://github.com/mtennekes/tmap

Data Carpentry’s Geospatial Workshop http://datacarpentry.org/geospatial-workshop/

Three part tutorial on prettyfying your`sf` & `ggplot2` static maps https://www.r-spatial.org/r/2018/10/25/ggplot2-sf.html https://www.r-spatial.org/r/2018/10/25/ggplot2-sf-2.html 
https://www.r-spatial.org/r/2018/10/25/ggplot2-sf-3.html


# Projections

Short video introducing projections https://www.youtube.com/watch?v=kIID5FDi2JQ 

Myriahedral Projections: Jarke J. van Wijk. Unfolding the Earth: Myriahedral Projections. *The Cartographic Journa*l, Vol. 45, No. 1, pp.32-42, February 2008. https://www.win.tue.nl/~vanwijk/myriahedral/ 


# Cartography

A short, friendly guide to basic principles of map design by axismaps https://www.axismaps.com/guide/


# Geospatial analysis

"Applied Spatial Data Analysis with R" book is a goto resouce explaining many concepts of spatial analysis and providing R code https://asdar-book.org/.

Theory behind many ideas and concepts can be found in "Geospatial Analysis - A comprehensive guide - 2018" book available at https://www.spatialanalysisonline.com/ (browse content at https://www.spatialanalysisonline.com/HTML/index.html). Chapter on global autocorrelation for instance is here https://www.spatialanalysisonline.com/HTML/index.html?global_spatial_autocorrelation.htm and local - here https://www.spatialanalysisonline.com/HTML/index.html?local_indicators_of_spatial_as.htm

Original work on Moran's I is: Moran, P.A.P. (1950). "Notes on Continuous Stochastic Phenomena". *Biometrika*. **37**(1): 17–23. https://doi.org/10.2307/2332142

Original work on LISA is: Anselin, L. (1995)  "Local indicators of spatial association - LISA". *Geographical Analysis* **27**:93-115. https://doi.org/10.1111/j.1538-4632.1995.tb00338.x

# Inspiration

Some pretty R based spatial visualizations http://spatial.ly/r/

Reddit galleryt of unusual maps https://www.reddit.com/r/MapPorn/
