VAST allows users to specify density covariates, and current format requires input `X_gtp` (for covariates all extrapolation-grid cells `g`) and `X_itp` (for all observations `i`). By default, VAST assumes that every covariate has a linear effect, but users can use polynomial (or other forms of) basis expansion to represent nonlinear effects -- I show a quadratic effect below.  Users can also use input `Xconfig_zcp` to specify other covariate effects, including zero-centered or non-centered spatially varying responses to covariates, or turning off individual covariates;  these decisions are made separately for both model linear predictors. See `?make_data` for details regarding input formats.  Below is a simple example of these features

```R
# Download release number 3.0.0; its useful for reproducibility to use a specific release number
devtools::install_github("james-thorson/FishStatsUtils", ref="development")

# Set local working directory (change for your machine)
setwd( "C:/Users/James.Thorson/Desktop/Work files/AFSC/2019-07 -- Density covariate vignette" )

# Load packages
library(TMB)
library(VAST)
library(abind)

# load data set
# see `?load_example` for list of stocks with example data
# that are installed automatically with `FishStatsUtils`.
example = load_example( data_set="covariate_example" )

# Make settings (turning off bias.correct to save time for example)
settings = make_settings( n_x=100, Region=example$Region, purpose="index",
  strata.limits=example$strata.limits, bias.correct=FALSE, fine_scale=FALSE )

# Using fine_scale=FALSE, so X_gtp is defined at knots x
X_gtp = example$X_xtp

##### Format covariates at locations
# Extract covariate measurements at samples
X_ip = as.matrix(example$sampling_data[, dimnames(X_gtp)[[3]] ])
# Expand to expected size of input
X_itp = aperm( outer(X_ip, rep(1,dim(X_gtp)[2])), perm=c(1,3,2) )

# Standardize for numerical stability using same conversion for both
for( p in 1:dim(X_itp)[3] ){
  X_itp[,,p] = (X_itp[,,p] - mean(X_gtp[,,p])) / sd(X_gtp[,,p])
  X_gtp[,,p] = (X_gtp[,,p] - mean(X_gtp[,,p])) / sd(X_gtp[,,p])
}

# Add quadratic temperature for demonstration purposes
X_gtp = abind( X_gtp, "BtmTemp-squared"=X_gtp[,,'BtmTemp',drop=FALSE]^2, along=3 )
X_itp = abind( X_itp, "BtmTemp-squared"=X_itp[,,'BtmTemp',drop=FALSE]^2, along=3 )

# Renumber years to have same indexing as covariates
t_i = match( example$sampling_data[,'Year'], unique(example$sampling_data[,'Year']) )

#### Make Xconfig_zcp, using arbitrary decisions for illustration
# By default, linear effect for each variable
Xconfig_zcp = array( 1, dim=c(2,1,dim(X_gtp)[3]) )
# Turn off for all covariate effects on 1st linear predictor
Xconfig_zcp[1,,] = 0
# Use spatially varying effect for "Strat" for 2nd predictor
Xconfig_zcp[2,1,2] = 3

# Run model
fit = fit_model( "settings"=settings, "Lat_i"=example$sampling_data[,'Lat'],
  "Lon_i"=example$sampling_data[,'Lon'], "t_i"=t_i,
  "c_i"=rep(0,nrow(example$sampling_data)), "b_i"=example$sampling_data[,'Catch_KG'],
  "a_i"=example$sampling_data[,'AreaSwept_km2'], "v_i"=example$sampling_data[,'Vessel'],
  "X_gtp"=X_gtp, "X_itp"=X_itp, "Xconfig_zcp"=Xconfig_zcp, optimize_args=list("getsd"=FALSE),
  "test_fit"=FALSE )

# Plot esults
plot( fit, plot_set=c(3,13,14) )
```