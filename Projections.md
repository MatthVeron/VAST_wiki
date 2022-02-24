# End-of-century projections

Stakeholders and scientists are often interested in projecting a model forward in time under alternative climate scenarios.  In particular, end-of-century projections are useful to attribute changes to future climate projections.  This is feasible when fitting VAST by padding `t_i` to include future years, without fitting those dummy observations using `PredTF_i`.  However, this process requires including many more random effects, which can result in slow model fitting and difficulty exploring scenarios.

As alternative, `VAST` allows users to project a fitted model without re-fitting and without extra computational burden from estimating these random effects.  This feature uses `project_model` which is currently available in the `dev` branch.  It typically requires treating intercepts as random, and is more interesting when intercepts and spatio-temporal terms follow an autoregressive or random-walk process (as done below)

### Code demo 

```R
# Load package
library(VAST)

# load data set
example = load_example( data_set="EBS_pollock" )

# Make settings (turning off bias.correct to save time for example)
settings = make_settings( n_x = 50,
  Region = example$Region,
  purpose = "index2",
  bias.correct = FALSE )
settings$RhoConfig[] = 4

# Run model
fit = fit_model( settings = settings,
  Lat_i = example$sampling_data[,'Lat'],
  Lon_i = example$sampling_data[,'Lon'],
  t_i = example$sampling_data[,'Year'],
  b_i = example$sampling_data[,'Catch_KG'],
  a_i = example$sampling_data[,'AreaSwept_km2'],
  newtonsteps = 0 )

# Generate projections
Index_tr = numeric(0)
for( rI in 1:100 ){
  Sim = project_model( x = fit,
                        n_proj = 80,
                        new_covariate_data = NULL,
                        historical_uncertainty = "both",
                        seed = rI )
  Index_tr = cbind( Index_tr, Sim$Index_ctl[1,,1] )
  #Index_tr = cbind( Index_tr, Sim$mean_Z_ctm[1,,2] )
}

# Plot 90% interval for the index
Index_tq = t(apply(Index_tr, MARGIN=1, FUN=quantile, probs=c(0.1,0.5,0.9) ))
Index_tq[,2] = apply(Index_tr, MARGIN=1, FUN=mean)
matplot( y=Index_tq, log="", col="black", lwd=c(1,2,1), type="l", lty="solid" )
```