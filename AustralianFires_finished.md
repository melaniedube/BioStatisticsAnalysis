---
title: "Australian Fires"
author: "Melanie Dube"
format: html
editor: visual
execute:
  keep-md: true
---



# Biostatistics Analysis 2: Australian Fires

## Abstract

This notebook examines data on temperature and rainfall in Australia, as well as data on a number of fires around the country. There are a number of questions which can be generated from this data, including how rainfall compares to temperature, what region of the country has the most fires, and how temperature correlates to chance of fire.

## Introduction

The climate has been rather unstable around the world in recent years, and seems to be changing at a rapid rate. Climate change is often associated with or attributed to altered weather patterns (including abnormal storms and temperatures), and rising sea levels. Lack of rainfall and higher temperatures are often a prime environment for an uncontrollable fire, which can cause irreversible damage to the flora and fauna in a region.

### Questions to Consider

-   How does rainfall compare to temperature?

-   Does higher temperature correlate with higher chance of fires?

-   Are the fires concentrated to one region of the country?

-   Which region/city has the highest temperatures?

-   Where is the most rain falling in the region, on average?

### Loading the packages


::: {.cell}

```{.r .cell-code}
#Load the tidyverse
library(tidyverse)
```

::: {.cell-output .cell-output-stderr}
```
── Attaching packages ─────────────────────────────────────── tidyverse 1.3.2 ──
✔ ggplot2 3.4.0      ✔ purrr   1.0.0 
✔ tibble  3.1.8      ✔ dplyr   1.0.10
✔ tidyr   1.2.1      ✔ stringr 1.5.0 
✔ readr   2.1.3      ✔ forcats 0.5.2 
── Conflicts ────────────────────────────────────────── tidyverse_conflicts() ──
✖ dplyr::filter() masks stats::filter()
✖ dplyr::lag()    masks stats::lag()
```
:::

```{.r .cell-code}
library(kableExtra)
```

::: {.cell-output .cell-output-stderr}
```

Attaching package: 'kableExtra'

The following object is masked from 'package:dplyr':

    group_rows
```
:::

```{.r .cell-code}
#install.packages("tidymodels")
library(tidymodels)
```

::: {.cell-output .cell-output-stderr}
```
── Attaching packages ────────────────────────────────────── tidymodels 1.0.0 ──
✔ broom        1.0.2     ✔ rsample      1.1.1
✔ dials        1.1.0     ✔ tune         1.0.1
✔ infer        1.0.4     ✔ workflows    1.1.2
✔ modeldata    1.1.0     ✔ workflowsets 1.0.0
✔ parsnip      1.0.3     ✔ yardstick    1.1.0
✔ recipes      1.0.4     
── Conflicts ───────────────────────────────────────── tidymodels_conflicts() ──
✖ scales::discard()        masks purrr::discard()
✖ dplyr::filter()          masks stats::filter()
✖ recipes::fixed()         masks stringr::fixed()
✖ kableExtra::group_rows() masks dplyr::group_rows()
✖ dplyr::lag()             masks stats::lag()
✖ yardstick::spec()        masks readr::spec()
✖ recipes::step()          masks stats::step()
• Use tidymodels_prefer() to resolve common conflicts.
```
:::

```{.r .cell-code}
#install.packages("skimr")
library(skimr)

#install.packages("sf")
```
:::


### Loading the data

One of the primary goals of compiling this data set is to spread awareness of these fires. There are many sources which report climate data on the region, which include the temperature and rainfall statistics.


::: {.cell}

```{.r .cell-code}
# Get the Data

# Get the Data

rainfall <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-01-07/rainfall.csv')
```

::: {.cell-output .cell-output-stderr}
```
Rows: 179273 Columns: 11
── Column specification ────────────────────────────────────────────────────────
Delimiter: ","
chr (6): station_code, city_name, month, day, quality, station_name
dbl (5): year, rainfall, period, lat, long

ℹ Use `spec()` to retrieve the full column specification for this data.
ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```
:::

```{.r .cell-code}
temperature <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-01-07/temperature.csv')
```

::: {.cell-output .cell-output-stderr}
```
Rows: 528278 Columns: 5
── Column specification ────────────────────────────────────────────────────────
Delimiter: ","
chr  (3): city_name, temp_type, site_name
dbl  (1): temperature
date (1): date

ℹ Use `spec()` to retrieve the full column specification for this data.
ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```
:::

```{.r .cell-code}
# IF YOU USE THIS DATA PLEASE BE CAUTIOUS WITH INTERPRETATION
nasa_fire <- readr::read_csv('https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2020/2020-01-07/MODIS_C6_Australia_and_New_Zealand_7d.csv')
```

::: {.cell-output .cell-output-stderr}
```
Rows: 34270 Columns: 13
── Column specification ────────────────────────────────────────────────────────
Delimiter: ","
chr  (4): acq_time, satellite, version, daynight
dbl  (8): latitude, longitude, brightness, scan, track, confidence, bright_t...
date (1): acq_date

ℹ Use `spec()` to retrieve the full column specification for this data.
ℹ Specify the column types or set `show_col_types = FALSE` to quiet this message.
```
:::

```{.r .cell-code}
# For JSON File of fires
url <- "http://www.rfs.nsw.gov.au/feeds/majorIncidents.json"

aus_fires <- sf::st_read(url)
```

::: {.cell-output .cell-output-stdout}
```
Reading layer `majorIncidents' from data source 
  `http://www.rfs.nsw.gov.au/feeds/majorIncidents.json' using driver `GeoJSON'
Simple feature collection with 41 features and 7 fields
Geometry type: GEOMETRY
Dimension:     XY
Bounding box:  xmin: 142.4221 ymin: -37.10481 xmax: 153.1909 ymax: -28.47865
Geodetic CRS:  WGS 84
```
:::
:::


Above I have loaded the data from the tidytuesday page, which has two data sets (temperature and rainfall).

## Exploratory Analysis

To begin analyzing the data, we want to skim it. The skim function allows for grouping of similar data elements for more organized analysis. First, I want to look at specifics from the `temperature` data frame.


::: {.cell}

```{.r .cell-code}
library(skimr)
rainfall%>%
  skim()
```

::: {.cell-output-display}
<table style='width: auto;'
      class='table table-condensed'>
<caption>Data summary</caption>
<tbody>
  <tr>
   <td style="text-align:left;"> Name </td>
   <td style="text-align:left;"> Piped data </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Number of rows </td>
   <td style="text-align:left;"> 179273 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Number of columns </td>
   <td style="text-align:left;"> 11 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> _______________________ </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Column type frequency: </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> character </td>
   <td style="text-align:left;"> 6 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> numeric </td>
   <td style="text-align:left;"> 5 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ________________________ </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Group variables </td>
   <td style="text-align:left;"> None </td>
  </tr>
</tbody>
</table>


**Variable type: character**

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> skim_variable </th>
   <th style="text-align:right;"> n_missing </th>
   <th style="text-align:right;"> complete_rate </th>
   <th style="text-align:right;"> min </th>
   <th style="text-align:right;"> max </th>
   <th style="text-align:right;"> empty </th>
   <th style="text-align:right;"> n_unique </th>
   <th style="text-align:right;"> whitespace </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> station_code </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 6 </td>
   <td style="text-align:right;"> 6 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 7 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> city_name </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 5 </td>
   <td style="text-align:right;"> 9 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 6 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> month </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 12 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> day </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 31 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> quality </td>
   <td style="text-align:right;"> 11764 </td>
   <td style="text-align:right;"> 0.93 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> station_name </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 8 </td>
   <td style="text-align:right;"> 34 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 7 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
</tbody>
</table>


**Variable type: numeric**

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> skim_variable </th>
   <th style="text-align:right;"> n_missing </th>
   <th style="text-align:right;"> complete_rate </th>
   <th style="text-align:right;"> mean </th>
   <th style="text-align:right;"> sd </th>
   <th style="text-align:right;"> p0 </th>
   <th style="text-align:right;"> p25 </th>
   <th style="text-align:right;"> p50 </th>
   <th style="text-align:right;"> p75 </th>
   <th style="text-align:right;"> p100 </th>
   <th style="text-align:left;"> hist </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> year </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 1964.21 </td>
   <td style="text-align:right;"> 43.35 </td>
   <td style="text-align:right;"> 1858.00 </td>
   <td style="text-align:right;"> 1932.00 </td>
   <td style="text-align:right;"> 1976.00 </td>
   <td style="text-align:right;"> 2000.00 </td>
   <td style="text-align:right;"> 2020.00 </td>
   <td style="text-align:left;"> ▂▃▃▆▇ </td>
  </tr>
  <tr>
   <td style="text-align:left;"> rainfall </td>
   <td style="text-align:right;"> 11760 </td>
   <td style="text-align:right;"> 0.93 </td>
   <td style="text-align:right;"> 2.40 </td>
   <td style="text-align:right;"> 8.42 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 0.00 </td>
   <td style="text-align:right;"> 0.80 </td>
   <td style="text-align:right;"> 327.60 </td>
   <td style="text-align:left;"> ▇▁▁▁▁ </td>
  </tr>
  <tr>
   <td style="text-align:left;"> period </td>
   <td style="text-align:right;"> 114797 </td>
   <td style="text-align:right;"> 0.36 </td>
   <td style="text-align:right;"> 1.03 </td>
   <td style="text-align:right;"> 0.29 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 23.00 </td>
   <td style="text-align:left;"> ▇▁▁▁▁ </td>
  </tr>
  <tr>
   <td style="text-align:left;"> lat </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> -33.51 </td>
   <td style="text-align:right;"> 2.89 </td>
   <td style="text-align:right;"> -37.83 </td>
   <td style="text-align:right;"> -34.92 </td>
   <td style="text-align:right;"> -33.86 </td>
   <td style="text-align:right;"> -31.96 </td>
   <td style="text-align:right;"> -27.48 </td>
   <td style="text-align:left;"> ▂▇▂▁▂ </td>
  </tr>
  <tr>
   <td style="text-align:left;"> long </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1.00 </td>
   <td style="text-align:right;"> 143.40 </td>
   <td style="text-align:right;"> 11.12 </td>
   <td style="text-align:right;"> 115.79 </td>
   <td style="text-align:right;"> 138.60 </td>
   <td style="text-align:right;"> 149.20 </td>
   <td style="text-align:right;"> 151.21 </td>
   <td style="text-align:right;"> 153.05 </td>
   <td style="text-align:left;"> ▂▁▁▆▇ </td>
  </tr>
</tbody>
</table>
:::

```{.r .cell-code}
library(skimr)
temperature%>%
  skim()
```

::: {.cell-output-display}
<table style='width: auto;'
      class='table table-condensed'>
<caption>Data summary</caption>
<tbody>
  <tr>
   <td style="text-align:left;"> Name </td>
   <td style="text-align:left;"> Piped data </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Number of rows </td>
   <td style="text-align:left;"> 528278 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Number of columns </td>
   <td style="text-align:left;"> 5 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> _______________________ </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Column type frequency: </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> character </td>
   <td style="text-align:left;"> 3 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Date </td>
   <td style="text-align:left;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> numeric </td>
   <td style="text-align:left;"> 1 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> ________________________ </td>
   <td style="text-align:left;">  </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Group variables </td>
   <td style="text-align:left;"> None </td>
  </tr>
</tbody>
</table>


**Variable type: character**

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> skim_variable </th>
   <th style="text-align:right;"> n_missing </th>
   <th style="text-align:right;"> complete_rate </th>
   <th style="text-align:right;"> min </th>
   <th style="text-align:right;"> max </th>
   <th style="text-align:right;"> empty </th>
   <th style="text-align:right;"> n_unique </th>
   <th style="text-align:right;"> whitespace </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> city_name </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 4 </td>
   <td style="text-align:right;"> 9 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 7 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> temp_type </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:right;"> 3 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 2 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> site_name </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:right;"> 9 </td>
   <td style="text-align:right;"> 25 </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 7 </td>
   <td style="text-align:right;"> 0 </td>
  </tr>
</tbody>
</table>


**Variable type: Date**

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> skim_variable </th>
   <th style="text-align:right;"> n_missing </th>
   <th style="text-align:right;"> complete_rate </th>
   <th style="text-align:left;"> min </th>
   <th style="text-align:left;"> max </th>
   <th style="text-align:left;"> median </th>
   <th style="text-align:right;"> n_unique </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> date </td>
   <td style="text-align:right;"> 0 </td>
   <td style="text-align:right;"> 1 </td>
   <td style="text-align:left;"> 1910-01-01 </td>
   <td style="text-align:left;"> 2019-05-31 </td>
   <td style="text-align:left;"> 1967-10-04 </td>
   <td style="text-align:right;"> 39963 </td>
  </tr>
</tbody>
</table>


**Variable type: numeric**

<table>
 <thead>
  <tr>
   <th style="text-align:left;"> skim_variable </th>
   <th style="text-align:right;"> n_missing </th>
   <th style="text-align:right;"> complete_rate </th>
   <th style="text-align:right;"> mean </th>
   <th style="text-align:right;"> sd </th>
   <th style="text-align:right;"> p0 </th>
   <th style="text-align:right;"> p25 </th>
   <th style="text-align:right;"> p50 </th>
   <th style="text-align:right;"> p75 </th>
   <th style="text-align:right;"> p100 </th>
   <th style="text-align:left;"> hist </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> temperature </td>
   <td style="text-align:right;"> 3465 </td>
   <td style="text-align:right;"> 0.99 </td>
   <td style="text-align:right;"> 16.6 </td>
   <td style="text-align:right;"> 7.88 </td>
   <td style="text-align:right;"> -11.5 </td>
   <td style="text-align:right;"> 11.1 </td>
   <td style="text-align:right;"> 16.3 </td>
   <td style="text-align:right;"> 21.6 </td>
   <td style="text-align:right;"> 48.3 </td>
   <td style="text-align:left;"> ▁▅▇▂▁ </td>
  </tr>
</tbody>
</table>
:::

```{.r .cell-code}
temperature %>%
  head() %>%
  kable() %>%
  kable_styling(c("hover", "striped"))
```

::: {.cell-output-display}

`````{=html}
<table class="table table-hover table-striped" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> city_name </th>
   <th style="text-align:left;"> date </th>
   <th style="text-align:right;"> temperature </th>
   <th style="text-align:left;"> temp_type </th>
   <th style="text-align:left;"> site_name </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> PERTH </td>
   <td style="text-align:left;"> 1910-01-01 </td>
   <td style="text-align:right;"> 26.7 </td>
   <td style="text-align:left;"> max </td>
   <td style="text-align:left;"> PERTH AIRPORT </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PERTH </td>
   <td style="text-align:left;"> 1910-01-02 </td>
   <td style="text-align:right;"> 27.0 </td>
   <td style="text-align:left;"> max </td>
   <td style="text-align:left;"> PERTH AIRPORT </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PERTH </td>
   <td style="text-align:left;"> 1910-01-03 </td>
   <td style="text-align:right;"> 27.5 </td>
   <td style="text-align:left;"> max </td>
   <td style="text-align:left;"> PERTH AIRPORT </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PERTH </td>
   <td style="text-align:left;"> 1910-01-04 </td>
   <td style="text-align:right;"> 24.0 </td>
   <td style="text-align:left;"> max </td>
   <td style="text-align:left;"> PERTH AIRPORT </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PERTH </td>
   <td style="text-align:left;"> 1910-01-05 </td>
   <td style="text-align:right;"> 24.8 </td>
   <td style="text-align:left;"> max </td>
   <td style="text-align:left;"> PERTH AIRPORT </td>
  </tr>
  <tr>
   <td style="text-align:left;"> PERTH </td>
   <td style="text-align:left;"> 1910-01-06 </td>
   <td style="text-align:right;"> 24.4 </td>
   <td style="text-align:left;"> max </td>
   <td style="text-align:left;"> PERTH AIRPORT </td>
  </tr>
</tbody>
</table>

`````

:::
:::


One of the first things I was interested in was highest temperature. One thing that I noted about the data set `temperature` is that it is extremely large. When viewing all of the data in that set, there appears to be 528,278 entries, sorted into five columns.


::: {.cell}

```{.r .cell-code}
temperature %>%
  count(temperature) %>%
  arrange(-n)
```

::: {.cell-output .cell-output-stdout}
```
# A tibble: 571 × 2
   temperature     n
         <dbl> <int>
 1        NA    3465
 2        15.8  3005
 3        17.2  2996
 4        17    2956
 5        15.2  2938
 6        18.1  2885
 7        13.9  2879
 8        16.4  2857
 9        14.7  2855
10        16.5  2852
# … with 561 more rows
```
:::
:::


This command counts the number of times each temperature appears in the data set. It appears that the most common numeric temperature is 15.8 degrees C, which appears 3,005 times. At least 220 specific (xx.x) temperatures also are counted over 1,000 times, proving how large this data set really is.


::: {.cell}

```{.r .cell-code}
temperature %>%
  group_by(site_name) %>%
  count(site_name) %>%
  arrange(-n)
```

::: {.cell-output .cell-output-stdout}
```
# A tibble: 7 × 2
# Groups:   site_name [7]
  site_name                     n
  <chr>                     <int>
1 MELBOURNE (OLYMPIC PARK)  79926
2 PERTH AIRPORT             79926
3 KENT TOWN                 79924
4 PORT LINCOLN AWS          79924
5 SYDNEY (OBSERVATORY HILL) 79924
6 CANBERRA AIRPORT          77526
7 BRISBANE AERO             51128
```
:::
:::


The command above allows us to sort the locations by order of most frequent entries. Interestingly, the top five locations all have around the same number of entries (between 79,924 - 79,926). It also appears that most of, if not all, of the locations are airports. Based on this I assume that the temperature at each of the airports is being recorded around the same time for all locations.

\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~\~

Now, I wanted to look at data regarding specific locations. To do this, I switched to the `rainfall` data frame, which has latitudes and longitudes for each location. I needed to install and load the `mapview` library first.


::: {.cell}

```{.r .cell-code}
#install.packages("mapview")

library(mapview)
library(leaflet)
```
:::

::: {.cell}

```{.r .cell-code}
rainfall %>%
  leaflet() %>%
  addTiles() %>%
  addCircleMarkers(lng = rainfall$long, lat = rainfall$lat)
```

::: {.cell-output-display}

```{=html}
<div class="leaflet html-widget html-fill-item-overflow-hidden html-fill-item" id="htmlwidget-81f58607f2e024e4f1ad" style="width:100%;height:464px;"></div>
```

:::

```{.r .cell-code}
#mapview(rainfall, xcol = "long", ycol = "lat", crs = 4326, grid = TRUE)
```
:::


This command organizes each of the latitudes and longitudes and plots them on a map. This way, we are able to visualize where each of the locations are in the country.


::: {.cell}

```{.r .cell-code}
rainfall %>%
  group_by(year) %>%
  count(year)
```

::: {.cell-output .cell-output-stdout}
```
# A tibble: 163 × 2
# Groups:   year [163]
    year     n
   <dbl> <int>
 1  1858   365
 2  1859   365
 3  1860   366
 4  1861   365
 5  1862   365
 6  1863   365
 7  1864   366
 8  1865   365
 9  1866   365
10  1867   365
# … with 153 more rows
```
:::
:::


This shows that data has been collected for the last 163 years, with the most recent year appearing to be 2017. What is interesting about this data is that the earlier years have 365 observations exactly, which may indicate that only one location was being tracked every day. For the most current years, there are over 2,000 entries per year which could correlate to the each of the different locations being measured per year.

## Conclusions

This is a complex data set with many different variables to analyze and consider. Despite the data being called "Australian Fires", the data I analyzed was more focused on measuring temperature than the actual fires, which was too large of a data set to download. If I had more time and experience, this would be an interesting analysis to continue.