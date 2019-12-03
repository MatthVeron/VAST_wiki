# Joint models fitting encounter, count, and biomass-sampling data

It is possible to use `VAST` to fit models jointly to different modes (a.k.a. types) of data, including encounter/non-encounter, count, and biomass-sampling data.  Combining data from different sampling designs can be useful to expand the spatial footprint beyond that covered by any one sampling program.

As a simple example, I show code for combining three data types for red snapper in the Gulf of Mexico (I note that the wrapper functions used here are currently in the development branch of `FishStatsUtils`, and the fine-scaled interpolation in the development branch of `VAST`, but both will be available in the next numbered release):

```R
# Download release number 3.0.0; its useful for reproducibility to use a specific release number
devtools::install_github("james-thorson/VAST", ref="3.0.0")

# Load packages
library(TMB)
library(VAST)

# load data set
# see `?load_example` for list of stocks with example data
# that are installed automatically with `FishStatsUtils`.
example = load_example( data_set="multimodal_red_snapper" )

# Make settings
settings = make_settings( n_x=100, Region=example$Region, purpose="index",
  strata.limits=example$strata.limits )

# Change `ObsModel` to indicate type of data for level of `e_i`
settings$ObsModel = cbind( c(13,14,2), 1 )

# Add a design matrix representing differences in catchability relative to a reference (biomass-sampling) gear
Q_ik = ThorsonUtilities::vector_to_design_matrix( example$sampling_data[,'Data_type'] )[,-3,drop=FALSE]

# Run model
input_list = fit_model( "settings"=settings, "Lat_i"=example$sampling_data[,'Lat'],
  "Lon_i"=example$sampling_data[,'Lon'], "t_i"=example$sampling_data[,'Year'],
  "c_i"=rep(0,nrow(example$sampling_data)), "b_i"=example$sampling_data[,'Response_variable'],
  "a_i"=example$sampling_data[,'AreaSwept_km2'], "e_i"=as.numeric(example$sampling_data[,'Data_type'])-1,
  "Q_ik"=Q_ik, run_model=FALSE )

# Map off unnecessary calibration coefficients for 2nd linear prediction, which isn't used for 
# encounter/non-encounter or count data sets
map0 = input_list$tmb_list$Map
map0$lambda2_k = factor(rep(NA,length(input_list$tmb_list$Parameters$lambda2_k)))

# Re-fit having changed map
fit = fit_model( "settings"=settings, "Lat_i"=example$sampling_data[,'Lat'],
  "Lon_i"=example$sampling_data[,'Lon'], "t_i"=example$sampling_data[,'Year'],
  "c_i"=rep(0,nrow(example$sampling_data)), "b_i"=example$sampling_data[,'Response_variable'],
  "a_i"=example$sampling_data[,'AreaSwept_km2'], "e_i"=as.numeric(example$sampling_data[,'Data_type'])-1,
  "Q_ik"=Q_ik, model_args=list("Map"=map0), optimize_args=list("lower"=-Inf, "upper"=Inf) )

# Plot results
plot_results( settings=settings, fit=fit )
```