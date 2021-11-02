---
output: github_document
---

<!-- README.md is generated from README.Rmd. Please edit that file -->



# zonal <img src="man/figures/logo.png" align="right" alt="" width="120" />

<!-- badges: start -->
[![R CMD Check](https://github.com/mikejohnson51/zonal/actions/workflows/R-CMD-check.yaml/badge.svg)](https://github.com/mikejohnson51/zonal/actions/workflows/R-CMD-check.yaml)
[![Project Status: Active](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active)
[![LifeCycle](man/figures/lifecycle/lifecycle-experimental.svg)](https://lifecycle.r-lib.org/articles/stages.html#experimental)
[![Dependencies](https://img.shields.io/badge/dependencies-8/29-orange?style=flat)](#)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://choosealicense.com/licenses/mit/)
[![Website deployment](https://github.com/mikejohnson51/zonal/actions/workflows/pkgdown.yaml/badge.svg)](https://github.com/mikejohnson51/zonal/actions/workflows/pkgdown.yaml)
<!-- badges: end -->

`zonal` is an active package for intersecting vector aggregation units with large gridded data. While there are many libraries that seek to tackle this problem (see credits) we needed a library that could handle large gridded extents storing categorical and continuous data, with multiple time layers with both many small vector units and few large units.

We also seek to segment the creation of grid weights from the zonal execution so that the same weight map can be applied across different products with the same structure.

## Installation

You can install the development version of `zonal` from [GitHub](https://github.com/) with:

``` r
# install.packages("remotes")
remotes::install_github("mikejohnson51/zonal")
```
## Example

This is a basic example that takes a NetCDF file containing a 4km grid for the continental USA and daily precipitation for the year 1979 (365 layers). Our goal is to subset this file to the southern USA, and compute daily county level averages. The result is a daily rainfall average for each county.


```r
library(zonal)
library(dplyr)
library(tidyr)
library(ggplot2)

file <- 'to_build/pr_2020.nc'
AOI  <- AOI::aoi_get(state = "south", county = "all") 

system.time({
  # Build Weight Grid
  w        = weighting_grid(file, geom = AOI, ID = "geoid")
  # Intersect
  pr_zone = execute_zonal(file, w = w)
})
#>    user  system elapsed 
#>   7.490   1.490   9.719

# PET zone: Counties, time slices/ID
dim(pr_zone)
#> [1] 1421  367
```

### Daily maximum mean rainfall in a county?


```r
x = merge(AOI, pr_zone)

# Plot Day with the maximum single county max rainfall.
n = colnames(pr_zone)[which(pr_zone[,-1] == max(pr_zone[,-1]), arr.ind = TRUE)[2] + 1]

ggplot() + 
  geom_sf(data = x, aes(fill = get(n)), color = NA) + 
  scale_fill_viridis_c() + 
  theme_void() + 
  labs(fill = "PR (mm)")
```

<img src="man/figures/README-unnamed-chunk-2-1.png" title="plot of chunk unnamed-chunk-2" alt="plot of chunk unnamed-chunk-2" width="100%" />

### Daily maximum rainfall in the south?


```r
# Plot Day with the maximum county wide rainfall
n2 = names(which.max(colSums(dplyr::select(pr_zone, -geoid))))

terra::plot(terra::rast(file)[[103]])
```

<img src="man/figures/README-unnamed-chunk-3-1.png" title="plot of chunk unnamed-chunk-3" alt="plot of chunk unnamed-chunk-3" width="100%" />

```r

ggplot() + 
  geom_sf(data = x, aes(fill = get(n2)), color = NA) + 
  scale_fill_viridis_c() + 
  theme_void() + 
  labs(fill = "PR (mm)")
```

<img src="man/figures/README-unnamed-chunk-3-2.png" title="plot of chunk unnamed-chunk-3" alt="plot of chunk unnamed-chunk-3" width="100%" />

### Timeseries of conuty with maximum annual rainfall


```r
data = pr_zone %>% 
  slice_max(rowSums(select(., -geoid))) %>% 
  pivot_longer(-geoid, names_to = "day", values_to = "prcp") %>% 
  mutate(day = as.numeric(gsub("V","", day)))

head(data)
#> # A tibble: 6 × 3
#>   geoid   day   prcp
#>   <chr> <dbl>  <dbl>
#> 1 37175     1  0    
#> 2 37175     2 26.7  
#> 3 37175     3 13.6  
#> 4 37175     4  0.234
#> 5 37175     5  0.144
#> 6 37175     6  2.52
```

<img src="man/figures/README-unnamed-chunk-5-1.png" title="plot of chunk unnamed-chunk-5" alt="plot of chunk unnamed-chunk-5" width="100%" />

# 1km Landcover Grid (Categorical)

One of the largest limitations of existing utilities is the ability to handle categorical data. Here we show an example for a 1km grid storing land cover data from MODIS. This grid was creating by mos acing 19 MODIS tiles covering CONUS.


```r
file = 'to_build/2019-01-01.tif'
rcl  = read.csv("to_build/modis_lc.csv") %>% 
  dplyr::select(from = Class, to = short)

system.time({
  lc = execute_zonal_cat(file, AOI, "geoid", rcl = rcl)
})
#> Error in UseMethod("window<-"): no applicable method for 'window<-' applied to an object of class "SpatRaster"
```

<img src="man/figures/README-unnamed-chunk-7-1.png" title="plot of chunk unnamed-chunk-7" alt="plot of chunk unnamed-chunk-7" width="100%" /><img src="man/figures/README-unnamed-chunk-7-2.png" title="plot of chunk unnamed-chunk-7" alt="plot of chunk unnamed-chunk-7" width="100%" />

----

## Getting involved

1. Code style should attempt to follow the tidyverse style guide.
2. Please avoid adding significant new dependencies without a documented reason why.
3. Please attempt to describe what you want to do prior to contributing by submitting an issue.
4. Please follow the typical github fork - pull-request workflow.
5. Make sure you use roxygen and run Check before contributing.

----

## Open source licensing info
1. [TERMS](TERMS.md)
2. [LICENSE](LICENSE)

----

## Credits and references

Similar R packages:

1. [exactexactr](https://github.com/isciences/exactextractr)
2. [intersectr](https://github.com/USGS-R/intersectr)
3. [areal](https://github.com/slu-openGIS/areal)
4. [sf](https://github.com/r-spatial/sf)
5. [raster](https://github.com/rspatial/raster)

**Logo Artwork:** [Justin Singh-Mohudpur](https://www.justinsingh.me/about/)
