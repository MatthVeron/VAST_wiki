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

You can then fit this new data set easily and compare results with your original fit
```R
# Build new object using simulated data `Sim`
Obj_new = Build_TMB_Fn( TmbData=Sim, ... )
# Re-optimize
Opt_new = TMBhelper::Optimize( obj=Obj_new  )
# Compare results
cbind( summary(Opt$SD,"fixed"), summary(Opt_new$SD,"fixed") )
```

# Demonstration
For example, a user can conduct a simulation test for Alaska pollock using the following code

```R
# Download release number 3.0.0; its useful for reproducibility to use a specific release number
devtools::install_github("james-thorson/VAST", ref="3.0.0")

RootDir = "C:/Users/James.Thorson/Desktop/UW Hideaway/NWFSC/2018-06 -- testing simulate feature/"

library(TMB)
library(VAST)

Data_Set = "EBS_pollock"

Version = get_latest_version( package="VAST" )
Method = c("Grid", "Mesh", "Spherical_mesh")[2]
grid_size_km = 25
n_x = 50   # Specify number of stations (a.k.a. "knots")
Kmeans_Config = list( "randomseed"=1, "nstart"=100, "iter.max"=1e3 )
FieldConfig = c("Omega1"=1, "Epsilon1"=1, "Omega2"=1, "Epsilon2"=1)
OverdispersionConfig = c("Eta1"=0, "Eta2"=0)
Options =  c("Calculate_Range"=0, "Calculate_effective_area"=0)

RhoConfig = c("Beta1"=4, "Beta2"=3, "Epsilon1"=0, "Epsilon2"=0)
ObsModel = c(2,1)

# Default
strata.limits <- data.frame('STRATA'="All_areas")
Region = "Eastern_Bering_Sea"

# Save
Date = Sys.Date()
DateFile = paste0(RootDir,'/',Date,'_V1/')
dir.create(DateFile)

# Load data
data( EBS_pollock_data )
Data_Geostat = data.frame( "Catch_KG"=EBS_pollock_data[,'catch'], "Year"=EBS_pollock_data[,'year'], "Vessel"="missing", "AreaSwept_km2"=0.01, "Lat"=EBS_pollock_data[,'lat'], "Lon"=EBS_pollock_data[,'long'], "Pass"=0)
Extrapolation_List = make_extrapolation_info( Region=Region, strata.limits=strata.limits )
Spatial_List = make_spatial_info( grid_size_km=grid_size_km, n_x=n_x, Method=Method, Lon=Data_Geostat[,'Lon'], Lat=Data_Geostat[,'Lat'], Extrapolation_List=Extrapolation_List, randomseed=Kmeans_Config[["randomseed"]], nstart=Kmeans_Config[["nstart"]], iter.max=Kmeans_Config[["iter.max"]], DirPath=DateFile, Save_Results=TRUE )
Data_Geostat = cbind( Data_Geostat, "knot_i"=Spatial_List$knot_i )

# Some settings
Nrep = 10
Year_Set = seq(min(Data_Geostat[,'Year']),max(Data_Geostat[,'Year']))
Years2Include = which( Year_Set %in% sort(unique(Data_Geostat[,'Year'])))

# Run first time
Data_orig = make_data("Version"=Version, "FieldConfig"=FieldConfig, "OverdispersionConfig"=OverdispersionConfig, "RhoConfig"=RhoConfig, "ObsModel"=ObsModel, "c_i"=rep(0,nrow(Data_Geostat)), "b_i"=Data_Geostat[,'Catch_KG'], "a_i"=Data_Geostat[,'AreaSwept_km2'], "v_i"=as.numeric(Data_Geostat[,'Vessel'])-1, "s_i"=Data_Geostat[,'knot_i']-1, "t_i"=Data_Geostat[,'Year'], "spatial_list"=Spatial_List, "Options"=Options )
TmbList_orig = make_model("TmbData"=Data_orig, "RunDir"=DateFile, "Version"=Version, "RhoConfig"=RhoConfig, "loc_x"=Spatial_List$loc_x, "Method"=Method)
Obj_orig = TmbList_orig[["Obj"]]

Opt_orig = TMBhelper::fit_tmb( obj=Obj_orig, lower=TmbList_orig[["Lower"]], upper=TmbList_orig[["Upper"]], getsd=TRUE, savedir=DateFile, bias.correct=TRUE, newtonsteps=1, bias.correct.control=list(sd=FALSE, split=NULL, nsplit=1, vars_to_correct="Index_cyl") )
Report_orig = Obj_orig$report()
Save_orig = list("Opt"=Opt_orig, "Report"=Report_orig, "ParHat"=Obj_orig$env$parList(Opt_orig$par), "Data"=Data_orig)
save(Save_orig, file=paste0(DateFile,"Save_orig.RData"))

# Change some values to demonstrate capacity to change operating model
Par = Obj_orig$env$last.par.best
# Double decorrelation distance
Par[c("logkappa1","logkappa2")] = Par[c("logkappa1","logkappa2")] - log(2)

# Loop through OM
for( rI in 1:Nrep ){
  Keep = FALSE
  while( Keep==FALSE ){
    Data_sim = Obj_orig$simulate( par=Par, complete=TRUE )
    Enc_t = tapply( Data_sim$b_i, INDEX=Data_orig$t_i, FUN=function(vec){mean(vec>0)})
    if( all(Enc_t>0 & Enc_t<1) ) Keep = TRUE
  }
  save(Data_sim, file=paste0(DateFile,"Data_sim",rI,".RData"))
}

# Loop through EM
for( rI in 1:Nrep ){
  load(file=paste0(DateFile,"Data_sim",rI,".RData"))
  RhoConfig = c("Beta1"=0, "Beta2"=0, "Epsilon1"=0, "Epsilon2"=0)
  ObsModel = c(2,0)

  Data_new = make_data("b_i"=Data_sim$b_i, "Version"=Version, "FieldConfig"=FieldConfig, "OverdispersionConfig"=OverdispersionConfig, "RhoConfig"=RhoConfig, "ObsModel"=ObsModel, "c_i"=rep(0,nrow(Data_Geostat)), "a_i"=Data_Geostat[,'AreaSwept_km2'], "v_i"=as.numeric(Data_Geostat[,'Vessel'])-1, "s_i"=Data_Geostat[,'knot_i']-1, "t_i"=Data_Geostat[,'Year'], "spatial_list"=Spatial_List, "Options"=Options )
  TmbList_new = make_model("TmbData"=Data_new, "RunDir"=DateFile, "Version"=Version, "RhoConfig"=RhoConfig, "loc_x"=Spatial_List$loc_x, "Method"=Method)
  Obj_new = TmbList_new[["Obj"]]

  Opt_new = TMBhelper::fit_tmb( obj=Obj_new, lower=TmbList_new[["Lower"]], upper=TmbList_new[["Upper"]], getsd=TRUE, savedir=DateFile, bias.correct=TRUE, newtonsteps=1, bias.correct.control=list(sd=FALSE, split=NULL, nsplit=1, vars_to_correct="Index_cyl") )
  Report_new = Obj_new$report()

  Index = plot_biomass_index( DirName=DateFile, TmbData=Data_new, Sdreport=Opt_new[["SD"]], Year_Set=Year_Set, Years2Include=Years2Include, use_biascorr=TRUE )
  Save_new = list("Opt"=Opt_new, "Report"=Report_new, "ParHat"=Obj_new$env$parList(Opt_new$par), "Data"=Data_new, "Index"=Index)
  save(Save_new, file=paste0(DateFile,"Save_new_",rI,".RData"))
}

# Compile results
Index_array = array(NA, dim=c(Nrep,length(Year_Set),3,2), dimnames=list(paste0("Rep_",1:Nrep),Year_Set,c("Orig","True","Est"),c("Index","SE")) )
for( rI in 1:Nrep ){
  load(file=paste0(DateFile,"Save_orig.RData"))
  load(file=paste0(DateFile,"Data_sim",rI,".RData"))
  load(file=paste0(DateFile,"Save_new_",rI,".RData"))
  Index_array[rI,,'Orig','Index'] = Save_orig$Report$Index_cyl[1,,1]
  Index_array[rI,,'True','Index'] = Data_sim$Index_cyl[1,,1]
  Index_array[rI,,'Est','Index'] = Save_new$Index$Table[,'Estimate_metric_tons'] # Save_new$Report$Index_cyl[1,,1]
}

# Compile data frame of results
DF = cbind( expand.grid(dimnames(Index_array[,,'True','Index'])), "Orig"=as.vector(Index_array[,,'Orig','Index']), "True"=as.vector(Index_array[,,'True','Index']), "Est"=as.vector(Index_array[,,'Est','Index']) )

# Test hyper-stability, should be 1.00
Lm = lm( log(Est) ~ 0 + factor(Var1) + log(True), data=DF )
summary(Lm)$coef['log(True)',]
# Test bias in average, should be 1.00
Lm = lm( Est/True ~ 1, data=DF )
summary(Lm)$coef['(Intercept)',]
```
