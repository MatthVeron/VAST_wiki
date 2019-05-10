# Expanding age/length composition data for use in stock assessments

It is possible to use `VAST` to expand subsamples of age/length composition obtained during bottom-trawl sampling.  To do so, we first perform first-stage expansion and then fit a spatio-temporal model to those expanded samples:

```R
# Download development branch (or perhaps master branch in future releases)
devtools::install_github("james-thorson/VAST")
devtools::install_github("james-thorson/FishStatsUtils")

# Load packages
library(TMB)
library(VAST)

# load data set
# see `?load_example` for list of stocks with example data
# that are installed automatically with `FishStatsUtils`.
example = load_example( data_set="Lingcod_comp_expansion" )

# Make settings
settings = make_settings( n_x=50, Region=example$Region, purpose="index",
  strata.limits=example$strata.limits )

# Change settings from defaults
settings$ObsModel = c(2,0)
settings$use_anisotropy = FALSE
settings$fine_scale = FALSE

# Run model
fit = fit_model( "settings"=settings, "Lat_i"=example$sampling_data[,'Lat'], "Lon_i"=example$sampling_data[,'Lon'],
  "t_i"=example$sampling_data[,'Year'], "c_i"=as.numeric(example$sampling_data[,"Length_bin"])-1,
  "b_i"=example$sampling_data[,'First_stage_expanded_numbers'],
  "a_i"=example$sampling_data[,'AreaSwept_km2'], model_args=list("Npool"=40),
  newtonsteps=1 )

# Plot results
results = plot_results( settings=settings, fit=fit )

# Calculate proportions
proportions = calculate_proportion( TmbData=fit$data_list, Index=results$Index, Year_Set=fit$year_labels )
```

Note: this code yields a slightly different result from those used for backwards-compatibility checks, due to differences in the k-means locations for knots between earlier reposited results and newer version of `FishStatsUtils`. Also, using `"Npool"=20` using the newer k-means results in a noninvertible Hessian, so I here use `"Npool"=40` instead.