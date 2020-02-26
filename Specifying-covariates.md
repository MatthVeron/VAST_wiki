VAST allows users to specify density covariates, with values drawn from `covariate_data` (which specifies values for a set of Lat and Lon locations in each Year) and formatted using input `formula`. This formula interface is similar to that used by `lm` and `glm`. Previous format required input `X_gtp` (for covariates all extrapolation-grid cells `g`) and `X_itp` (for all observations `i`). These can still be specified for backwards compatibility. By default, the formula interface generates these arrays internally.

By default, VAST assumes that every covariate has a linear effect, but users can use polynomial (or other forms of) basis expansion to represent nonlinear effects -- I show a quadratic effect below.  Users can also use input `Xconfig_zcp` to specify other covariate effects, including zero-centered or non-centered spatially varying responses to covariates, or turning off individual covariates;  these decisions are made separately for both model linear predictors. See `?make_data` for details regarding input formats.  Below is a simple example of these features

```R

# Install development branch for now to access example data in right format
devtools::install_github( "james-thorson/FishStatsUtils" )

# Set local working directory (change for your machine)
setwd( "C:/Users/James.Thorson/Desktop/Work files/AFSC/2019-09 -- Formula interface" )

# Load packages
library(TMB)
library(VAST)

# load data set
# see `?load_example` for list of stocks with example data
# that are installed automatically with `FishStatsUtils`.
example = load_example( data_set="covariate_example" )

# Make settings (turning off bias.correct to save time for example)
settings = make_settings( n_x=100, Region=example$Region, purpose="index", use_anisotropy=FALSE,
  strata.limits=example$strata.limits, bias.correct=FALSE, fine_scale=TRUE ) #, ObsModel=c(1,0) )

# Define formula
formula = ~ BOT_DEPTH + I(BOT_DEPTH^2)

# set Year = NA to treat all covariates as "static" (not changing among years)
# If using a mix of static and dynamic covariates, please email package author to add easy capability
example$covariate_data[,'Year'] = NA

# Rescale covariates being used to have an SD >0.1 and <10 (for numerical stability)
example$covariate_data[,'BOT_DEPTH'] = example$covariate_data[,'BOT_DEPTH'] / 100

# Run model
fit = fit_model( "settings"=settings, "Lat_i"=example$sampling_data[,'Lat'],
  "Lon_i"=example$sampling_data[,'Lon'], "t_i"=example$sampling_data[,'Year'],
  "b_i"=example$sampling_data[,'Catch_KG'], "a_i"=example$sampling_data[,'AreaSwept_km2'],
  "formula"=formula, "covariate_data"=example$covariate_data )

#
plot( fit, plot_set=c(3,11,13,14), TmbData=fit$data_list )
```