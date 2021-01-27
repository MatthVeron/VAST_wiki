## Surplus production models for data-poor species

`VAST` includes features to track fishing mortality `F_ct` for each category and year.  A 1st order autoregressive process is equivalent to Gompertz density dependence, and it is then possible to track the net effect of past and present fishing mortality on current biomass, as well as calculate biomass and fishing mortality reference points (the latter using "spawning potential ratio" F_%SPR metrics).  

We show this process in an example below for Gulf of Alaska pollock.  We note that initial fits yielded an implausible value for autocorrelation (and resulting fishing mortality targets).  We have therefore taken the F_40% = 0.281 from the [2019 stock assessment](https://apps-afsc.fisheries.noaa.gov/refm/docs/2019/GOApollock.pdf) and back-calculated the resulting magnitude of first-order autocorrelation, Rho = 0.69.  We then fixed this value during model fits.  

This approach could be substantially extended in future applications.  For example:
* The current approach inputs a fishing mortality `F_ct`, e.g., from a time-series of fishing effort and assumed catchability coefficient within an area-swept estimator. Future developments could instead fit to annual catch data.
* The software implementation does not allow autocorrelation in both intercepts `beta` and spatio-temporal variation `epsilon`, but this could easily be changed.

For more details see e.g.
* An extended [version with movement](https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/1365-2664.12664)
* An extended [multi-species version]( https://doi.org/10.1111/faf.12398)

```R
library(VAST)
example = load_example( "GOA_MICE_example" )

# Get settings
settings = make_settings( n_x=50,
  purpose="MICE",
  Region=example$Region,
  n_categories = 1,
  max_cells = 5000 )

# Modify defaults
settings$VamConfig['Rank'] = 0  # Reduce to single axis of ratio-dependent interactions

# Subset MICE-in-space example data
sampling_data = subset(example$sampling_data, spp=="Gadus_chalcogrammus")
sampling_data$spp = droplevels(sampling_data$spp)
F_ct = example$F_ct['POLLOCKWCWYK2016',,drop=FALSE]

# Build initial model
  # May take many hours, even given informative starting value
Inputs = fit_model( settings = settings,
  Lat_i = sampling_data[,'Lat'],
  Lon_i = sampling_data[,'Lon'],
  b_i = sampling_data[,'Catch_KG'],
  a_i = sampling_data[,'AreaSwept_km2'],
  v_i = as.numeric(sampling_data[,'Vessel']),
  t_i = sampling_data[,'Year'],
  F_ct = F_ct,
  build_model = FALSE )

# Map off 1st order autocorrelation
# F_40 = 0.281
Map = Inputs$tmb_list$Map
  Map$Epsilon_rho1_f = factor(NA)
Params = Inputs$tmb_list$Parameters
  Params$Epsilon_rho1_f = 1 - (0.281 / (-1 * log(0.4)))

# Run initial model
  # May take many hours, even given informative starting value
Fit = fit_model( settings = settings,
  Lat_i = sampling_data[,'Lat'],
  Lon_i = sampling_data[,'Lon'],
  b_i = sampling_data[,'Catch_KG'],
  a_i = sampling_data[,'AreaSwept_km2'],
  v_i = as.numeric(sampling_data[,'Vessel']),
  t_i = sampling_data[,'Year'],
  F_ct = F_ct,
  Map = Map,
  Parameters = Params,
  getsd = TRUE )

# Fix issue in auto-generated plot labels
Fit$year_labels = c( 1975, Fit$year_labels )
Fit$years_to_plot = c(1, Fit$years_to_plot+1)

# Plots
plot_results( fit = Fit,
  settings = settings,
  projargs = '+proj=natearth +lon_0=-170 +units=km',
  check_residuals = FALSE,
  country = "united states of america" )
```