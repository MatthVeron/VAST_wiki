# Example of using VAST with high-level wrapper functions

I here demonstrate my efforts to develop high-level functions to simplify running `VAST` for several common model purposes.  The default structure shown here is for "index standardization."

```R
# Download latest release number; its useful for reproducibility to use a specific release number
devtools::install_github("James-Thorson-NOAA/VAST")

# Set local working directory (change for your machine)
setwd( "D:/UW Hideaway (SyncBackFree)/AFSC/2019-03 -- Making helper functions for VAST" )

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
  bias.correct = FALSE )

# Run model
fit = fit_model( settings = settings, 
  Lat_i = example$sampling_data[,'Lat'], 
  Lon_i = example$sampling_data[,'Lon'], 
  t_i = example$sampling_data[,'Year'], 
  b_i = example$sampling_data[,'Catch_KG'], 
  a_i = example$sampling_data[,'AreaSwept_km2'] )

# Plot results
plot( fit )
```

This should provide an abundance index for whichever species is chosen in `load_example`, which loads Alaska pollock in the eastern Bering Sea by default. To use `VAST` with a new data set, input new data in the same format as example data and see `Data_Fn` for more details.