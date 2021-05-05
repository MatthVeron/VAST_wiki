VAST allows users to specify density covariates, with values drawn from `covariate_data` (which specifies values for a set of Lat and Lon locations in each Year) and formatted using input `formula`. This formula interface is similar to that used by `lm` and `glm`. Previous format required input `X_gtp` (for covariates all extrapolation-grid cells `g`) and `X_itp` (for all observations `i`). These can still be specified for backwards compatibility. By default, the formula interface generates these arrays internally.

By default, VAST assumes that every covariate has a linear effect, but users can use polynomial (or other forms of) basis expansion to represent nonlinear effects -- I show a quadratic effect below.  Users can also use input `Xconfig_zcp` to specify other covariate effects, including zero-centered or non-centered spatially varying responses to covariates, or turning off individual covariates;  these decisions are made separately for both model linear predictors. See `?make_data` for details regarding input formats.  Below is a simple example of these features.

VAST also has two options to visualize covariate responses and those are demonstrated [elsewhere in the Wiki](https://github.com/James-Thorson-NOAA/VAST/wiki/Visualize-covariate-response).

```R
# Load packages
library(VAST)
library(splines)  # Used to include basis-splines
library(effects)  # Used to visualize covariate effects

# load data set
# see `?load_example` for list of stocks with example data
# that are installed automatically with `FishStatsUtils`.
example = load_example( data_set="covariate_example" )

# Make settings (turning off bias.correct to save time for example)
settings = make_settings( n_x=100,
  Region=example$Region,
  purpose="index2",
  use_anisotropy=FALSE,
  bias.correct=FALSE,
  fine_scale=TRUE )

# Define formula.
# In this case I'm demonstrating how to use a basis-spline with
# three degrees of freedom to model a nonlinear effect of log-transformed bottom depth,
# based on example developed by Nicholas Ducharme-Barth.
X1_formula = ~ bs( log(BOT_DEPTH), degree=2, intercept=FALSE)
#X1_formula = ~ poly( log(BOT_DEPTH), degree=2 )
# I'm also showing how to construct an interaction
X2_formula = ~ poly(log(BOT_DEPTH), degree=2) + poly( BOT_TEMP, degree=2 )

# If all covariates as "static" (not changing among years),
#  then set Year = NA to cause values to be duplicated internally for all values of Year
# If using a mix of static and dynamic covariates,
#  then duplicate rows for static covariates for every value of Year
# Here, all covariates are static, so I'm using the first approach.
example$covariate_data[,'Year'] = NA

# Rescale covariates being used to have an SD >0.1 and <10 (for numerical stability)
example$covariate_data[,'BOT_DEPTH'] = example$covariate_data[,'BOT_DEPTH'] / 100

# Run model
fit = fit_model( "settings" = settings,
  Lat_i = example$sampling_data[,'Lat'],
  Lon_i = example$sampling_data[,'Lon'],
  t_i = example$sampling_data[,'Year'],
  b_i = example$sampling_data[,'Catch_KG'],
  a_i = example$sampling_data[,'AreaSwept_km2'],
  X1_formula = X1_formula,
  X2_formula = X2_formula,
  covariate_data = example$covariate_data )

# Standard plots
plot( fit,
  plot_set=c(3,11,12),
  TmbData=fit$data_list )
```