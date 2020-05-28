# Joint model for fish condition and density
We developed a multivariate spatio-temporal modeling approach that simultaneously jointly estimates population density (measured as numbers per area) and fish condition (the relative weight of an individual fish given its body length); the model is then used to predict density-weighted average condition by summing over the product of population density, local condition, and surface area. Density-weighted average condition corrects for biases that would arise when condition (weight-at-length) samples are not distributed proportional to population densities.  Our approach treats both density and condition as “categories” in VAST, and accounts for density-dependent condition by estimating a correlation between population density and condition. 

Here, we demonstrate our approach for the Pacific cod (Gadus macrocephalus) population of the eastern Bering Sea for 1982-2016. The model developed here does not include any environmental covariates, but is simplified from Grüss et al. (2020). 

Grüss, A., Gao, J., Thorson, J.T., Rooper, C., Thompson, G., Boldt, J. and Lauth, R. (2020) Estimating synchronous changes in condition and density in Eastern Bering Sea fishes. Marine Ecology Progress Series.

One key step for the estimation of density-dependent fish condition is the definition of the `Expansion_cz` object. In our case: ` Expansion_cz = matrix( c( 0,0, 2,0 ), ncol = 2, byrow=TRUE )` which specifies that the annual index fish condition will be calculated as the weighted average of local condition, weighted by local densities. 

```R
# Load packages
library( VAST )

# load data set
# see `?load_example` for list of stocks with example data
# that are installed automatically with `FishStatsUtils`.
example = load_example( data_set = "goa_arrowtooth_condition_and_density" )

# Format data
b_i = ifelse( !is.na(example$sampling_data[,'cpue_kg_km2']),
  example$sampling_data[,'cpue_kg_km2'],
  example$sampling_data[,'weight_g'] )
c_i = ifelse( !is.na(example$sampling_data[,'cpue_kg_km2']), 0, 1 )
Q_i = ifelse(!is.na(example$sampling_data[,'cpue_kg_km2']),
  0, log(example$sampling_data[,'length_mm']/10) )

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
  Q_ik = matrix(Q_i, ncol=1),
  Expansion_cz = Expansion_cz,
  build_model = FALSE )

# Modify Map
Map = fit$tmb_list$Map
  Map$lambda2_k = factor(NA)

# Run model
fit = fit_model( settings = settings,
  Lat_i = example$sampling_data[,'latitude'],
  Lon_i = example$sampling_data[,'longitude'],
  t_i = example$sampling_data[,'year'],
  c_i = c_i,
  b_i = b_i,
  a_i = rep(1, nrow(example$sampling_data)),
  Q_ik = matrix(Q_i, ncol=1),
  Expansion_cz = Expansion_cz,
  Map = Map )

# Inspect sample size for each data type
Nsamp_tc = tapply( example$sampling_data[,'Response_variable'],
  INDEX = list( example$sampling_data[,'Year'], example$sampling_data[,'Category'] ),
  FUN=length )
fit$years_to_plot = which( rowSums( is.na( Nsamp_tc ) ) == 0 )

# Produce and plot an abundance index and a density-dependent fish condition index
Index = plot_biomass_index( DirName = paste0( getwd(),"/" ),
  TmbData = fit$data_list,
  Sdreport = fit$parameter_estimates$SD,
  Year_Set = fit$year_labels,
  Years2Include = fit$years_to_plot,
  category_names = c( "Abundance", "Fish condition" ) )
```
