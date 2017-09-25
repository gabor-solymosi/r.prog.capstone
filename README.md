[![Travis-CI Build Status](https://travis-ci.org/Solymi90/r.prog.capstone.svg?branch=master)](https://travis-ci.org/Solymi90/r.prog.capstone)

# Mastering Software Development in R 
# Capstone Project - NOAA Earthquake Visualizations
#### Created by: Gábor Solymosi   

This is an R package created for the purpose of visualizing NOAA earthquake data. It processes data from [NOAA database](https://www.ngdc.noaa.gov/nndc/struts/form?t=101650&s=1&d=1). 

## Description

The package includes several exported functions to handle NOAA data. The provided data set includes data on earthquakes starting year 2150 B.C. and contains dates, locations, magnitudes, severity (casualties, injuries...) and other details. 

This package handles basic data cleaning using function `eq_clean_data()` and then two types of visualizations. The first is a `ggplot2`-based earthquake timeline of selected earthquakes using `geom_timeline()` and `geom_timeline_label()` with optional usage of `theme_timeline()` function. The second visualization is based on `leaflet` package and shows the earthquakes with some basic parameters on a map.

```{r, echo = FALSE, include = FALSE}
library(r.prog.capstone)
library(dplyr)
library(ggplot2)
library(readr)
library(grid)
library(lubridate)
library(leaflet)
library(scales)
```

The readme file gives a brief overview of the r.prog.capstone R package created for the purpose of visualizing NOAA earthquake data. It processes data from [NOAA database](https://www.ngdc.noaa.gov/nndc/struts/form?t=101650&s=1&d=1) and visualizes them using `ggplot2` and `leaflet` packages. 

## Installation

To install the package type the following (the development version from Github):

```
library(devtools)
install_github("Solymi90/r.prog.capstone")
library(r.prog.capstone)
```

## Package functions

There are six exported functions available to users:

- `eq_clean_data()`
- `geom_timeline()`
- `geom_timeline_label()`
- `theme_timeline()`
- `eq_create_label()`
- `eq_map()`

Please find a short description with examples on how to use the functions. The example data from NOAA can be found in the package directory under `\extdata` folder.

## Clean data

The first function is required to clean data for the visualization. It creates a DATE column in `Date` format, transforms latitude and longitude to numeric format and trims country from LOCATION_NAME.

```{r eq_read_example, message = FALSE}
filename <- system.file("extdata/earthquakes.tsv.gz", package = "earthquakeVis")
data <- readr::read_delim(filename, delim = "\t")

eq_clean_data(data)
```

## Visualize earthquake timeline

The next three functions utilize `ggplot2` package to visualize earthquake timeline. The basic `geom_timeline()` geom requires clean data from the previous paragraph. The required aesthetics is `x` with dates, optional are `y` for grouping by country, and `size` and `color` that can be use according to user needs. The `geom_timeline_label()` function requires additional `label` aesthetic for labeling. For better visualization of these two geoms, `theme_timeline()` theme was added. Here is an example:

```{r eq_timeline_example, fig.width = 7, fig.height = 4}
data %>% eq_clean_data() %>%
     filter(COUNTRY %in% c("GREECE", "ITALY"), YEAR > 2000) %>%
     ggplot(aes(x = DATE,
                y = COUNTRY,
                color = as.numeric(TOTAL_DEATHS),
                size = as.numeric(EQ_PRIMARY)
     )) +
     geom_timeline() +
     geom_timeline_label(aes(label = LOCATION_NAME), n_max = 5) +
     theme_timeline() +
     labs(size = "Richter scale value", color = "# deaths")
```

## Visualize earthquakes on map

The package utilized `leaflet` functions to visualize earthquakes on a map using `eq_map()` function. The map is automatically trimmed to display the input data frame. Optional annotations can be created using `eq_create_label()` function. The result is an interactive map where user can click on individual points to get details:

```{r eq_map_example, fig.width = 7, fig.height = 4}
data %>% 
  eq_clean_data() %>% 
  dplyr::filter(COUNTRY == "MEXICO" & lubridate::year(DATE) >= 2000) %>% 
  dplyr::mutate(popup_text = eq_create_label(.)) %>% 
  eq_map(annot_col = "popup_text")
```
