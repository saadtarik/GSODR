---
title: "Working with spatial and climate data from GSODR"
author: "Adam H Sparks"
date: "2017-01-13"
output: rmarkdown::html_vignette
vignette: >
  %\VignetteEngine{knitr::knitr}
  %\VignetteIndexEntry{Working with spatial and climate data from GSODR}
  %\VignetteEncoding{UTF-8}
---

# Introduction

The GSODR package provides the ability to interact with GSOD data using
spatial methods. The `get_GSOD()` function allows for the data to be saved as
a GeoPackage file which can be read by most GIS software packages or in R using
R's GIS capabilities with contributed packages as well.

Following is an example of how you might download and save GSOD annual data for
a given country, Philippines in this example, and convert it into a KML file for
viewing in GoogleEarth. The second portion uses the same GeoPackage file to
import the data back into R and combine the GSOD data with CHELSA data from the
[GSODRdata package](https://github.com/adamhsparks/GSODRdata) available from
GitHub and plot the station temperatures for daily GSOD, average monthly GSOD
and CHELSA temperatures (1979-2013). 

## Example - Download and plot data for a single country
Download data for Philippines for year 2010 and generate a spatial, year summary
file, PHL-2010.gpkg, in the user's home directory.

### Install GSODRdata package

This package is only available from GitHub; due to its large size (5.5Mb
installed) it is not allowed on CRAN. It provides optional data for use with the
GSODR package.  See https://github.com/adamhsparks/GSODRdata for more.

```r
install.packages("devtools")
devtools::install_github("adamhsparks/GSOD.data")
```

### Working with climate data from GSODRdata

Now that the extra data have been installed, take a look at the CHELSA data.
CHELSA (Climatologies at high resolution for the earth’s land surface areas) are
climate data at (30 arc sec) for the earth land surface areas. 

**Description of CHELSA data from CHELSA website**

> CHELSA is a high resolution (30 arc sec) climate data set for the earth land surface areas currently under development in coorporation with the Department of Geography of the University of Hamburg (Prof. Dr. Jürgen Böhner, Dr. Olaf Conrad, Tobias Kawohl), the Swiss Federal Institute for Forest, Snow and Landscape Research WSL (Prof. Dr. Niklaus Zimmermann), the University of Zurich (Dr. Dirk N. Karger, Dr. Michael Kessler), and the University of Göttingen (Prof. Dr. Holger Kreft).
It includes monthly mean temperature and precipitation patterns for the time period 1979-2013.
CHELSA is based on a quasi-mechanistical statistical downscaling of the ERA interim global circulation model (http://www.ecmwf.int/en/research/climate-reanalysis/era-interim) with a GPCC bias correction (https://www.dwd.de/EN/ourservices/gpcc/gpcc.html) and is freely available  in the download section.

See http://chelsa-climate.org for more information on these data.

```r
library(GSODR)
library(GSODRdata)
head(CHELSA)
```

```
##          STNID    LON    LAT CHELSA_bio10_1979-2013_V1_1 CHELSA_bio11_1979-2013_V1_1 CHELSA_bio1_1979-2013_V1_1
## 1 008268-99999 65.567 32.950                        30.0                         5.8                       18.3
## 2 010014-99999  5.341 59.792                        14.3                         1.0                        7.1
## 3 010015-99999  5.867 61.383                        12.5                        -3.2                        4.2
## 4 010882-99999 11.342 62.578                        12.6                        -7.0                        2.3
## 5 010883-99999  6.117 61.833                        15.2                        -0.2                        7.0
## 6 010884-99999  9.567 59.185                        16.3                        -2.2                        6.6
##   CHELSA_bio12_1979-2013_V1_1 CHELSA_bio13_1979-2013_V1_1 CHELSA_bio14_1979-2013_V1_1 CHELSA_bio15_1979-2013_V1_1
## 1                       214.0                        54.6                         0.1                       104.9
## 2                      1889.3                       223.7                        87.7                        30.3
## 3                      2209.1                       254.7                       108.9                        30.1
## 4                       563.0                        73.0                        26.7                        30.7
## 5                      1710.3                       200.6                        78.5                        32.1
## 6                       987.0                       120.9                        52.3                        24.4
##   CHELSA_bio16_1979-2013_V1_1 CHELSA_bio17_1979-2013_V1_1 CHELSA_bio18_1979-2013_V1_1 CHELSA_bio19_1979-2013_V1_1
## 1                       155.4                         0.6                         1.7                        96.9
## 2                       650.7                       273.5                       433.3                       490.4
## 3                       752.8                       332.6                       448.3                       621.1
## 4                       216.2                        85.5                       198.7                       122.0
## 5                       590.0                       244.7                       330.9                       494.4
## 6                       338.0                       175.3                       242.3                       182.7
##   CHELSA_bio2_1979-2013_V1_1 CHELSA_bio3_1979-2013_V1_1 CHELSA_bio4_1979-2013_V1_1 CHELSA_bio5_1979-2013_V1_1
## 1                       23.0                       48.3                      884.1                       40.5
## 2                       12.5                       44.8                      486.3                       21.5
## 3                       16.7                       45.3                      591.5                       22.2
## 4                       20.1                       45.5                      734.1                       23.6
## 5                       15.6                       45.0                      573.4                       23.9
## 6                       17.1                       44.3                      693.1                       25.4
##   CHELSA_bio6_1979-2013_V1_1 CHELSA_bio7_1979-2013_V1_1 CHELSA_bio8_1979-2013_V1_1 CHELSA_bio9_1979-2013_V1_1 CHELSA_prec_10_1979-2013
## 1                       -7.2                       47.7                       10.5                       26.7                      3.6
## 2                       -6.5                       28.0                        5.7                        7.8                    230.6
## 3                      -14.5                       36.8                       -1.7                        8.4                    237.5
## 4                      -20.7                       44.3                       12.5                       -0.1                     48.9
## 5                      -10.7                       34.7                        1.3                       12.2                    192.8
## 6                      -13.3                       38.7                        8.5                        3.8                    122.1
##   CHELSA_prec_11_1979-2013 CHELSA_prec_1_1979-2013 CHELSA_prec_12_1979-2013 CHELSA_prec_1979-2013_land CHELSA_prec_2_1979-2013
## 1                     10.5                    35.8                     25.4                      214.5                    46.5
## 2                    219.9                   198.4                    215.5                     1912.0                   153.3
## 3                    237.8                   227.7                    248.3                     2170.2                   188.4
## 4                     41.3                    40.3                     40.5                      559.3                    32.5
## 5                    189.6                   189.5                    201.5                     1719.6                   153.7
## 6                    106.2                    79.0                     82.4                      998.1                    53.0
##  CHELSA_prec_3_1979-2013 CHELSA_prec_4_1979-2013 CHELSA_prec_5_1979-2013 CHELSA_prec_6_1979-2013 CHELSA_prec_7_1979-2013
## 1                    54.8                    28.3                     7.0                     1.2                     1.0
## 2                   151.9                   101.1                    90.6                   100.5                   116.7
## 3                   176.4                   114.4                   107.0                   110.1                   119.2
## 4                    31.8                    26.6                    38.9                    58.0                    69.7
## 5                   145.0                    91.6                    79.0                    84.0                    90.1
## 6                    63.7                    57.0                    72.1                    76.1                    84.3
##   CHELSA_prec_8_1979-2013 CHELSA_prec_9_1979-2013 CHELSA_temp_10_1979-2013 CHELSA_temp_11_1979-2013 CHELSA_temp_1_1979-2013
## 1                     0.3                     0.1                     19.2                     13.2                     4.9
## 2                   165.6                   204.0                      8.0                      4.5                     1.2
## 3                   159.6                   229.6                      4.3                      0.2                    -3.4
## 4                    72.7                    57.9                      2.3                     -3.0                    -7.2
## 5                   121.6                   181.4                      7.2                      3.0                    -0.4
## 6                   105.0                    97.2                      6.7                      2.2                    -2.4
##   CHELSA_temp_12_1979-2013 CHELSA_temp_1979-2013_land CHELSA_temp_2_1979-2013 CHELSA_temp_3_1979-2013 CHELSA_temp_4_1979-2013
## 1                      7.7                       18.3                     7.0                    12.2                    18.3
## 2                      2.1                        7.1                     0.8                     2.3                     5.1
## 3                     -2.7                        4.1                    -3.2                    -0.8                     3.0
## 4                     -6.5                        2.3                    -6.6                    -3.3                     1.5
## 5                      0.3                        6.9                    -0.2                     2.2                     5.8
## 6                     -1.4                        6.5                    -2.2                     0.8                     5.1
##   CHELSA_temp_5_1979-2013 CHELSA_temp_6_1979-2013 CHELSA_temp_7_1979-2013 CHELSA_temp_8_1979-2013 ## CHELSA_temp_9_1979-2013
## 1                    23.5                    28.2                    28.2                    29.7                    25.1
## 2                     9.0                    12.0                    12.0                    14.2                    11.5
## 3                     7.6                    10.9                    10.9                    12.2                     8.6
## 4                     6.4                    10.9                    10.9                    12.1                     7.7
## 5                    10.1                    13.2                    13.2                    14.9                    11.5
## 6                    10.7                    14.7                    14.7                    15.8                    11.6
```

```r
get_GSOD(years = 2010, country = "Philippines", dsn = "~/",
         filename = "PHL-2010", GPKG = TRUE, max_missing = 5)
```


```r
library(rgdal)
```

```
## Loading required package: sp
```

```
## rgdal: version: 1.2-4, (SVN revision 643)
##  Geospatial Data Abstraction Library extensions to R successfully loaded
##  Loaded GDAL runtime: GDAL 1.11.5, released 2016/07/01
##  Path to GDAL shared files: /usr/local/Cellar/gdal/1.11.5_1/share/gdal
##  Loaded PROJ.4 runtime: Rel. 4.9.3, 15 August 2016, [PJ_VERSION: 493]
##  Path to PROJ.4 shared files: (autodetected)
##  Linking to sp version: 1.2-3
```

```r
library(spacetime)
library(plotKML)
```

```
## plotKML version 0.5-6 (2016-05-02)
```

```
## URL: http://plotkml.r-forge.r-project.org/
```

```r
layers <- ogrListLayers(dsn = path.expand("~/PHL-2010.gpkg"))
pnts <- readOGR(dsn = path.expand("~/PHL-2010.gpkg"), layers[1])
```

```
## OGR data source with driver: GPKG 
## Source: "/Users/U8004755/PHL-2010.gpkg", layer: "GSOD"
## with 4703 features
## It has 46 fields
```

```r
# Plot results in Google Earth as a spacetime object:
pnts$DATE = as.Date(paste(pnts$YEAR, pnts$MONTH, pnts$DAY, sep = "-"))
row.names(pnts) <- paste("point", 1:nrow(pnts), sep = "")

tmp_ST <- STIDF(sp = as(pnts, "SpatialPoints"),
                time = pnts$DATE - 0.5,
                data = pnts@data[, c("TEMP", "STNID")],
                endTime = pnts$DATE + 0.5)

shape = "http://maps.google.com/mapfiles/kml/pal2/icon18.png"

kml(tmp_ST, dtime = 24 * 3600, colour = TEMP, shape = shape, labels = TEMP,
    file.name = "Temperatures_PHL_2010-2010.kml", folder.name = "TEMP")
```

```
## KML file opened for writing...
```

```
## Writing to KML...
```

```
## Closing  Temperatures_PHL_2010-2010.kml
```

```r
system("zip -m Temperatures_PHL_2010-2010.kmz Temperatures_PHL_2010-2010.kml")
```

Compare the GSOD weather data from the Philippines with climatic data provided
by the GSODRdata package in the `CHELSA` data set.


```r
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
library(ggplot2)
library(reshape2)

data(GSOD_clim)
cnames <- paste0("CHELSA_temp_", 1:12, "_1979-2013")
clim_temp <- GSOD_clim[GSOD_clim$STNID %in% pnts$STNID,
                       paste(c("STNID", cnames))]
clim_temp_df <- data.frame(STNID = rep(clim_temp$STNID, 12),
                           MONTHC = as.vector(sapply(1:12, rep,
                                                    times = nrow(clim_temp))), 
                           CHELSA_TEMP = as.vector(unlist(clim_temp[, cnames])))

pnts$MONTHC <- as.numeric(paste(pnts$MONTH))
temp <- left_join(pnts@data, clim_temp_df, by = c("STNID", "MONTHC"))
```

```
## Warning in left_join_impl(x, y, by$x, by$y, suffix$x, suffix$y): joining
## factors with different levels, coercing to character vector
```

```r
temp <- temp %>% 
  group_by(MONTH) %>% 
  mutate(AVG_DAILY_TEMP = round(mean(TEMP), 1))

df_melt <- na.omit(melt(temp[, c("STNID", "DATE", "CHELSA_TEMP", "TEMP", "AVG_DAILY_TEMP")],
                        id = c("DATE", "STNID")))

ggplot(df_melt, aes(x = DATE, y = value)) +
  geom_point(aes(color = variable), alpha = 0.2) +
  scale_x_date(date_labels = "%b") +
  ylab("Temperature (C)") +
  xlab("Month") +
  labs(colour = "") +
  scale_color_brewer(palette = "Dark2") +
  facet_wrap( ~ STNID)
```

![Comparison of GSOD daily values and average monthly values with CHELSA climate monthly values](figure/example_1.2-1.png)

# Notes

## Sources

#### CHELSA climate layers
CHELSA (climatic surfaces at 1 km resolution) is based on a quasi-mechanistical
statistical downscaling of the ERA interim global circulation model
(Karger et al. 2016). ESA's CCI-LC cloud probability monthly averages are based
on the MODIS snow products (MOD10A2). <http://chelsa-climate.org/>

#### EarthEnv MODIS cloud fraction 
<http://www.earthenv.org/cloud>

#### ESA's CCI-LC cloud probability
<http://maps.elie.ucl.ac.be/CCI/viewer/index.php>

#### Elevation Values

90m hole-filled SRTM digital elevation (Jarvis *et al.* 2008) was used
to identify and correct/remove elevation errors in data for station
locations between -60˚ and 60˚ latitude. This applies to cases here
where elevation was missing in the reported values as well. In case the
station reported an elevation and the DEM does not, the station reported
is taken. For stations beyond -60˚ and 60˚ latitude, the values are
station reported values in every instance. See
<https://github.com/adamhsparks/GSODR/blob/devel/data-raw/fetch_isd-history.md>
for more detail on the correction methods.

## WMO Resolution 40. NOAA Policy

*Users of these data should take into account the following (from the [NCDC website](http://www7.ncdc.noaa.gov/CDO/cdoselect.cmd?datasetabbv=GSOD&countryabbv=&georegionabbv=)):*

> "The following data and products may have conditions placed on their 
international commercial use. They can be used within the U.S. or for
non-commercial international activities without restriction. The
non-U.S. data cannot be redistributed for commercial purposes.
Re-distribution of these data by others must provide this same
notification." [WMO Resolution 40. NOAA
Policy](http://www.wmo.int/pages/about/Resolution40.html)

# References
Stachelek, J. 2016. Using the Geopackage Format with R. 
URL: https://jsta.github.io/2016/07/14/geopackage-r.html