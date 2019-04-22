# Expanding age/length composition data for use in stock assessments

It is possible to use `VAST` to conduct "ordination."  This process estimates a reduced set of axes that collectively explain variability in a data set.  By identifying species that have similar "loadings" for these different axes, ordination can identify species that are similar.  As an example, please see code below.  At time of writing, this requires installing the development branch of FishStatsUtils, although future releases will include this code in the master branch and I may forget to update this page.

```R
# Download development branch (or perhaps master branch in future releases)
devtools::install_github("james-thorson/VAST", ref="development")
devtools::install_github("james-thorson/FishStatsUtils", ref="development")

setwd( "D:/UW Hideaway (SyncBackFree)/AFSC/2019-04 -- Wrapper function demo for ordination" )

# Load packages
library(TMB)
library(VAST)

# load data set
# see `?load_example` for list of stocks with example data
# that are installed automatically with `FishStatsUtils`.
example = load_example( data_set="five_species_ordination" )

# Make settings
settings = make_settings( n_x=100, Region=example$Region, purpose="ordination",
  strata.limits=example$strata.limits, n_categories=2 )

# Change settings from defaults
settings$RhoConfig[c('Beta1','Beta2')] = 2

# Run model
fit = fit_model( "settings"=settings, "Lat_i"=example$sampling_data[,'Lat'], "Lon_i"=example$sampling_data[,'Lon'],
  "t_i"=example$sampling_data[,'Year'], "c_i"=as.numeric(example$sampling_data[,"species_number"])-1,
  "b_i"=example$sampling_data[,'Catch_KG'], "a_i"=example$sampling_data[,'AreaSwept_km2'] )

# Plot results
results = plot_results( settings=settings, fit=fit )
```