# Estimating species interactions using a spatial Model of Intermediate Complexity

Starting with `Version = "VAST_v5_1_0"`, users can estimate species interactions similar to how it is done in package MIST (R package available [here](https://github.com/James-Thorson/MIST)).  This can be done using the simplified wrapper functions (although you made need the development version to access the reposited data and example output):

```R
# load libraries
library(VAST)

# load data
example = load_example( "GOA_MICE_example" )

# Get settings
settings = make_settings( n_x=50, purpose="MICE", Region=example$Region,
  n_categories=nlevels(example$sampling_data$spp) )

# Modify defaults
settings$VamConfig['Rank'] = 1  # Reduce to single axis of ratio-dependent interactions
settings$fine_scale = FALSE  # Make it run faster

# Load previous estimates to speed it up
  # Requires loading the previous Kmeans results to get identical results
test_path = file.path(system.file("extdata", package="VAST"),"GOA_MICE_example")
load( file.path(test_path,"saved_estimates.RData") )

# Run model
  # May take many hours, even given informative starting value
Fit = fit_model( "settings"=settings, "Lat_i"=example$sampling_data[,'Lat'],
  "Lon_i"=example$sampling_data[,'Lon'], "b_i"=example$sampling_data[,'Catch_KG'],
  "a_i"=example$sampling_data[,'AreaSwept_km2'], "v_i"=as.numeric(example$sampling_data[,'Vessel']),
  "t_i"=example$sampling_data[,'Year'], "c_i"=as.numeric(example$sampling_data[,'spp'])-1, "F_ct"=example$F_ct,
  "newtonsteps"=0, optimize_args=list("getsd"=FALSE, "startpar"=parameter_estimates$par),
  "working_dir"=test_path )

# Fix issue in auto-generated plot labels
Fit$year_labels = c( 1983, Fit$year_labels )
Fit$years_to_plot = c(1, Fit$years_to_plot+1)

# Plots
plot_results( fit=Fit, settings=settings )
```

Alternatively, it can be fitted using lower-level functions .  The MICE in space model is only possible when using a Poisson-link delta model (`ObsModel[2]=1`), and is done by including a new, optional input `VamConfig` to `Data_Fn`:

```R
# Interaction settings
Rank = Nspecies # Add option here
VamConfig = c("Method"=1, "Rank"=Rank, "Timing"=0)
# Other non-standard options
FieldConfig = c("Omega1"=Nspecies, "Epsilon1"=Nspecies, "Omega2"=0, "Epsilon2"=0)
OverdispersionConfig = c("Eta1"=0, "Eta2"=0)
RhoConfig = c("Beta1"=3, "Beta2"=3, "Epsilon1"=4, "Epsilon2"=0)
# Build data
Data = Data_Fn( ..., VamConfig=VamConfig, FieldConfig=FieldConfig, RhoConfig=RhoConfig )
```

where `Rank` represents the number of "axes of regulation" that are apparent in community data (Rank <= Nspecies). A few considerations are then needed:

1.  Do you want to attribute all temporal variation to spatio-temporal variation and interactions, like is done in package `MIST`?  If yes, then turn off all temporal variation in intercepts (`RhoConfig[1:2] = 3`).

2.  Do you want to estimate partial regulation (reduced rank for the interaction matrix)?  If yes, then consider turning on some temporal structure on spatio-temporal variation, i.e., autoregressive (`RhoConfig[3]=4`) or random-walk structure (`RhoConfig[3]=2`).

3.  Do you want to model additional variation beyond what is explained by interactions?  Then perhaps turn on spatial, spatio-temporal, or variation in intercepts for the 2nd linear predictor.  