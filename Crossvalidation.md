### How to conduct k-fold cross-validation

Analysts sometimes want to compare different models via their predictive performance.  In this case, its helpful to fit the model to a partition of the data ("in-bag") and predict data that was held out ("out-of-bag").  I show how to do that below, using a simple-random design for a 10-fold crossvalidation experiment. Stratified designs for excluding data (i.e., predicting years sequentially) are also worth exploring, and require changing the process to generate `Partition_i`

```R
# Set local working directory (change for your machine)
setwd( "C:/Users/James.Thorson/Desktop/Work files/AFSC/2019-09 -- Crossvalidation example" )

# Load packages
library(TMB)
library(VAST)

# load data set
# see `?load_example` for list of stocks with example data
# that are installed automatically with `FishStatsUtils`.
example = load_example( data_set="EBS_pollock" )

# Make settings (turning off bias.correct to save time for example)
settings = make_settings( n_x=100, Region=example$Region, purpose="index",
  strata.limits=example$strata.limits, bias.correct=FALSE )

# Fit the model and a first time and record MLE
fit = fit_model( "settings"=settings, "Lat_i"=example$sampling_data[,'Lat'],
  "Lon_i"=example$sampling_data[,'Lon'], "t_i"=example$sampling_data[,'Year'],
  "c_i"=rep(0,nrow(example$sampling_data)), "b_i"=example$sampling_data[,'Catch_KG'],
  "a_i"=example$sampling_data[,'AreaSwept_km2'], "v_i"=example$sampling_data[,'Vessel'] )
ParHat = fit$ParHat

# Generate partitions in data
n_fold = 10
Partition_i = sample( 1:n_fold, size=nrow(example$sampling_data), replace=TRUE )
prednll_f = rep(NA, n_fold )

# Loop through partitions, refitting each time with a different PredTF_i
for( fI %in% 1:n_fold ){
  PredTF_i = ifelse( Partition_i==fI, TRUE, FALSE )

  # Refit, starting at MLE, without calculating standard errors (to save time)
  fit_new = fit_model( "settings"=settings, "Lat_i"=example$sampling_data[,'Lat'],
    "Lon_i"=example$sampling_data[,'Lon'], "t_i"=example$sampling_data[,'Year'],
    "c_i"=rep(0,nrow(example$sampling_data)), "b_i"=example$sampling_data[,'Catch_KG'],
    "a_i"=example$sampling_data[,'AreaSwept_km2'], "v_i"=example$sampling_data[,'Vessel'],
    "PredTF_i"=PredTF_i, "Parameters"=ParHat, "getsd"=FALSE )

  # Save fit to out-of-bag data
  prednll_f[fI] = fit_new$Report$pred_jnll
}

# Check fit to all out=of-bag data and use as metric of out-of-bag performance
sum( prednll_f )
```