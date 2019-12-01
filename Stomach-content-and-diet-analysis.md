 # Joint model for stomach content, predator biomass catch rate, and predator-expanded-stomach-contents (PESCs)

We developed an approach that fits a spatio-temporal model with `VAST` to both prey-biomass-per-predator-biomass data (i.e., the ratio of prey biomass in stomachs to predator weight) and predator biomass catch rate data (predator biomass per unit area), to predict “predator-expanded-stomach-contents” (PESC; the product of prey-biomass-per-predator-biomass, predator biomass per unit area, and surface area). The PESC estimates can be used to visualize either the annual landscape of PESC (spatio-temporal variation), or can be aggregated across space to calculate annual variation in diet proportions (variation among prey items and among years). 

Here, we demonstrate our approach in a data-limited situation involving West Florida Shelf red grouper (Epinephelus morio, Epinephelidae) for 2011-2015. Four prey items are considered: crabs, fish, shrimps, and “other prey”. We demonstrate how diet proportions are calculated from the PESCs estimated by our model. 

One key step for the estimation of PESCs is the definition of the `Expansion_cz` object. In our case:
`Expansion_cz = matrix( c(0,1,1,1,1,0,0,0,0,0), nrow = nlevels(example$sampling_data[,"spp"]), ncol = 2 )`
which entails that the estimated biomass (the product of biomass per unit area and surface area) for the first “category”, namely the predator (red grouper), will be multiplied by the estimated biomasses for the other categories, namely the prey items (crabs, fish, other prey, and shrimps), to obtain PESC estimates (for crabs, fish, other prey, and shrimps). 

```R
######## Download release number 3.2.2; it is useful for reproducibility to use a specific release number
devtools::install_github( "james-thorson-noaa/FishStatsUtils", ref="2.4.0" )
devtools::install_github( "james-thorson-noaa/VAST", ref="3.2.2" )

######## Load packages
library( TMB )
library( VAST )

######## load data set
######## see `?load_example` for list of stocks with example data
######## that are installed automatically with `FishStatsUtils`.
example = load_example( data_set="PESC_example_red_grouper" )

######## Make settings
settings = make_settings( n_x=300, Region=example$Region, purpose="index",
	strata.limits=example$strata.limits )

######## Change settings from defaults
settings$ObsModel = c( 2, 1 )
settings$Options[2:4] = FALSE
settings$use_anisotropy = FALSE
settings$fine_scale = FALSE
Expansion_cz = matrix( c(0,1,1,1,1,0,0,0,0,0), nrow = nlevels(example$sampling_data[,"spp"]), ncol = 2 ) 

######## Run model
fit = fit_model( "settings"=settings, "Lat_i"=example$sampling_data[,'Lat'], "Lon_i"=example$sampling_data[,'Lon'],
  	"t_i"=example$sampling_data[,'Year'], "c_i"=as.numeric(example$sampling_data[,"spp"])-1,
  	"b_i"=example$sampling_data[,'Catch_KG'], "a_i"=example$sampling_data[,'AreaSwept_km2'], 
	"Expansion_cz"=Expansion_cz, "input_grid"=example$input_grid, 
	"knot_method"="grid", "Npool"=20, "newtonsteps"=1, "getsd"=TRUE, test_fit=FALSE )

######## Calculate and plot diet proportions
Index = plot_biomass_index( DirName=paste0(getwd(),"/"), TmbData=fit$data_list, Sdreport=fit$parameter_estimates$SD, 
	Year_Set=fit$year_labels, Years2Include=fit$years_to_plot, use_biascorr=TRUE, 
	category_names=levels(as.factor(example$sampling_data[,'spp'])) )
proportions = calculate_proportion( fit$data_list, Index=Index, Expansion_cz=Expansion_cz, 
	Year_Set=fit$year_labels, strata_names=NULL, 
	category_names=levels(as.factor(example$sampling_data[,'spp'])), PlotName2=NA )

```
