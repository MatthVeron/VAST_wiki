# Speed test and profiling VAST

## Speed testing
When developing new features, I try to check how changes affect speed for estimating parameters and conducting bias-correction.  Below, I show code for doing this using the simplified user-interface (in this case using the AFSC virtual machine):

```R
# Download release number 3.0.0; its useful for reproducibility to use a specific release number
devtools::install_github("james-thorson/VAST", ref="3.0.0")

# Set local working directory (change for your machine)
setwd( "C:/Users/james.thorson.vm/Desktop/Speed_test" )

# Load packages
library(TMB)
library(VAST)

# load data set
# see `?load_example` for list of stocks with example data
# that are installed automatically with `FishStatsUtils`.
example = load_example( data_set="EBS_pollock" )

# Make settings
settings = make_settings( n_x=50, Region=example$Region, purpose="index",
  strata.limits=example$strata.limits )

# Simulation settings
n_replicates = 5
nx_set = c(25, 50, 100, 200, 400)
version_set = c("VAST_v7_0_0", "VAST_v8_0_0", "VAST_v8_0_0")
fine_set = c(FALSE, FALSE, TRUE)
biascorrect_set = c(FALSE, TRUE)
Speed_rxcb = array( NA, dim=c(n_replicates,length(nx_set),length(version_set),length(biascorrect_set)) )
dimnames(Speed_rxcb) = list( 1:n_replicates,paste0("nx=",nx_set), paste(version_set,"-fine=",fine_set,sep=""),
  paste0("biascorr=",biascorrect_set) )

# Loop
for( rI in 1:dim(Speed_rxcb)[1] ){
for( xI in 1:dim(Speed_rxcb)[2] ){
for( cI in 1:dim(Speed_rxcb)[3] ){
for( bI in 1:dim(Speed_rxcb)[4] ){
  # Set seed to conserve across xI and cI, and across runs of the experiment
  set.seed(rI)

  # Settings for each configuration cI
  settings$Version = version_set[cI]
  settings$fine_scale = fine_set[cI]
  settings$n_x = nx_set[xI]
  settings$bias.correct = biascorrect_set[bI]

  # Run model
  fit = fit_model( "settings"=settings, "Lat_i"=example$sampling_data[,'Lat'],
    "Lon_i"=example$sampling_data[,'Lon'], "t_i"=example$sampling_data[,'Year'],
    "c_i"=rep(0,nrow(example$sampling_data)), "b_i"=example$sampling_data[,'Catch_KG'],
    "a_i"=example$sampling_data[,'AreaSwept_km2'], "v_i"=example$sampling_data[,'Vessel'] )

  # Record run-time
  Speed_rxcb[rI,xI,cI,bI] = as.double( fit$parameter_estimates$time_for_run, units="mins")

  # Save
  save( Speed_rxcb, file="Speed_rxcb.RData" )
}}}}

apply( Speed_rxcb, MARGIN=2:4, FUN=mean, na.rm=TRUE )
```

This exercise shows that V8.0.0 is only 10-20% slower than previous versions when using `fine_scale=FALSE` and perhaps 50% slower when using `fine_scale=TRUE` without bias-correction.  However, using `fine_scale=TRUE` and bias-correction, it is more like 5-10 fold slower than previous versions.


## Profiling
[Profling](https://en.wikipedia.org/wiki/Profiling_(computer_programming))
of computer code tells you about the usage of memory and CPU resources
during its execution, giving insight into which aspects of the code could
be made more efficient. Here I adapt the profile tools set up for TMB
models
[here](https://github.com/kaskr/adcomp/tree/master/tmb_examples#profiling)
and show how to do it with a VAST model.

The first step is to download and install the Intel vtune tool
[here](https://software.intel.com/en-us/vtune/choose-download#standalone). Then
add the executable folder to your environmental path, which on my machine
is `C:\Program Files (x86)\IntelSWTools\VTune Amplifier 2019\bin64`. Check
it is in your path by opening a command window and running `where
amplxe-cl` and it should point to the path.

Now clone adcomp from github and open a terminal window in
adcomp/tmb_examples. From the terminal run `make spatial.profile`. You will
see the spatial R example output running in the console. If you get errors
try (1) editing the tmb_exampes/tools/profile.R file so line 9 is `Rexe <-
"R"` (on Win10). Try rerunning. If you still get errors, try opening the
command window as an administrator (on Win10 click start menu, search for
'cmd' then right click and 'Run as Administrator'). A successful profiling
event produes a folder spatial.profile in the tmb_examples folder. In the
console type `amplxe-gui spatial.profile`. This will launch the visual
profiling tool.

Now to get it to work with VAST, you need to trick TMB into thinking VAST
is one of the examples. First create an empty file 'VAST.cpp' in the
tmb_examples folder. Then create the R script 'VAST.R' and put in code
that will run VAST. E.g., the simple example on the wiki. To make it run
faster try chopping off some years like:
```
example$sampling_data  <- subset(example$sampling_data, Year > 2010)
```
Run the example in R to compile the VAST cpp and make sure it runs
fine. Then, go back to the console and run 'make VAST.profile'. It should
start the profiler and run the VAST code. 

Now you can visualize the profile with the command `amplxe-gui
VAST.profile`. Copy the VAST.profile folder and you can now change the
script VAST.R to match your situation, and rerun the profile as above. This
lets you profile different combinations and configurations of VAST. For
instance you can try profiling with and without spatiotemporal effects, or
with inference via MLE vs tmbstan.

Refer to the help files for Vtune for how to interpret the output. 
