### Custom modifications for advanced users

For advanced users, it is sometimes convenient to modify the default data and parameters that are created using the high level wrapper `fit_model` or even the internal logical functions `make_data`, `make_map` or `make_parameters`.  This can be useful e.g. to:
1.  Specify restrictions on covariate effects to be constant among species;
2.  Specify restrictions on loadings matrices to implement delta-change methods, such that individual variables are driven by the same spatio-temporal factors;
3.  Modify the extrapolation-area constructions to fine-tune area expansion methods;
4.  Modify the coordinates used when calculating center-of-gravity;
and for many other reasons.

These changes can generally be accomplished by first building the model without running it, then making modifications as needed, and the passing those modified inputs to overwrite the default constructions.

```R
# Initial fit
fit_orig = fit_model( ...,
  build_model = FALSE )

# Extract default constructions
data_custom = fit_orig$data_list
map_custom = fit_orig$tmb_list$Map
parameters_custom = fit_orig$tmb_list$Parameters

# Modify inputs as needed
# [Insert stuff here]

# Pass and run model
fit_orig = fit_model( ..., 
  data = data_custom,
  Parameters = parameters_custom,
  Map = map_custom ) 
```

We here illustrate a few specific examples.

#### Modify the coordinates used when calculating center-of-gravity

[To be added by Alexa Fredston]