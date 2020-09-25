# Joint models fitting encounter, count, and biomass-sampling data

It is possible to use `VAST` to fit models jointly to different modes (a.k.a. types) of data, including encounter/non-encounter, count, and biomass-sampling data.  Combining data from different sampling designs can be useful to expand the spatial footprint beyond that covered by any one sampling program.

As a simple example, I show code for combining three data types for red snapper in the Gulf of Mexico (I note that the wrapper functions used here are currently in the development branch of `FishStatsUtils`, and the fine-scaled interpolation in the development branch of `VAST`, but both will be available in the next numbered release):

```R
# Download release number 3.6.0; its useful for reproducibility to use a specific release number
devtools::install_github("james-thorson/VAST", ref="3.6.0")

# Load packages
library(VAST)

# load data set
# see `?load_example` for list of stocks with example data
# that are installed automatically with `FishStatsUtils`.
example = load_example( data_set="multimodal_red_snapper" )

# Make settings
settings = make_settings( n_x = 100,
  Region = example$Region,
  purpose = "index",
  strata.limits = example$strata.limits )

# Change `ObsModel` to indicate type of data for level of `e_i`
settings$ObsModel = cbind( c(13,14,2), 1 )

# Add a design matrix representing differences in catchability relative to a reference (biomass-sampling) gear
catchability_data = example$sampling_data[,'Data_type',drop=FALSE]
Q1_formula = ~ factor(Data_type)

# Run model
fit = fit_model( settings = settings,
  Lat_i = example$sampling_data[,'Lat'],
  Lon_i = example$sampling_data[,'Lon'],
  t_i = example$sampling_data[,'Year'],
  c_i = rep(0,nrow(example$sampling_data)),
  b_i = example$sampling_data[,'Response_variable'],
  a_i = example$sampling_data[,'AreaSwept_km2'],
  e_i = as.numeric(example$sampling_data[,'Data_type'])-1,
  Q1_formula = Q1_formula,
  catchability_data = catchability_data )

# Plot results
plot( fit )
```