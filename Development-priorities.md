# List of development priorities and rationale

Based on feedback from the community of users, as well as discussions with the steering committee, we will attempt to update a list of future development priorities here.

| Change | Rationale| Link for details |
| ------------- | ------------- | ------------- |
| Explore oneStepPredict(.) for delta models  | Allow better use of probability-integral-transform diagnostics  | |
| Add formula interface for X1 and X2 separately, and modify CPP, make_data, and Xconfig_zcp to allow its use  | Simplify user interface for covariates in delta model | | 
| Add plots for marginal or partial dependence of covariates | Simplify user interface and model exploration | https://github.com/James-Thorson-NOAA/VAST/issues/240#event-3486935522 | 
| Add formula interface for Q1 and Q2 separately, and modify CPP, make_data to allow its use  | Simplify user interface for catchability covariates in delta model | | 
| Add Q1config_cp and Q2config_cp to allow spatially varying catchability response | Allow for spatially varying gear performance estimation | | 
| Add generalized lognormal-gamma distribution | Explore for improved performance w.r.t. diagnostics and index scale | | 
| Add climate velocity as automated output based on raster of density predictions | Additional interpretation of existing outputs | https://besjournals.onlinelibrary.wiley.com/doi/full/10.1111/2041-210X.13295 | 
| Add Hurlbert and other additional overlap metrics | Additional flexibility in multivariate modelling | https://onlinelibrary.wiley.com/doi/abs/10.1111/geb.12984 | 
| Add predict S3 method | Improve user interface, e.g., for use when predicting bycatch |
| Add options to visualize covariate responses | Improve user interface when interpreting results |
| Add stepwise AIC model-selection algorithm | Improve user interface when building models | 
| Add options to automatically query publicly available data to population `covariate_data` to use in formula interface | Facilitate exploring covariates for introductory users |  
| Merge with EOFR package by adding optional new input that is correlated with EOF axis | Facilitate exploration of spatial drivers for population dynamics | 
