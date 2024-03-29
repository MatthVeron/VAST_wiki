VAST simultaneously fits a spatio-temporal regression model, and then predicts responses across a specified spatial and temporal domain.  This combines two steps that are typically done separately in other regression packages within R.  The advantages are:
1. Avoid user errors, e.g., when doing area-expansion of model results;
2. Require explicit consideration of whether covariates affect density (used for both fitting and prediction) or catchability (used just for fitting);
3. Allow for bias-correction and asymptotic standard errors in model predictions and derived quantities;

However, combining these two tasks requires the user to pre-specify the spatial domain for prediction (the "extrapolation-grid").  We here show how users can make a new extrapolation-grid, and also refer users to `?make_extrapolation_info`.

### Warning
Note that the `locator` function used below only works properly in Rstudio when screen setting "zoom" is set to 100% and see details [here](https://stackoverflow.com/questions/73256552/problem-with-point-location-with-locator-function-in-r).  We recommend carefully checking the extrapolation-grid constructed against the intended area (thanks to Ellen Yasumiishi for identifying this issue and finding the explanation)

### Code demo
```R
### An example of how to create user-defined extrapolation
### regions (extents) for VAST.

### Cecilia O'Leary and Cole Monnahan | December 2020

### The extrapolation region defines the extent over which the
### model predictions are integrated. Any density outside of this
### region will not be included in estimates of the index. It is
### not used in model fitting. It comes with many built-in
### regions but often a user needs to define their own. Here, we
### demonstrate two ways of doing this: (1) From a set of points
### representing the outer extent of the region; (2) from an
### existing shape file.

library(sp) # 1.4.4
library(sf) # 0.9.6

### Method 1: use a set of lat/lon coordinates which define the
### outer edge of the region. For instance you might want to plot
### your data and simply create a region that captures it. The
### locator() function can be useful for this as shown
### below. Here we use a subset of the Eastern Bering Sea.

### Use this to draw points around your data
## plot(dat$lon, dat$lat)
## LL <- locator()
## saveRDS(LL, 'extent_LL.rds')

## Take a data.frame of coordinates in longitude/latitude that
## define the outer limits of the region (the extent).
LL <- readRDS('extent_LL.rds')
region_extent <- data.frame(long=LL$x, lat=LL$y)
str(region_extent)
## > 'data.frame':	42 obs. of  2 variables:
## $ long: num  -166 -166 -165 -165 -164 ...
## $ lat : num  53.9 54.1 54.2 54.6 55 ...

#### Turn it into a spatial polygon object
## Need to duplicate a point so that it is connected
region_extent <- rbind(region_extent, region_extent[1,])
## https://www.maths.lancs.ac.uk/~rowlings/Teaching/Sheffield2013/cheatsheet.html
poly <- Polygon(region_extent)
polys <- Polygons(list(poly), ID='all')
sps <- SpatialPolygons(list(polys))
## I think the F_AREA could be dropped here
sps <- SpatialPolygonsDataFrame(sps, data.frame(Id=factor('all'), F_AREA=1, row.names='all'))
proj4string(sps)<- CRS("+proj=longlat +datum=WGS84")
sps <- spTransform(sps, CRS("+proj=longlat +lat_0=90 +lon_0=180 +x_0=0 +y_0=0 +datum=WGS84 +units=m +no_defs +ellps=WGS84 +towgs84=0,0,0 "))
### Get UTM zone for conversion to UTM projection
## retrieves spatial bounding box from spatial data [,1] is
## longitude
lon <- sum(bbox(sps)[1,])/2
## convert decimal degrees to utm zone for average longitude, use
## for new CRS
utmzone <- floor((lon + 180)/6)+1
crs_LL <- CRS('+proj=longlat +ellps=WGS84 +no_defs')
sps@proj4string <- crs_LL
### End method 1
### --------------------------------------------------


### --------------------------------------------------
### Method 2: Get it from an existing shapefile.
library(rgdal) # '1.5.18'
shp <- readOGR("nbs_strata.shp", layer="nbs_strata")
sps <- spTransform(shp, CRS("+proj=longlat +lat_0=90 +lon_0=180 +x_0=0 +y_0=0 +datum=WGS84 +units=m +no_defs +ellps=WGS84 +towgs84=0,0,0 "))
lon <- sum(bbox(sps)[1,])/2
## convert decimal degrees to utm zone for average longitude, use
## for new CRS
utmzone <- floor((lon + 180)/6)+1
### End method 2
### --------------------------------------------------


### --------------------------------------------------
### Create the VAST extroplation grid for method 1 and 2
## Convert the final in polygon to UTM
crs_UTM <- CRS(paste0("+proj=utm +zone=",utmzone," +ellps=WGS84 +datum=WGS84 +units=m +no_defs "))
region_polygon <- spTransform(sps, crs_UTM)

### Construct the extroplation grid for VAST using sf package
## Size of grid **in meters** (since working in UTM). Controls
## the resolution of the grid.
cell_size <- 2000
## This step is slow at high resolutions
region_grid <- st_make_grid(region_polygon, cellsize = cell_size, what = "centers")
## Convert region_grid to Spatial Points to SpatialPointsDataFrame
region_grid <- as(region_grid, "Spatial")
region_grid_sp <- as(region_grid, "SpatialPointsDataFrame")
## combine shapefile data (region_polygon) with Spatial Points
## (region_grid_spatial) & place in SpatialPointsDataFrame data
## (this provides you with your strata identifier (here called
## Id) in your data frame))
region_grid_sp@data <- over(region_grid, region_polygon)

## Convert back to lon/lat coordinates as that is what VAST uses
region_grid_LL <- as.data.frame(spTransform(region_grid_sp, crs_LL))
region_df <- with(region_grid_LL,
                  data.frame(Lon=coords.x1,
                             Lat=coords.x2, Id,
                             Area_km2=( (cell_size/1000)^2),
                             row=1:nrow(region_grid_LL)))
## Filter out the grid that does not overlap (outside extent)
region <- subset(region_df, !is.na(Id))
## This is the final file needed.
str(region)
## > 'data.frame':	106654 obs. of  5 variables:
##  $ Lon     : num  -166 -166 -166 -166 -166 ...
##  $ Lat     : num  53.9 53.9 54 53.9 53.9 ...
##  $ Id      : Factor w/ 1 level "all": 1 1 1 1 1 1 1 1 1 1 ...
##  $ Area_km2: num  4 4 4 4 4 4 4 4 4 4 ...
##  $ row     : int  401 402 975 976 977 978 1549 1550 1551 1552 ...

### Save it to be read in and passed to VAST later.
saveRDS(region, file = "user_region.rds")
### End of creating user extrapolation region object
### --------------------------------------------------

### Quick plots of the process for method 1
png('user_region.png', width=7, height=7, units='in', res=200)
par(mfrow=c(2,2))
with(region_extent, plot(long, lat, main='Extent in points in LL'))
plot(region_polygon, main='Polygon in UTM', axes=TRUE)
plot(region_grid, col=ifelse(is.na(region_df$Id), 'red', 'black'),
     axes=TRUE, main='Extrapolation area UTM')
with(region, plot(Lon, Lat, main='Extrapolation region in LL', pch='.'))
dev.off()


### Show how to run it in VAST
library(VAST)
dat <- load_example(data_set='EBS_pollock')$sampling_data
dat <- subset(dat, Year==2000)

settings <- make_settings(n_x=200, Region='User',
                          purpose="index2", bias.correct=FALSE,
                          knot_method='grid')
settings$FieldConfig[2,] <- 0 ## turn off temporal components
user_region <- readRDS('user_region.rds')
fit <- fit_model(settings=settings,
                 Lat_i=dat$Lat, Lon_i=dat$Lon,
                 t_i=dat$Year, b_i=dat$Catch_KG,
                 a_i=dat$AreaSwept_km2,
                 input_grid=user_region)
plot_results(fit, plot_set=3)
```

