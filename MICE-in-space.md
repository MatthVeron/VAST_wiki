# Estimating species interactions using a spatial Model of Intermediate Complexity

Starting with `Version = "VAST_v5_1_0"`, users can estimate species interactions similar to how it is done in package MIST (R package available [here](https://github.com/James-Thorson/MIST)).  This can be done using the simplified wrapper functions:

```R
# Get settings
settings = make_settings( n_x=n_x, purpose="MICE", Region=Region, n_categories=length(Species_set) )

# Run model
Fit = fit_model( "settings"=settings, "Lat_i"=Data_Geostat[,'Lat'], 
  "Lon_i"=Data_Geostat[,'Lon'], "b_i"=Data_Geostat[,'Catch_KG'], 
  "a_i"=Data_Geostat[,'AreaSwept_km2'], "v_i"=as.numeric(Data_Geostat[,'Vessel'])-1,
  "t_i"=Data_Geostat[,'Year'], "c_i"=Data_Geostat[,'spp']-1, "F_ct"=F_ct )

# Plots
plot_results( fit=Fit, settings=settings )
```

Alternatively, it can be fitted using lower-level functions as shown below.  The MICE in space model is only possible when using a Poisson-link delta model (`ObsModel[2]=1`), and is done by including a new, optional input to `Data_Fn`:

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

# Big picture conclusions

So far I have done some limited testing of the new "interactions" feature.  A few conclusions:

1.  I get identical parameter estimates and marginal likelihood when using Timing=0 (`VamConfig[3]==0`) or Timing=1 (`VamConfig[3]=1`).  Timing=0 seems slightly faster and also easier to compare with `VAST` model results without interactions.  So I recommend using Timing=0 by default.

2.  I also get very similar parameter estimates to package `MIST` when using:

```R
# Settings to be similar to MIST
VamConfig = c("Method"=1, "Rank"=Rank, "Timing"=0)
FieldConfig = c("Omega1"="IID", "Epsilon1"=Nspecies, "Omega2"=0, "Epsilon2"=0)
RhoConfig = c("Beta1"=3, "Beta2"=3, "Epsilon1"=4, "Epsilon2"=0)
```

so this seems like a good starting point.

3.  Simulation testing suggests that `VAST` gives unbiased estimates of interaction sign and strength when simulation tested using the simulator from MIST ([here](https://github.com/James-Thorson/MIST/blob/master/R/Sim_Fn.R))
