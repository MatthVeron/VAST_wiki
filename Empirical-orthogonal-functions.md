# Empirical Orthogonal Function (EOF) analysis for biological data

It is possible to use `VAST` to fit a generalization of empirical orthogonal function (EOF) analysis, while accounting for the noisy, zero-inflated and multivariate data that we have in fisheries.  To do so, take a univariate or multivariate model and switch the indexing for years (`t_i`) and categories (`c_i`) such that the model is performing rank-reduction (a.k.a. ordination) on years rather than categories. Then also specify a Poisson-link delta-model, `ObsModel=c(2,1)` with e.g. `FieldConfig=c(0,3,0,0)` to estimate three modes of variability.  In this case, the model will then estimate `n_factors=3` annual indices of variability, as well a map for each mode and species indicating the deviation from uniform distribution associated with a positive value of each index for each species.

```R
# Load packages
library(TMB)
library(VAST)

# load data set
# see `?load_example` for list of stocks with example data
# that are installed automatically with `FishStatsUtils`.
example = load_example( data_set="five_species_ordination" )

# Make settings
settings = make_settings( n_x=50, Region=example$Region, purpose="EOF",
  strata.limits=example$strata.limits, n_categories=3, bias.correct=FALSE ) 

# Run model
fit = fit_model( "settings"=settings, "Lat_i"=example$sampling_data[,'Lat'], "Lon_i"=example$sampling_data[,'Lon'],
  "t_i"=as.numeric(example$sampling_data[,"species_number"]), 
  "c_i"=match(example$sampling_data[,'Year'],sort(unique(example$sampling_data[,'Year'])))-1,
  "b_i"=example$sampling_data[,'Catch_KG'], "a_i"=example$sampling_data[,'AreaSwept_km2'] )

# Plot results
results = plot( fit, check_residuals=FALSE )

# Plot factor representation
plot_factors( Report=fit$Report, ParHat=fit$ParHat, Data=fit$data_list, mapdetails_list=results$map_list )
```

In this example, we apply EOF to five groundfishes in the eastern Bering Sea.  The 3rd index then is highly correlated with the cold-pool extent, as explored in Thorson Ciannelli & Litzow (2020).

**Works cited**

Thorson, J.T., Ciannelli, L. and Litzow, M. (2020) Defining indices of ecosystem variability using biological samples of fish communities: a generalization of empirical orthogonal functions. Progress In Oceanography.