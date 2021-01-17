# List of development priorities and rationale

### Pending development priorities
Based on feedback from the community of users, as well as discussions with the steering committee, we will attempt to update a list of future development priorities here.

| Change | Rationale| Link for details |
| ------------- | ------------- | ------------- |
| Explore oneStepPredict(.) for delta models  | Allow better use of probability-integral-transform diagnostics  | |
| Add generalized lognormal-gamma distribution | Explore for improved performance w.r.t. diagnostics and index scale | | 
| Add climate velocity as automated output based on raster of density predictions | Additional interpretation of existing outputs | https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/2041-210X.13295 | 
| Add Hurlbert and other additional overlap metrics | Additional flexibility in multivariate modelling | https://onlinelibrary.wiley.com/doi/abs/10.1111/geb.12984 | 
| Add stepwise AIC model-selection algorithm | Improve user interface when building models | 
| Add options to automatically query publicly available data to population `covariate_data` to use in formula interface | Facilitate exploring covariates for introductory users |  
| Merge with EOFR package by adding optional new input that is correlated with EOF axis | Facilitate exploration of spatial drivers for population dynamics | 
| Add option to specify correlations based on traits for each category c | |
| Implement structural equation modelling features via covariance matrix specification | |
| Add feature to pass through covariate names to outputs for parameter estimates and plotting | |
| Integrate `units` package to provide automatic error checks and plotting labels | |
| Add options to use p-splines | Improved statistical efficiency relative to basis-splines that are currently available | 
| Explore multi-scale correlation functions, via additive function of two Omega GMRFs operating with different kappas | Assimilate within and among-transect locational data better | https://github.com/James-Thorson-NOAA/VAST/issues/273 |

### Completed development priorities
Please consult the [NEWS document](https://github.com/James-Thorson-NOAA/VAST/blob/master/manual/NEWS.pdf) for a list of previous releases, listing features satisfying previous development priorities.  
