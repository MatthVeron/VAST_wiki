### How to simulate new data

I am adding a new feature starting with Version 4.4.0 (i.e., `Version=VAST_v4_4_0`) that allows users to generate a new data set using a parametric bootstrap.  To do so, (1) build a model using an existing data set, (2) estimate parameters, and (3) use the `simulate` feature inherited from `TMB`

```R
# Build object using standard inputs including the original data set `data`
Obj = Build_TMB_Fn( TmbData=data, ... )
# Optimize
Opt = TMBhelper::Optimize( obj=Obj )
# Simulate new data
Sim = Obj$simulate( complete=TRUE )
```

You can then fit this new data set easily and compare results with your original fit. In the following I load the development branch to use the function `FishStatsUtils::simulate_data(.)`, which will be added to the main branch in FishStatsUtils release number 2.6.0. 
```R
devtools::install_github( "C:/Users/James.Thorson/Desktop/Git/FishStatsUtils", ref="development" )
devtools::install_github( "C:/Users/James.Thorson/Desktop/Git/VAST", ref="development" )

# Define working directory for saving files
working_dir = "C:/Users/James.Thorson/Desktop/Work files/AFSC/2020-02 -- Test simulate_data/"

# Load packages
library(VAST)

# load data set
# see `?load_example` for list of stocks with example data
# that are installed automatically with `FishStatsUtils`.
example = load_example( data_set="EBS_pollock" )

# Make settings (changing defaults to mis-specify OM and EM)
settings = make_settings( n_x=100, Region=example$Region, purpose="index",
  strata.limits=example$strata.limits, bias.correct=FALSE, ObsModel=c(2,1),
  RhoConfig=c("Beta1"=2, "Beta2"=2, "Epsilon1"=0, "Epsilon2"=0) )

# Run model
fit_orig = fit_model( "settings"=settings, "Lat_i"=example$sampling_data[,'Lat'],
  "Lon_i"=example$sampling_data[,'Lon'], "t_i"=example$sampling_data[,'Year'],
  "c_i"=rep(0,nrow(example$sampling_data)), "b_i"=example$sampling_data[,'Catch_KG'],
  "a_i"=example$sampling_data[,'AreaSwept_km2'], "v_i"=example$sampling_data[,'Vessel'],
  "getJointPrecision"=TRUE, "working_dir"=working_dir )

# Some settings
Nrep = 10
Type = 3  # What type of simulator

# Save initial fit
save(fit_orig, file=paste0(working_dir,"fit_orig.RData"))

# Loop through OM
for( rI in 1:Nrep ){
  Keep = FALSE

  # Only keep replicate with encounters <100% and >0% in every year
  while( Keep==FALSE ){
    Data_sim = simulate_data( fit_orig, type=Type )
    Enc_t = tapply( Data_sim$b_i, INDEX=Data_sim$t_i, FUN=function(vec){mean(vec>0)})
    if( all(Enc_t>0 & Enc_t<1) ) Keep = TRUE
  }
  save(Data_sim, file=paste0(working_dir,"Sim_",rI,".RData") )
}

# Loop through EM
for( rI in 1:Nrep ){

  # Load data for replicate
  load( file=paste0(working_dir,"Sim_",rI,".RData") )

  # Change settings for EM relative to OM
  settings_em = settings
  settings_em[['RhoConfig']] = c("Beta1"=0, "Beta2"=0, "Epsilon1"=0, "Epsilon2"=0)
  settings_em[['ObsModel']] = c(2,0)
  settings_em[['n_x']] = 50
  settings_em[['bias.correct']] = TRUE

  # Run EM on simulated data
  fit_sim = fit_model( "settings"=settings_em, "Lat_i"=example$sampling_data[,'Lat'],
    "Lon_i"=example$sampling_data[,'Lon'], "t_i"=example$sampling_data[,'Year'],
    "c_i"=rep(0,nrow(example$sampling_data)), "b_i"=Data_sim$b_i,
    "a_i"=example$sampling_data[,'AreaSwept_km2'], "v_i"=example$sampling_data[,'Vessel'],
    "working_dir"=working_dir, "newtonsteps"=0 )

  # Compute and save abundance-index for EM
  Index = plot_biomass_index( DirName=working_dir, TmbData=fit_sim$data_list,
    use_biascorr=TRUE, Sdreport=fit_sim$parameter_estimates$SD,
    Year_Set=fit_sim$year_labels, Years2Include=fit_sim$years_to_include )
  save(Index, file=paste0(working_dir,"Index_",rI,".RData"))
}

# Compile results
Index_rtzz = array(NA, dim=c(Nrep,length(fit_orig$year_labels),2,2),
  dimnames=list(paste0("Rep_",1:Nrep), fit_orig$year_labels, c("True","Est"), c("Index","SE")) )
for( rI in 1:Nrep ){
  load( file=paste0(working_dir,"Sim_",rI,".RData") )
  load( file=paste0(working_dir,"Index_",rI,".RData") )
  Index_rtzz[rI,,'True','Index'] = Data_sim$Index_cyl[1,,1]
  Index_rtzz[rI,,'Est','Index'] = Index$Table[,'Estimate_metric_tons']
}

# Convert results to data-frame for use in linear model using lm(.)
DF = cbind( expand.grid(dimnames(Index_rtzz[,,'True','Index'])),
  "True"=as.vector(Index_rtzz[,,'True','Index']),
  "Est"=as.vector(Index_rtzz[,,'Est','Index']) )

# Test hyper-stability, should be 1.00
Lm = lm( log(Est) ~ 0 + factor(Var1) + log(True), data=DF )
summary(Lm)$coef['log(True)',]
# Test bias in average, should be 1.00
Lm = lm( Est/True ~ 1, data=DF )
summary(Lm)$coef['(Intercept)',]
```
