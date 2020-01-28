# Joint model for fish condition and density
We developed a multivariate spatio-temporal modeling approach that simultaneously jointly estimates population density (measured as numbers per area) and fish condition (the relative weight of an individual fish given its body length); the model is then used to predict density-weighted average condition by summing over the product of population density, local condition, and surface area. Density-weighted average condition corrects for biases that would arise when condition (weight-at-length) samples are not distributed proportional to population densities.  Our approach treats both density and condition as “categories” in VAST, and accounts for density-dependent condition by estimating a correlation between population density and condition. 

Here, we demonstrate our approach for the Pacific cod (Gadus macrocephalus) population of the eastern Bering Sea for 1982-2016. The model developed here does not include any environmental covariates, and is published as Grüss et al. (2020). 

Grüss, A., Gao, J., Thorson, J.T., Rooper, C., Thompson, G., Boldt, J. and Lauth, R. (2020) Estimating synchronous changes in condition and density in Eastern Bering Sea fishes. Marine Ecology Progress Series.

One key step for the estimation of density-dependent fish condition is the definition of the `Expansion_cz` object. In our case: ` Expansion_cz = matrix( c( 0, 1, 0, 0 ), nrow = 2, ncol = 2 )` which specifies that the predicted local abundance (the product of annual density and surface area) will be multiplied by the estimated annual condition to obtain annual indices of density-dependent fish condition. 

```R

######## Download release number 3.3.0; it is useful for reproducibility to use a specific release number
devtools::install_github( "james-thorson-noaa/FishStatsUtils", ref = "2.5.0" )
devtools::install_github( "james-thorson-noaa/VAST", ref = "3.3.0" )

######## Load packages
library( TMB )
library( VAST )

######## load data set
######## see `?load_example` for list of stocks with example data
######## that are installed automatically with `FishStatsUtils`.
example = load_example( data_set = "condition_and_density" )

######## Make settings
settings = make_settings( n_x = 50, Region = example$Region, purpose = "condition_and_density" )
settings$bias.correct = FALSE
Expansion_cz = matrix( c( 0, 1, 0, 0 ), nrow = 2, ncol = 2 )

######## Run model
fit = fit_model( "settings" = settings, "Lat_i" = example$sampling_data[,'Lat'], "Lon_i" = example$sampling_data[,'Lon'],
	"t_i" = example$sampling_data[,'Year'], "c_i" = as.numeric(example$sampling_data[,'Category'])-1,
  	"b_i" = example$sampling_data[,'Response_variable'], "a_i" = example$sampling_data[,'AreaSwept_km2'],
  	"v_i" = example$sampling_data[,'Vessel'], "Q_ik" = as.matrix(example$sampling_data[, "logLength_lncm"]), 
  	"Expansion_cz" = Expansion_cz, "knot_method"="grid" )

######## Inspect sample size for each data type
Nsamp_tc = tapply( example$sampling_data[,'Response_variable'], INDEX = list( 
	example$sampling_data[,'Year'], as.numeric(example$sampling_data[,'Category'])-1), FUN=length )

######## Produce and plot an abundance index and a density-dependent fish condition index
fit$years_to_plot = which( rowSums( is.na( Nsamp_tc ) ) == 0 )
Index = plot_biomass_index( DirName = paste0( getwd(),"/" ), TmbData = fit$data_list, 
	Sdreport = fit$parameter_estimates$SD, Year_Set = fit$year_labels, Years2Include = fit$years_to_plot, 
	use_biascorr = FALSE, category_names = c( "Abundance", "Fish condition" ) )

```
