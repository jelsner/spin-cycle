Animate Tornado Genesis Locations
================
James Elsner
2018-07-24

Based on: <https://gist.github.com/thomasp85/9362bbfae956f2690794abeb2c11cdcc>

Get packages.

``` r
#devtools::install_github('thomasp85/gganimate')
library(ggplot2)
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(sf)
```

    ## Linking to GEOS 3.6.1, GDAL 2.1.3, proj.4 4.9.3

``` r
library(gganimate)
library(USAboundaries)
```

Simplest. No map projection. Tornado data from <https://www.spc.noaa.gov/wcm/#data>. Get state borders. Read and filter the tornado data frame. Add a unique ID column.

``` r
sts <- state.name[!state.name %in% c("Alaska", "Hawaii")]
stateBorders <- us_states(states = sts)

Tor.df <- read.csv(file = "1950-2017_actual_tornadoes.csv") %>%
  filter(yr >= 1994, 
         mag != -9,
         !st %in% c('AK', 'PR', 'HI')) %>%
  mutate(ID = seq.int(length(yr)))

ggplot() +
  geom_sf(data = stateBorders) +
  geom_point(data = Tor.df, 
             aes(x = slon, y = slat, group = ID)) +
  labs(title = 'Year: {frame_time}') +
  xlab("") + ylab("") +
  theme_minimal() +
  transition_time(yr) +
  enter_fade() +
  exit_fade() 
```

![](SpinCycle_files/figure-markdown_github/unnamed-chunk-2-1.gif)

With map projection. Color dots by EF rating.

``` r
stateBordersP <- st_transform(stateBorders, crs = st_crs(102003))

Tor.sfdf <- st_as_sf(Tor.df, coords = c("slon", "slat"), 
                 crs = 4326)
Tor.sfdfP <- st_transform(Tor.sfdf, crs = st_crs(102003))
Tor.df <- cbind(Tor.df, st_coordinates(Tor.sfdfP))

library(viridis)
```

    ## Loading required package: viridisLite

``` r
ggplot() +
  geom_sf(data = stateBordersP) +
  geom_point(data = Tor.df, aes(x = X, y = Y, 
                                group = ID, color = factor(mag))) +
  scale_color_viridis_d(name = "EF Rating", direction = -1) +
  labs(title = 'Year: {closest_state}') +
  ylab(NULL) + xlab(NULL) +
  theme_minimal() +
  transition_states(yr, 
                    transition_length = .5, 
                    state_length = 1) +
  enter_fade() +
  exit_fade()
```

![](SpinCycle_files/figure-markdown_github/unnamed-chunk-3-1.gif)

2017 by month.

``` r
Tor.df2 <- Tor.df %>%
  filter(yr == 2017)

ggplot() +
  geom_sf(data = stateBordersP, color = "white", fill = "grey95", inherit.aes = FALSE) +
  geom_point(data = Tor.df2, aes(x = X, y = Y, color = factor(mag))) +
  scale_color_viridis_d(name = "Maximum\nEFRating", direction = -1) +
  labs(title = 'Year: 2017, Month: {closest_state}') +
  ylab(NULL) + xlab(NULL) +
  theme_minimal() +
  transition_states(mo, 
                    transition_length = .1, 
                    state_length = 1) +
  enter_fade() +
  exit_fade()
```

![](SpinCycle_files/figure-markdown_github/unnamed-chunk-4-1.gif)

Use `anim_save()` to save rendered animation. Saves the last animation to a file.

``` r
anim_save(file = "LastYear.gif")
```

2017 by day

``` r
Tor.df2 <- Tor.df %>%
  filter(yr == 2017) %>%
  mutate(dy = format(as.Date(date,format="%m/%d/%y"), "%d"),
         Date = as.POSIXct(paste(yr, mo, dy), format = "%Y%m%d"))

ggplot() +
  geom_sf(data = stateBordersP, color = "white", fill = "grey95", inherit.aes = FALSE) +
  geom_point(data = Tor.df2, aes(x = X, y = Y, color = factor(mag))) +
  scale_color_viridis_d(name = "Maximum\nEFRating", direction = -1) +
  labs(title = 'Date: {frame_time}') +
  ylab(NULL) + xlab(NULL) +
  theme_minimal() +
  transition_time(Date) 
```

![](SpinCycle_files/figure-markdown_github/unnamed-chunk-6-1.gif)
