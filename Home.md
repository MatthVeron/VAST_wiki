# Overview
Welcome to the wiki for the Vector Autoregressive Spatio-Temporal (VAST) model.

I post most materials regarding `VAST` on the wiki of its main dependency, package `SpatialDeltaGLMM`, and [please see materials here](https://github.com/nwfsc-assess/geostatistical_delta-GLMM/wiki).  However, I will add materials to the `VAST` wiki that apply only to `VAST`.

### How to simulate new data

I am adding a new feature starting with Version 4.4.0 (i.e., `Version=VAST_v4_4_0`) that allows users to generate a new data set using a parametric bootstrap.  To do so, (1) build a model using an existing data set, (2) estimate parameters, and (3) use the `simulate` feature inherited from `TMB`

```R
# Build object using standard inputs including the original data set `data`
Obj = Build_TMB_Fn( TmbData=data, ... )
# Optimize
Opt = TMBhelper::Optimize( obj=Obj )
# Simulate new data
Sim = Obj.simulate( complete=TRUE )
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

