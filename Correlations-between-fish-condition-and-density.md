# Joint model for fish condition and density
We developed a multivariate spatio-temporal modeling approach that jointly estimates population density (measured as numbers per area) and fish condition (the relative weight of an individual fish given its body length); the model is then used to predict density-weighted average condition by summing over the product of population density, local condition, and surface area. Density-weighted average condition corrects for biases that would arise when condition (weight-at-length) samples are not distributed proportional to population densities.  Our approach treats both density and condition as “categories” in VAST.  In models that estimate a covariance between these categories (i.e., using a factor-model for Omega and/or Epsilon), the correlation between the density and condition variables can be interpreted as density dependence (i.e., the causal impact of changing density on changing condition). 

Here, we demonstrate our approach for the arrowtooth flounder stock in the Gulf of Alaska. The model developed here does not include any environmental covariates, but is simplified from Grüss et al. (2020). 

Grüss, A., Gao, J., Thorson, J.T., Rooper, C., Thompson, G., Boldt, J. and Lauth, R. (2020) Estimating synchronous changes in condition and density in Eastern Bering Sea fishes. Marine Ecology Progress Series.

One key step for the estimation of density-dependent fish condition is the definition of the `Expansion_cz` object. In our case: ` Expansion_cz = matrix( c( 0,0, 2,0 ), ncol = 2, byrow=TRUE )` which specifies that the annual index fish condition will be calculated as the weighted average of local condition, weighted by local densities. 

```R
# Load packages
library( VAST )

# load data set
# see `?load_example` for list of stocks with example data
# that are installed automatically with `FishStatsUtils`.
example = load_example( data_set = "GOA_arrowtooth_condition_and_density" )

# Format data
b_i = ifelse( !is.na(example$sampling_data[,'cpue_kg_km2']),
  example$sampling_data[,'cpue_kg_km2'],
  example$sampling_data[,'weight_g'] )
c_i = ifelse( !is.na(example$sampling_data[,'cpue_kg_km2']), 0, 1 )
catchability_data = data.frame( "length_cm" = ifelse(!is.na(example$sampling_data[,'cpue_kg_km2']),
  1, example$sampling_data[,'length_mm']/10 ))

# Make settings
settings = make_settings( n_x = 250,
  Region = example$Region,
  purpose = "condition_and_density",
  bias.correct = FALSE,
  knot_method = "grid" )
settings$FieldConfig[c("Omega","Epsilon"),"Component_1"] = "IID"
Expansion_cz = matrix( c( 0,0, 2,0 ), ncol=2, byrow=TRUE )
settings$ObsModel = matrix( c(2,4, 1,4), ncol=2, byrow=TRUE )

# Run model
fit = fit_model( settings = settings,
  Lat_i = example$sampling_data[,'latitude'],
  Lon_i = example$sampling_data[,'longitude'],
  t_i = example$sampling_data[,'year'],
  c_i = c_i,
  b_i = b_i,
  a_i = rep(1, nrow(example$sampling_data)),
  catchability_data = catchability_data,
  Q2_formula= ~ log(length_cm),
  #Q2config_k = c(3), # Potential switch to make allometric weight-length a spatially varying term
  Expansion_cz = Expansion_cz )

# standard plots
plot( fit,
  Yrange=c(NA,NA),
  category_names=c("Biomass","Condition (grams per cm^power)") )
```

alternatively, it is possible to run a similar analysis just on condition (weight and length) data, as shown below.  The density maps are unlikely to differ greatly, but this could be useful for comparison with other methods.  Of course, only analyzing condition data precludes doing density-weighting when calculating an annual index of condition.

```R
# Load packages
library( VAST )

# load data set
example = load_example( data_set = "GOA_arrowtooth_condition_and_density" )

# Make settings
settings = make_settings( n_x = 250,
  Region = example$Region,
  purpose = "index",
  bias.correct = FALSE,
  knot_method = "grid",
  ObsModel = c(1,4) )

# Only use condition data
condition_data = example$sampling_data
  condition_data = subset( condition_data, is.na(cpue_kg_km2) )

# Run model
fit = fit_model( settings = settings,
  Lat_i = condition_data[,'latitude'],
  Lon_i = condition_data[,'longitude'],
  t_i = condition_data[,'year'],
  b_i = condition_data[,'weight_g'],
  a_i = rep(1, nrow(condition_data)),
  Q_ik = matrix(log(condition_data[,'length_mm']/10), ncol=1),
  build_model = FALSE )

# Modify Map
Map = fit$tmb_list$Map
  Map$lambda2_k = factor(NA)

# Run model
fit = fit_model( settings = settings,
  Lat_i = condition_data[,'latitude'],
  Lon_i = condition_data[,'longitude'],
  t_i = condition_data[,'year'],
  b_i = condition_data[,'weight_g'],
  a_i = rep(1, nrow(condition_data)),
  Q_ik = matrix(log(condition_data[,'length_mm']/10), ncol=1),
  Map = Map,
  test_fit = FALSE,
  getsd = FALSE )

# standard plots
plot( fit,
  Yrange=c(NA,NA) )
```