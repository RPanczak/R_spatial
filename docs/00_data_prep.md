---
title: "R and spatial data: data preparation"
author: "Radoslaw Panczak"
date: "26 February, 2019"
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

<!-- ------------------------------------------------------------ --> 
<!-- ------------------------------------------------------------ --> 



# What is it?

Short script to prepare the dataset used in the course. Not needed by students but left for the curious ones. Simple data preps steps trying to remove some problems, making data smaller, etc.

# Inside Airbnb

Inside Airbnb data for Melbourne obtained from http://insideairbnb.com/about.html. Verision of the data (`Date compiled`) is `07 December, 2018`.

Selected variables were kept and file was resaved as `csv` without any further modifications.


```r
listings <- read_csv("data/raw/inside_airbnb/listings.csv") %>% 
  select(-neighbourhood_group, - minimum_nights, -number_of_reviews, -last_review, -reviews_per_month, -calculated_host_listings_count, -availability_365)
```

```
## Parsed with column specification:
## cols(
##   id = col_double(),
##   name = col_character(),
##   host_id = col_double(),
##   host_name = col_character(),
##   neighbourhood_group = col_logical(),
##   neighbourhood = col_character(),
##   latitude = col_double(),
##   longitude = col_double(),
##   room_type = col_character(),
##   price = col_double(),
##   minimum_nights = col_double(),
##   number_of_reviews = col_double(),
##   last_review = col_date(format = ""),
##   reviews_per_month = col_double(),
##   calculated_host_listings_count = col_double(),
##   availability_365 = col_double()
## )
```

```r
write_csv(listings, "data/listings.csv")
```

# SEIFA

## Tabular data

Socio-Economic Indexes for Areas (SEIFA) obtained from [ABS](http://www.abs.gov.au/AUSSTATS/abs@.nsf/Lookup/2033.0.55.001Main+Features12016?OpenDocument)

Data preparation included cleaning names, removing empty rows and columns and creating more sensible variable names.


```r
SEIFA <- read_excel("data/raw/abs/2033055001 - sa2 indexes.xls", 
                            sheet = "Table 1", skip = 5, na = "-") %>% 
  clean_names() %>%
  remove_empty(c("rows", "cols")) %>% 
  dplyr::rename(SA2_MAIN16 = `x1`, SA2_NAME16 = `x2`, 
                IRSD_s = `score_3`, IRSD_d = `decile_4`,
                IRSAD_s = `score_5`, IRSAD_d = `decile_6`, 
                IER_s = `score_7`, IER_d = `decile_8`, 
                IEO_s = `score_9`, IEO_d = `decile_10`, 
                URP = `x11`) %>% 
  dplyr::mutate(IRSD_d = as.integer(IRSD_d),
                IRSAD_d = as.integer(IRSAD_d),
                IER_d = as.integer(IER_d),
                IEO_d = as.integer(IEO_d)) %>% 
  select(-SA2_NAME16)
```

```
## Warning in read_fun(path = enc2native(normalizePath(path)), sheet_i =
## sheet, : Expecting numeric in A2199 / R2199C1: got 'Â© Commonwealth of
## Australia 2018'
```

```
## New names:
## * `` -> `..1`
## * `` -> `..2`
## * Score -> Score..3
## * Decile -> Decile..4
## * Score -> Score..5
## * ... and 6 more
```

```r
saveRDS(SEIFA, "data/SEIFA.Rds")
```


## SA2s

SA2 files from [Australian Statistical Geography Standard (ASGS)](http://www.abs.gov.au/ausstats/abs@.nsf/mf/1270.0.55.001)

Data for GCC_NAME16 Greater Melbourne selected. SA3, GCC and STE variables removed. `SA2_MAIN16` was convertedfrom factor to numeric to ease joins.


```r
SA2 <- st_read("./data/raw/abs/1270055001_sa2_2016_aust_shape/SA2_2016_AUST.shp") 
```

```
## Reading layer `SA2_2016_AUST' from data source `C:\Users\uqrpancz\Google Drive\edu\ReduSpatial\data\raw\abs\1270055001_sa2_2016_aust_shape\SA2_2016_AUST.shp' using driver `ESRI Shapefile'
## Simple feature collection with 2310 features and 12 fields (with 18 geometries empty)
## geometry type:  MULTIPOLYGON
## dimension:      XY
## bbox:           xmin: 96.81694 ymin: -43.74051 xmax: 167.998 ymax: -9.142176
## epsg (SRID):    4283
## proj4string:    +proj=longlat +ellps=GRS80 +towgs84=0,0,0,0,0,0,0 +no_defs
```

```r
SA2_2016_MELB <- SA2 %>% 
  filter(GCC_NAME16 == "Greater Melbourne") %>% 
  select(-starts_with("SA3"), -starts_with("GCC"), -starts_with("STE"), -AREASQKM16, -SA2_5DIG16)

SA2_2016_MELB$SA2_MAIN16 <- as.numeric(as.character(SA2_2016_MELB$SA2_MAIN16))

st_write(SA2_2016_MELB, "./data/SA2_2016_MELB.shp")
```

```
## Writing layer `SA2_2016_MELB' to data source `./data/SA2_2016_MELB.shp' using driver `ESRI Shapefile'
## features:       309
## fields:         4
## geometry type:  Multi Polygon
```

