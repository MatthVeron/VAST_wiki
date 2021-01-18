VAST has recently added two options to visualize covariate response curves:
1.  using package `effects` to visualize "conditional" response curves, showing the additive impact of the covariate upon the linear predictors. This does not visualize the effect of spatially varying coefficients, or responses after applying the inverse-link function; 
2.  using package `pdp` (and interfacing with `predict.fit_model`) to visualize "marginal" response curves (a.k.a. partial dependence plots), showing the effect of an exogenous change in any covariate for every observation, and then averaging across observations to calculate the marginal effect given the sample covariance among covariates.  This does visualize the effect of spatially varying coefficients, as well as the action of the inverse-link function or when summing the impact of a given covariate across both linear predictors, but is also relatively slow and can yield a nonparametric (difficult to summarize) response curve even for a parametric functional form.

These currently both require the development branch.  The options are shown using a simple model with a spatially varying response to cold-pool extent in the eastern Bering Sea.

```R
# Load packages
library(VAST)

# load data set
example = load_example( data_set="EBS_pollock" )
covariate_data = data.frame( "Lat"=0, "Lon"=0, "Year"=example$covariate_data[,'Year'],
  "CPE"=(example$covariate_data[,'AREA_SUM_KM2_LTE2']-100000)/100000)

# Make settings (turning off bias.correct to save time for example)
settings = make_settings( n_x=100,
  Region=example$Region,
  purpose="index2" )

# Change Beta1 to AR1, to allow linear covariate effect
settings$RhoConfig['Beta1'] = 4

# Define formula.
X1_formula = ~ CPE
X2_formula = ~ 1

#
X1config_cp = array(3,dim=c(1,1))

# Run model
fit = fit_model( "settings" = settings,
  Lat_i = example$sampling_data[,'Lat'],
  Lon_i = example$sampling_data[,'Lon'],
  t_i = example$sampling_data[,'Year'],
  b_i = example$sampling_data[,'Catch_KG'],
  a_i = example$sampling_data[,'AreaSwept_km2'],
  X1_formula = X1_formula,
  X2_formula = X2_formula,
  X1config_cp = X1config_cp,
  covariate_data = covariate_data,
  test_fit = FALSE )

#####################
# Effects package
#####################

library(effects)  # Used to visualize covariate effects

# Must add data-frames to global environment (hope to fix in future)
covariate_data_full = fit$effects$covariate_data_full
catchability_data_full = fit$effects$catchability_data_full

# Plot 1st linear predictor
pred = Effect.fit_model( fit,
  focal.predictors = c("CPE"),
  which_formula = "X1" )
plot(pred)

#####################
# pdp package
#####################

library(pdp)

# Make function to interface with pdp
pred.fun = function( object, newdata ){
  predict( x=object,
    Lat_i = object$data_frame$Lat_i,
    Lon_i = object$data_frame$Lon_i,
    t_i = object$data_frame$t_i,
    a_i = object$data_frame$a_i,
    what = "P1_iz",
    new_covariate_data = newdata,
    do_checks = FALSE )
}

# Run partial
Partial = partial( object = fit,
                   pred.var = "CPE",
                   pred.fun = pred.fun,
                   train = fit$covariate_data )

# Make plot using ggplot2
library(ggplot2)
autoplot(Partial)
```