# Demo 1:  Getting standard cosmological theory functions for a single cosmology #

This demo will calculate standard cosmological distances, and matter and cmb power spectra using the cosmological parameters given in the values1.ini file.
Original file: https://bitbucket.org/joezuntz/cosmosis/wiki/Demo1

## Running ##

After installing CosmoSIS, go to your cosmosis directory and set up the cosmosis environment if you haven't already:

```
#!bash
cd <path to cosmosis>
source config/setup-cosmosis
```

then run demo 1
```
cosmosis demos/demo1.ini
```

You should see some output from CAMB.  A directory will now exist called demo_output_1, containing basic cosmology data, like CMB spectra, matter power, and cosmological parameters.  You can make plots of them. To do this you need to have the matplotlib package installed. This is automatically installed by the CosmoSIS auto-installer, so you can run from the cosmosis source directory: 

```
#!bash

postprocess demos/demo1.ini -o plots -p demo1

```
(If you have installed matplotlib yourself then open a new shell_session and use your own version.)
If this doesn't work and you have installed R, you can instead run:
```
#!bash

 ./cosmosis/plotting/cosmology_theory_plots.r demo_output_1 -o plots -p demo1

```

You will now have a collection of .png plots in the plots directory showing CMB and matter power spectra, and distances as a function of redshift:


```
#!bash

> ls plots
README
demo1_angular_distance.png
demo1_bb.png
demo1_comoving_distance.png
demo1_distance_modulus.png
demo1_ee.png
demo1_grand.png
demo1_hubble.png
demo1_luminosity_distance.png
demo1_matter_power.png
demo1_te.png
demo1_tt.png

```

(Some of the filenames are different if you have used the _R_ script.)

For example, the demo1_matter_power.png should look like this:
![demo1_matter_power.png](https://bitbucket.org/repo/KdA86K/images/390541202-demo1_matter_power.png)

## Understanding ##

Have a look at the file demos/demo1.ini.

It starts with the *runtime* option:

```
[runtime]
; The test sampler just runs a single parameter set
sampler = test
```

This tells cosmoSIS what to do after, as we will see, the different modules have run.
As the name of the option suggest the results of the modules can be passed to an actual sampler like PyMC or emcee. 
In this demo we just want to test our pipeline so we choose the simpler sampler, *sampler = test*.

The next thing to do is to set the options of our sampler. 
When we will use actual sampler we will see more interesting options. Right know our sampler test just receive the results from the modules and save them. 
So we just need to tell it where to do that with:
```
[test]
; These are the parameters for this sampler.
; In this case there is just one parameter
save_dir=demo_output_1
```




As we mentioned earlier CosmoSIS is based on *modules*.  Each module is a single piece of code that does a calculation, taking some inputs from previous modules and saving some outputs for later.

The list of modules is shown in the **pipeline** section:

```
#!ini

[pipeline]
; The list of modules to be run, in this order.
; The modules named here must appear as sections below
modules = consistency camb halofit

```

We first ran "consistency", a handy module for the start of pipelines that generates the simpler derived parameters, like omega_c from omega_m and omega_b.

We then ran a CAMB module, to do the main cosmology calculations, and a halofit module, to compute non-linear power.  In later demos we will see some other [CosmoSIS modules](modules) we can add here. 

The inputs to the pipeline are some parameters to specify the cosmology.  Those parameters are in the file given by this part of the pipeline section:

```
#!ini

; The file to get cosmological and nuisance parameters
; from.
values = demos/values1.ini

```

This tells cosmosis about another file where the parameters are found.  
**Have a look** in the file *demos/values1.ini*.

Parameters and data in CosmoSIS are organized into sections.  There is only one section in this case, called cosmological_parameters.  And the parameters are specified here, for example like this:

```
#!ini

omega_m = 0.3
h0 = 0.72
omega_b = 0.04

```

At the moment we are just specifying a single value for each parameter.  Later we will see how to set a range of parameters to be sampled over.

Internally, the CAMB module asked for the values of the parameters that it needed, and they were found from here.  CAMB then computed some things and added them to the collection of saved data, so that the next module, halofit, could look at them too.

Halofit read its input parameters and data and added its own new data, the non-linear power spectrum.

The resulting data are passed to the sampler, in this case test, that just save them.
