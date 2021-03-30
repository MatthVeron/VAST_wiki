## Rationale: Changing axes
(contributed by Alexa Hermann)

By default, VAST uses a mesh of knots that is projected using the UTM coordinate system -- that is, in northings and eastings. 

Sometimes users might want to estimate ranges in reference to different axes of measurement. For example, it might be of interest to estimate the position of the range centroid or range edge along a coastline [Fredston *et al.* 2021](https://doi.org/10.1111/gcb.15614). This can be done in VAST by modifying the object `Z_gm`, the matrix of knot locations that is by default measured in northings and eastings. 

Begin with the [VAST simple example](https://github.com/James-Thorson-NOAA/VAST/wiki/Index-standardization):

```{r simple example}
# Load package
library(VAST)

# load data set
# see `?load_example` for list of stocks with example data 
# that are installed automatically with `FishStatsUtils`. 
example = load_example( data_set="EBS_pollock" )

# Make settings (turning off bias.correct to save time for example)
settings = make_settings( n_x = 100, 
                          Region = example$Region, 
                          purpose = "index2", 
                          strata.limits = example$strata.limits, 
                          bias.correct = FALSE )

# Run model: this deviates from the simple example, because we don't actually need
# the model outputs until it's been fitted with the custom axis, and this is much faster
fit = fit_model( settings = settings,
                 Lat_i = example$sampling_data[,'Lat'], 
                 Lon_i = example$sampling_data[,'Lon'], 
                 t_i = example$sampling_data[,'Year'], 
                 c_i = rep(0,nrow(example$sampling_data)), 
                 b_i = example$sampling_data[,'Catch_KG'], 
                 a_i = example$sampling_data[,'AreaSwept_km2'], 
                 v_i = example$sampling_data[,'Vessel'],
                 run_model = FALSE )
```

The Eastern Bering Sea has a complex coastline, some islands, and a shelf that runs roughly NW-SE. As a result, measuring range shifts in northings and eastings could obscure whether marine species are shifting in the direction we really expect them to move in -- "up" or "down" the shelf. In this example, we'll add a column to the `Z_gm` matrix in VAST to estimate range shifts along the Middle Domain, a continental shelf zone in the EBS (see Figure 1b in [Fredston *et al.* 2021](https://doi.org/10.1111/gcb.15614)).

The process to customize `Z_gm` is as follows:

1. Run VAST once to produce the default coordinates.
1. Pair those coordinates with the nearest point along the novel axis of measurement. Here, we'll do this by converting the UTM coordinates to lat/lon and merging them with another spatial object that has columns lat, lon, and distance along custom axis. Alternatively, you could convert your custom spatial object to UTM first. 
1. Pass this custom `Z_gm` object to `fit_model()`. 

## Create your custom axis

I'm doing this below by drawing a straight line between two points chosen to represent the end points of the Middle Domain. Note that this doesn't need to be a line; it can be any linear spatial object, e.g., [a coastline](https://github.com/afredston/range-edge-niches/blob/master/scripts/get_axes_of_measurement.R#L33). 

```R
library(tidyverse)
library(sf)

# set line endpoints 
ebs.lon <- c(-176.5,-161)
ebs.lat <- c(62,56)

# draw line
ebs.line <- data.frame(ebs.lon, ebs.lat) %>%
  rename('x'=ebs.lon,'y'=ebs.lat) %>%
  st_as_sf(coords=c('x','y')) %>%
  summarise() %>%
  st_cast("LINESTRING") %>% 
  smoothr::densify(n=99) %>% # choose number of line segments: here it's 99 for 100 points
  st_cast("MULTIPOINT") 

ebs.points <- data.frame(st_coordinates(ebs.line)) %>%
  rename("x"=X, "y"=Y)

ebs.dists <- raster::pointDistance(ebs.points[-nrow(ebs.points), c("x", "y")], 
  ebs.points[-1, c("x", "y")], 
  lonlat=TRUE)

axisdistdat <- data.frame(ebs.points[, c('x','y')], seglength=c(0, ebs.dists))
axisdistdat$lengthfromhere <- rev(cumsum(rev(axisdistdat[,"seglength"])))
```

## Add custom axis to VAST coordinates

Here we're extracting the matrix of VAST knots, measured in northings and eastings, and pairing each one with the nearest point along the custom axis (based on minimizing Euclidean distance). To prepare it for use in VAST, we just need to bind the vector of axis distances to the VAST coordinates. In other words, we start with two columns -- northings `N_km` and eastings `E_km` -- and end with three, `N_km`, `E_km`, and distance along the custom axis (here, `line_km`). 

```R
# extract northings and eastings 
Z_gm = fit$spatial_list$loc_g 

# create UTM object with correct attributes 
tmpUTM = cbind('PID'=1,'POS'=1:nrow(Z_gm),'X'=Z_gm[,'E_km'],'Y'=Z_gm[,'N_km'])
attr(tmpUTM,"projection") = "UTM"
attr(tmpUTM,"zone") = 2 

# convert to lat/lon for merging with custom axis
latlon_g = PBSmapping::convUL(tmpUTM)                                                         
latlon_g = cbind( 'Lat'=latlon_g[,"Y"], 'Lon'=latlon_g[,"X"])
latlon_g <- as.data.frame(latlon_g) # should be a df with lat/lon coordinates 

# function to save the custom axis position that minimize Euclidean distance from each set of VAST coordinates
get_length <- function(lon, lat, distdf) {
  tmp <- distdf 
  tmp$abs.diff.x2 = abs(distdf$x-lon)^2
  tmp$abs.diff.y2 = abs(distdf$y-lat)^2
  # get Euclidean dist from VAST points to all points along the coastal axis  
  tmp$abs.diff.xy = sqrt(tmp$abs.diff.x2 + tmp$abs.diff.y2) 
  tmp <- tmp[tmp$abs.diff.xy==min(tmp$abs.diff.xy),] # keep only the row with the minimum distance
  return(tmp$lengthfromhere) # save the position of that row
}

# apply this to match each VAST knot with a position along the custom axis
line_km=NULL
for(k in 1:nrow(latlon_g)){
  out = get_length(lon=latlon_g$Lon[k], lat=latlon_g$Lat[k], distdf = axisdistdat)/1000 
  line_km <- c(line_km, out)
}

# bind the axis positions that match each knot back to Z_gm
Z_gm = cbind( Z_gm, "line_km"=line_km )
Z_gm_axes = colnames(Z_gm)

# Plot to confirm
plot( x=Z_gm[,1], y=Z_gm[,2], col=viridisLite::viridis(25)[cut(Z_gm[,3],25)] )
```

## Fitting the model to the custom axis 

To put this all together, all we need to do is pass our new, customized `Z_gm` to VAST.

```R
fit = fit_model( settings = settings, 
                 Lat_i = example$sampling_data[,'Lat'], 
                 Lon_i = example$sampling_data[,'Lon'], 
                 t_i = example$sampling_data[,'Year'], 
                 c_i = rep(0,nrow(example$sampling_data)), 
                 b_i = example$sampling_data[,'Catch_KG'], 
                 a_i = example$sampling_data[,'AreaSwept_km2'], 
                 v_i = example$sampling_data[,'Vessel'],
                 Z_gm = Z_gm)

# outputs now reflect the additional custom axis: 
# note new plot RangeEdge_line_km.png, and new panel in center_of_gravity.png
plot(fit) 
```
