---
title: "R and spatial data: data preparation"
author: "Radoslaw Panczak"
date: "12 February, 2019"
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

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206011105 of field
## SA2_MAIN16 of feature 0 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206011106 of field
## SA2_MAIN16 of feature 1 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206011107 of field
## SA2_MAIN16 of feature 2 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206011108 of field
## SA2_MAIN16 of feature 3 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206011109 of field
## SA2_MAIN16 of feature 4 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206021110 of field
## SA2_MAIN16 of feature 5 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206021111 of field
## SA2_MAIN16 of feature 6 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206021112 of field
## SA2_MAIN16 of feature 7 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206031113 of field
## SA2_MAIN16 of feature 8 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206031114 of field
## SA2_MAIN16 of feature 9 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206031115 of field
## SA2_MAIN16 of feature 10 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206031116 of field
## SA2_MAIN16 of feature 11 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206041117 of field
## SA2_MAIN16 of feature 12 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206041118 of field
## SA2_MAIN16 of feature 13 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206041119 of field
## SA2_MAIN16 of feature 14 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206041120 of field
## SA2_MAIN16 of feature 15 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206041121 of field
## SA2_MAIN16 of feature 16 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206041122 of field
## SA2_MAIN16 of feature 17 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206041123 of field
## SA2_MAIN16 of feature 18 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206041124 of field
## SA2_MAIN16 of feature 19 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206041125 of field
## SA2_MAIN16 of feature 20 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206041126 of field
## SA2_MAIN16 of feature 21 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206041127 of field
## SA2_MAIN16 of feature 22 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206051128 of field
## SA2_MAIN16 of feature 23 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206051129 of field
## SA2_MAIN16 of feature 24 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206051130 of field
## SA2_MAIN16 of feature 25 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206051131 of field
## SA2_MAIN16 of feature 26 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206051132 of field
## SA2_MAIN16 of feature 27 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206051133 of field
## SA2_MAIN16 of feature 28 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206051134 of field
## SA2_MAIN16 of feature 29 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206061135 of field
## SA2_MAIN16 of feature 30 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206061136 of field
## SA2_MAIN16 of feature 31 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206061137 of field
## SA2_MAIN16 of feature 32 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206061138 of field
## SA2_MAIN16 of feature 33 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206071139 of field
## SA2_MAIN16 of feature 34 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206071140 of field
## SA2_MAIN16 of feature 35 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206071141 of field
## SA2_MAIN16 of feature 36 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206071142 of field
## SA2_MAIN16 of feature 37 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206071143 of field
## SA2_MAIN16 of feature 38 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206071144 of field
## SA2_MAIN16 of feature 39 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 206071145 of field
## SA2_MAIN16 of feature 40 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207011146 of field
## SA2_MAIN16 of feature 41 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207011147 of field
## SA2_MAIN16 of feature 42 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207011148 of field
## SA2_MAIN16 of feature 43 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207011149 of field
## SA2_MAIN16 of feature 44 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207011150 of field
## SA2_MAIN16 of feature 45 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207011151 of field
## SA2_MAIN16 of feature 46 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207011152 of field
## SA2_MAIN16 of feature 47 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207011153 of field
## SA2_MAIN16 of feature 48 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207011154 of field
## SA2_MAIN16 of feature 49 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207011155 of field
## SA2_MAIN16 of feature 50 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207021156 of field
## SA2_MAIN16 of feature 51 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207021157 of field
## SA2_MAIN16 of feature 52 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207021159 of field
## SA2_MAIN16 of feature 53 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207021160 of field
## SA2_MAIN16 of feature 54 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207021424 of field
## SA2_MAIN16 of feature 55 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207021425 of field
## SA2_MAIN16 of feature 56 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207031161 of field
## SA2_MAIN16 of feature 57 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207031162 of field
## SA2_MAIN16 of feature 58 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207031163 of field
## SA2_MAIN16 of feature 59 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207031164 of field
## SA2_MAIN16 of feature 60 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207031165 of field
## SA2_MAIN16 of feature 61 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207031166 of field
## SA2_MAIN16 of feature 62 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 207031167 of field
## SA2_MAIN16 of feature 63 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208011168 of field
## SA2_MAIN16 of feature 64 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208011169 of field
## SA2_MAIN16 of feature 65 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208011170 of field
## SA2_MAIN16 of feature 66 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208011171 of field
## SA2_MAIN16 of feature 67 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208011172 of field
## SA2_MAIN16 of feature 68 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208011173 of field
## SA2_MAIN16 of feature 69 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208021174 of field
## SA2_MAIN16 of feature 70 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208021176 of field
## SA2_MAIN16 of feature 71 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208021177 of field
## SA2_MAIN16 of feature 72 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208021178 of field
## SA2_MAIN16 of feature 73 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208021179 of field
## SA2_MAIN16 of feature 74 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208021180 of field
## SA2_MAIN16 of feature 75 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208021181 of field
## SA2_MAIN16 of feature 76 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208021182 of field
## SA2_MAIN16 of feature 77 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208021426 of field
## SA2_MAIN16 of feature 78 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208021427 of field
## SA2_MAIN16 of feature 79 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208031183 of field
## SA2_MAIN16 of feature 80 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208031184 of field
## SA2_MAIN16 of feature 81 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208031185 of field
## SA2_MAIN16 of feature 82 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208031186 of field
## SA2_MAIN16 of feature 83 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208031187 of field
## SA2_MAIN16 of feature 84 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208031188 of field
## SA2_MAIN16 of feature 85 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208031189 of field
## SA2_MAIN16 of feature 86 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208031190 of field
## SA2_MAIN16 of feature 87 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208031191 of field
## SA2_MAIN16 of feature 88 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208031192 of field
## SA2_MAIN16 of feature 89 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208031193 of field
## SA2_MAIN16 of feature 90 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208041194 of field
## SA2_MAIN16 of feature 91 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 208041195 of field
## SA2_MAIN16 of feature 92 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209011196 of field
## SA2_MAIN16 of feature 93 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209011197 of field
## SA2_MAIN16 of feature 94 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209011198 of field
## SA2_MAIN16 of feature 95 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209011199 of field
## SA2_MAIN16 of feature 96 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209011200 of field
## SA2_MAIN16 of feature 97 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209011201 of field
## SA2_MAIN16 of feature 98 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209011202 of field
## SA2_MAIN16 of feature 99 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209011203 of field
## SA2_MAIN16 of feature 100 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209011204 of field
## SA2_MAIN16 of feature 101 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209021205 of field
## SA2_MAIN16 of feature 102 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209021207 of field
## SA2_MAIN16 of feature 103 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209021208 of field
## SA2_MAIN16 of feature 104 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209021428 of field
## SA2_MAIN16 of feature 105 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209021429 of field
## SA2_MAIN16 of feature 106 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209031209 of field
## SA2_MAIN16 of feature 107 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209031210 of field
## SA2_MAIN16 of feature 108 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209031211 of field
## SA2_MAIN16 of feature 109 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209031212 of field
## SA2_MAIN16 of feature 110 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209031213 of field
## SA2_MAIN16 of feature 111 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209031214 of field
## SA2_MAIN16 of feature 112 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209031215 of field
## SA2_MAIN16 of feature 113 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209041216 of field
## SA2_MAIN16 of feature 114 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209041217 of field
## SA2_MAIN16 of feature 115 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209041219 of field
## SA2_MAIN16 of feature 116 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209041220 of field
## SA2_MAIN16 of feature 117 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209041221 of field
## SA2_MAIN16 of feature 118 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209041223 of field
## SA2_MAIN16 of feature 119 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209041224 of field
## SA2_MAIN16 of feature 120 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209041225 of field
## SA2_MAIN16 of feature 121 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209041430 of field
## SA2_MAIN16 of feature 122 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209041431 of field
## SA2_MAIN16 of feature 123 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209041432 of field
## SA2_MAIN16 of feature 124 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209041433 of field
## SA2_MAIN16 of feature 125 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209041434 of field
## SA2_MAIN16 of feature 126 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209041435 of field
## SA2_MAIN16 of feature 127 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209041436 of field
## SA2_MAIN16 of feature 128 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 209041437 of field
## SA2_MAIN16 of feature 129 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210011226 of field
## SA2_MAIN16 of feature 130 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210011227 of field
## SA2_MAIN16 of feature 131 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210011228 of field
## SA2_MAIN16 of feature 132 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210011229 of field
## SA2_MAIN16 of feature 133 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210011230 of field
## SA2_MAIN16 of feature 134 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210011231 of field
## SA2_MAIN16 of feature 135 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210021232 of field
## SA2_MAIN16 of feature 136 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210021233 of field
## SA2_MAIN16 of feature 137 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210021234 of field
## SA2_MAIN16 of feature 138 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210021235 of field
## SA2_MAIN16 of feature 139 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210031236 of field
## SA2_MAIN16 of feature 140 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210031237 of field
## SA2_MAIN16 of feature 141 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210031239 of field
## SA2_MAIN16 of feature 142 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210031438 of field
## SA2_MAIN16 of feature 143 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210031439 of field
## SA2_MAIN16 of feature 144 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210031440 of field
## SA2_MAIN16 of feature 145 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210041240 of field
## SA2_MAIN16 of feature 146 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210041241 of field
## SA2_MAIN16 of feature 147 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210051242 of field
## SA2_MAIN16 of feature 148 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210051243 of field
## SA2_MAIN16 of feature 149 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210051245 of field
## SA2_MAIN16 of feature 150 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210051246 of field
## SA2_MAIN16 of feature 151 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210051247 of field
## SA2_MAIN16 of feature 152 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210051248 of field
## SA2_MAIN16 of feature 153 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210051249 of field
## SA2_MAIN16 of feature 154 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210051250 of field
## SA2_MAIN16 of feature 155 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210051441 of field
## SA2_MAIN16 of feature 156 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210051442 of field
## SA2_MAIN16 of feature 157 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210051443 of field
## SA2_MAIN16 of feature 158 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210051444 of field
## SA2_MAIN16 of feature 159 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 210051445 of field
## SA2_MAIN16 of feature 160 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211011251 of field
## SA2_MAIN16 of feature 161 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211011254 of field
## SA2_MAIN16 of feature 162 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211011255 of field
## SA2_MAIN16 of feature 163 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211011256 of field
## SA2_MAIN16 of feature 164 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211011257 of field
## SA2_MAIN16 of feature 165 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211011258 of field
## SA2_MAIN16 of feature 166 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211011259 of field
## SA2_MAIN16 of feature 167 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211011260 of field
## SA2_MAIN16 of feature 168 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211011446 of field
## SA2_MAIN16 of feature 169 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211011447 of field
## SA2_MAIN16 of feature 170 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211011448 of field
## SA2_MAIN16 of feature 171 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211011449 of field
## SA2_MAIN16 of feature 172 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211021261 of field
## SA2_MAIN16 of feature 173 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211021262 of field
## SA2_MAIN16 of feature 174 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211031263 of field
## SA2_MAIN16 of feature 175 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211031265 of field
## SA2_MAIN16 of feature 176 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211031266 of field
## SA2_MAIN16 of feature 177 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211031267 of field
## SA2_MAIN16 of feature 178 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211031268 of field
## SA2_MAIN16 of feature 179 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211031450 of field
## SA2_MAIN16 of feature 180 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211031451 of field
## SA2_MAIN16 of feature 181 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211031452 of field
## SA2_MAIN16 of feature 182 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211041269 of field
## SA2_MAIN16 of feature 183 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211041270 of field
## SA2_MAIN16 of feature 184 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211041271 of field
## SA2_MAIN16 of feature 185 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211041272 of field
## SA2_MAIN16 of feature 186 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211041273 of field
## SA2_MAIN16 of feature 187 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211051274 of field
## SA2_MAIN16 of feature 188 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211051275 of field
## SA2_MAIN16 of feature 189 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211051276 of field
## SA2_MAIN16 of feature 190 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211051277 of field
## SA2_MAIN16 of feature 191 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211051278 of field
## SA2_MAIN16 of feature 192 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211051279 of field
## SA2_MAIN16 of feature 193 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211051280 of field
## SA2_MAIN16 of feature 194 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211051281 of field
## SA2_MAIN16 of feature 195 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211051282 of field
## SA2_MAIN16 of feature 196 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211051283 of field
## SA2_MAIN16 of feature 197 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211051284 of field
## SA2_MAIN16 of feature 198 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211051285 of field
## SA2_MAIN16 of feature 199 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 211051286 of field
## SA2_MAIN16 of feature 200 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212011287 of field
## SA2_MAIN16 of feature 201 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212011288 of field
## SA2_MAIN16 of feature 202 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212011289 of field
## SA2_MAIN16 of feature 203 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212011290 of field
## SA2_MAIN16 of feature 204 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212011291 of field
## SA2_MAIN16 of feature 205 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212011292 of field
## SA2_MAIN16 of feature 206 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212021293 of field
## SA2_MAIN16 of feature 207 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212021294 of field
## SA2_MAIN16 of feature 208 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212021295 of field
## SA2_MAIN16 of feature 209 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212021297 of field
## SA2_MAIN16 of feature 210 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212021299 of field
## SA2_MAIN16 of feature 211 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212021453 of field
## SA2_MAIN16 of feature 212 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212021454 of field
## SA2_MAIN16 of feature 213 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212021455 of field
## SA2_MAIN16 of feature 214 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212021456 of field
## SA2_MAIN16 of feature 215 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212031300 of field
## SA2_MAIN16 of feature 216 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212031301 of field
## SA2_MAIN16 of feature 217 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212031302 of field
## SA2_MAIN16 of feature 218 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212031303 of field
## SA2_MAIN16 of feature 219 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212031304 of field
## SA2_MAIN16 of feature 220 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212031305 of field
## SA2_MAIN16 of feature 221 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212031306 of field
## SA2_MAIN16 of feature 222 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212031308 of field
## SA2_MAIN16 of feature 223 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212031457 of field
## SA2_MAIN16 of feature 224 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212031458 of field
## SA2_MAIN16 of feature 225 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212041309 of field
## SA2_MAIN16 of feature 226 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212041310 of field
## SA2_MAIN16 of feature 227 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212041311 of field
## SA2_MAIN16 of feature 228 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212041312 of field
## SA2_MAIN16 of feature 229 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212041313 of field
## SA2_MAIN16 of feature 230 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212041314 of field
## SA2_MAIN16 of feature 231 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212041316 of field
## SA2_MAIN16 of feature 232 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212041317 of field
## SA2_MAIN16 of feature 233 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212041318 of field
## SA2_MAIN16 of feature 234 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212041459 of field
## SA2_MAIN16 of feature 235 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212041460 of field
## SA2_MAIN16 of feature 236 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212051319 of field
## SA2_MAIN16 of feature 237 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212051320 of field
## SA2_MAIN16 of feature 238 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212051321 of field
## SA2_MAIN16 of feature 239 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212051322 of field
## SA2_MAIN16 of feature 240 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212051323 of field
## SA2_MAIN16 of feature 241 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212051324 of field
## SA2_MAIN16 of feature 242 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212051325 of field
## SA2_MAIN16 of feature 243 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212051326 of field
## SA2_MAIN16 of feature 244 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 212051327 of field
## SA2_MAIN16 of feature 245 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213011328 of field
## SA2_MAIN16 of feature 246 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213011329 of field
## SA2_MAIN16 of feature 247 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213011330 of field
## SA2_MAIN16 of feature 248 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213011331 of field
## SA2_MAIN16 of feature 249 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213011332 of field
## SA2_MAIN16 of feature 250 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213011333 of field
## SA2_MAIN16 of feature 251 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213011334 of field
## SA2_MAIN16 of feature 252 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213011335 of field
## SA2_MAIN16 of feature 253 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213011336 of field
## SA2_MAIN16 of feature 254 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213011337 of field
## SA2_MAIN16 of feature 255 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213011338 of field
## SA2_MAIN16 of feature 256 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213011339 of field
## SA2_MAIN16 of feature 257 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213011340 of field
## SA2_MAIN16 of feature 258 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213021341 of field
## SA2_MAIN16 of feature 259 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213021342 of field
## SA2_MAIN16 of feature 260 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213021343 of field
## SA2_MAIN16 of feature 261 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213021344 of field
## SA2_MAIN16 of feature 262 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213021345 of field
## SA2_MAIN16 of feature 263 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213021346 of field
## SA2_MAIN16 of feature 264 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213031347 of field
## SA2_MAIN16 of feature 265 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213031348 of field
## SA2_MAIN16 of feature 266 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213031349 of field
## SA2_MAIN16 of feature 267 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213031350 of field
## SA2_MAIN16 of feature 268 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213031351 of field
## SA2_MAIN16 of feature 269 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213031352 of field
## SA2_MAIN16 of feature 270 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213041353 of field
## SA2_MAIN16 of feature 271 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213041355 of field
## SA2_MAIN16 of feature 272 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213041356 of field
## SA2_MAIN16 of feature 273 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213041357 of field
## SA2_MAIN16 of feature 274 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213041358 of field
## SA2_MAIN16 of feature 275 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213041359 of field
## SA2_MAIN16 of feature 276 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213041360 of field
## SA2_MAIN16 of feature 277 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213041461 of field
## SA2_MAIN16 of feature 278 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213041462 of field
## SA2_MAIN16 of feature 279 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213041463 of field
## SA2_MAIN16 of feature 280 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213051361 of field
## SA2_MAIN16 of feature 281 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213051362 of field
## SA2_MAIN16 of feature 282 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213051363 of field
## SA2_MAIN16 of feature 283 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213051365 of field
## SA2_MAIN16 of feature 284 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213051366 of field
## SA2_MAIN16 of feature 285 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213051368 of field
## SA2_MAIN16 of feature 286 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213051369 of field
## SA2_MAIN16 of feature 287 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213051464 of field
## SA2_MAIN16 of feature 288 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213051465 of field
## SA2_MAIN16 of feature 289 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213051466 of field
## SA2_MAIN16 of feature 290 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213051467 of field
## SA2_MAIN16 of feature 291 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 213051468 of field
## SA2_MAIN16 of feature 292 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 214011370 of field
## SA2_MAIN16 of feature 293 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 214011371 of field
## SA2_MAIN16 of feature 294 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 214011372 of field
## SA2_MAIN16 of feature 295 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 214011373 of field
## SA2_MAIN16 of feature 296 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 214011374 of field
## SA2_MAIN16 of feature 297 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 214011375 of field
## SA2_MAIN16 of feature 298 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 214011376 of field
## SA2_MAIN16 of feature 299 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 214021377 of field
## SA2_MAIN16 of feature 300 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 214021378 of field
## SA2_MAIN16 of feature 301 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 214021379 of field
## SA2_MAIN16 of feature 302 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 214021380 of field
## SA2_MAIN16 of feature 303 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 214021381 of field
## SA2_MAIN16 of feature 304 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 214021382 of field
## SA2_MAIN16 of feature 305 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 214021383 of field
## SA2_MAIN16 of feature 306 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 214021384 of field
## SA2_MAIN16 of feature 307 not successfully written. Possibly due to too
## larger number with respect to field width
```

```
## Warning in CPL_write_ogr(obj, dsn, layer, driver,
## as.character(dataset_options), : GDAL Message 1: Value 214021385 of field
## SA2_MAIN16 of feature 308 not successfully written. Possibly due to too
## larger number with respect to field width
```

