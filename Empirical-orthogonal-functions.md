
# Empirical Orthogonal Function (EOF) analysis for biological data

It is possible to use `VAST` to fit a generalization of empirical orthogonal function (EOF) analysis, while accounting for the noisy, zero-inflated and multivariate data that we have in fisheries.  To do so, take a univariate or multivariate model and perform rank-reduction (a.k.a. ordination) on years; this contrasts with "conventional" joint dynamic species distribution models, which ordinate on species. In this case, the model will then estimate spatial variability for each species, one or more annual indices of variability, as well a map for each mode and species indicating the deviation from uniform distribution associated with a positive value of each index for each species.  It is possible to visualize the statistical significance of each model component, e.g., to visualize whether index variability is "significant" or to identify locations that are "significantly" associated with a given index.

In this example, we apply EOF to five groundfishes in the eastern Bering Sea.  The 1st index (primary mode of variability) is highly correlated with the cold-pool extent. This was initially explored and documented in Thorson Ciannelli & Litzow (2020). Since that paper, however, we have improved the interface and model structure to control for persistent variability using the spatial term; this requires VAST release >= `3.4.0`.  This improvement has resulted in cold-pool extent being correlated with the primary mode of spatio-temporal variability, rather than the 3rd factor in the Thorson Ciannelli & Litzow (2020).

**Works cited**

Thorson, J. T., Ciannelli, L., & Litzow, M. A. (2020). Defining indices of ecosystem variability using biological samples of fish communities: A generalization of empirical orthogonal functions. Progress in Oceanography, 181, 102244. https://doi.org/10.1016/j.pocean.2019.102244

**Example code**

```R
# Load packages
library(VAST)

# load data set
example = load_example( data_set="five_species_ordination" )

# Make settings:
# including modifications from default settings to match 
# analysis in original paper
settings = make_settings( n_x=50,
  Region=example$Region,
  purpose="EOF2",
  n_categories=2,
  ObsModel=c(1,1),
  RhoConfig=c("Beta1"=0,"Beta2"=0,"Epsilon1"=0,"Epsilon2"=0) )

# Run model (including settings to speed up run)
fit = fit_model( settings=settings,
  Lat_i=example$sampling_data[,'Lat'],
  Lon_i=example$sampling_data[,'Lon'],
  t_i=example$sampling_data[,'Year'],
  c_i=example$sampling_data[,'species_number']-1,
  b_i=example$sampling_data[,'Catch_KG'],
  a_i=example$sampling_data[,'AreaSwept_km2'],
  newtonsteps=0,
  getsd=FALSE,
  Use_REML=TRUE )

# Plot results, including spatial term Omega1
results = plot( fit,
  check_residuals=FALSE,
  plot_set=c(3,16) )

# Plot factor representation
plot_factors( Report=fit$Report,
  ParHat=fit$ParHat,
  Data=fit$data_list,
  mapdetails_list=results$map_list,
  Year_Set=fit$year_labels )
```

However, as with any multivariate model this model may take a long time (hours-days) to fit.  If this is happening please try one or more ways to simplify the problem:

* Use fewer extrapolation-grid cells.  With recent versions of VAST, this can be done e.g., using `max_cells=1000` for 1000 extrapolation-grid cells.
* Use fewer factors, via decreasing `FieldConfig` in `make_data(.)` and/or `n_categories` in `make_settings(.)`
* Use fewer knots, via decreasing `n_x` in `make_settings(.)`
* Use fewer categories and/or years of data
* Eliminate bias-correction and/or run without getting standard errors.  
* Get access to a better computer
* Use MRAN for windows, or other version of R that uses parallelization.