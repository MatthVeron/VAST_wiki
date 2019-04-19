# Example of using VAST with high-level wrapper functions

I here demonstrate my efforts to develop high-level functions to simplify running `VAST` for several common model purposes.  At time of writing, this requires installing the development branch of `FishStatsUtils`, although future releases will include this code in the master branch and I may forget to update this page.

```R
# Download development branch (or perhaps master branch in future releases)
devtools::install_github("james-thorson/VAST", ref="development")
devtools::install_github("james-thorson/FishStatsUtils", ref="development")

# Set local working directory (change for your machine)
setwd( "D:/UW Hideaway (SyncBackFree)/AFSC/2019-03 -- Making helper functions for VAST" )

# Load packages
library(TMB)               
library(VAST)

# load data set
# see `?load_example` for list of stocks with example data 
# that are installed automatically with `FishStatsUtils`. 
example = load_example( data_set="EBS_pollock" )

# Make settings
settings = make_settings( n_x=50, Region=example$Region, purpose="index", 
  strata.limits=example$strata.limits )

# Run model
fit = fit_model( "settings"=settings, "Lat_i"=example$sampling_data[,'Lat'], 
  "Lon_i"=example$sampling_data[,'Lon'], "t_i"=example$sampling_data[,'Year'], 
  "c_i"=rep(0,nrow(example$sampling_data)), "b_i"=example$sampling_data[,'Catch_KG'], 
  "a_i"=example$sampling_data[,'AreaSwept_km2'], "v_i"=example$sampling_data[,'Vessel'] )

# Plot results
plot_results( settings=settings, fit=fit )
```

This should provide an abundance index for whichever species is chosen in `load_example`, which loads Alaska pollock in the eastern Bering Sea by default. To use `VAST` with a new data set, input new data in the same format as example data and see `Data_Fn` for more details.