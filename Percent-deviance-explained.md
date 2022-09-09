It is possible to calculate proportion-deviance-explained using VAST (currently implemented for gamma and lognormal distributions only, using either the conventional or Poisson-linked delta models).  This can then be used to measure model explanatory power and compare this across software packages.  Note that the calculation treats empirical Bayes predictions of random effects as fixed, and therefore does not propagate random-effect predictive variance (this behavior is conceptually similar to treatment in mgcv and elsewhere).

The calculation involves comparing `fit$Report$deviance` for a target model with the same calculation using a null model including only intercepts.  See below for an example (which requires the VAST release > 3.7.1, or the dev branch until that release):

```R
# Load package
library(VAST)

# load data set
example = load_example( data_set="EBS_pollock" )

###### Target model
run_dir = paste0(getwd(),"/pollock/target/")
  dir.create(run_dir,recursive=TRUE)

# Make settings (turning off bias.correct to save time for example)
settings1 = make_settings( n_x = 100,
  Region = example$Region,
  purpose = "index2",
  strata.limits = example$strata.limits,
  bias.correct = FALSE )
settings1$Options = c( settings1$Options, "report_additional_variables"=TRUE )

# Run model
fit1 = fit_model( settings = settings1,
  Lat_i = example$sampling_data[,'Lat'],
  Lon_i = example$sampling_data[,'Lon'],
  t_i = example$sampling_data[,'Year'],
  b_i = example$sampling_data[,'Catch_KG'],
  a_i = example$sampling_data[,'AreaSwept_km2'],
  working_dir = run_dir )

###### Null model
run_dir = paste0(getwd(),"/pollock/null/")
  dir.create(run_dir,recursive=TRUE)

# Make settings (turning off bias.correct to save time for example)
settings0 <- settings1
settings0$FieldConfig = matrix( c(0,0,0,0,"IID","IID"), byrow=TRUE, ncol=2 )
settings0$RhoConfig[c("Beta1","Beta2")] = 3

# Run model
fit0 = fit_model( settings = settings0,
  Lat_i = example$sampling_data[,'Lat'],
  Lon_i = example$sampling_data[,'Lon'],
  t_i = example$sampling_data[,'Year'],
  b_i = example$sampling_data[,'Catch_KG'],
  a_i = example$sampling_data[,'AreaSwept_km2'],
  working_dir = run_dir )

###### Calculate percent-deviance-explained
1 - fit1$Report$deviance/fit0$Report$deviance
``` 