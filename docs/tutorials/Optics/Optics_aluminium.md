# Optics - Aluminium

In this tutorial, we will have brief look at how to calculate the optical properties of metals, looking at aluminium in particular, and how the results are different from a standard Optados optics calculation. It is highly recommended that you first go through the [previous tutorial](Optics.md), where we go more into depth on it - this tutorial is mostly just highlighting how it is slightly different for metals.

We will use the `cell` file

*Al.cell*
```
%BLOCK LATTICE_CART
       4.050000000000000       0.000000000000000       0.000000000000000
       0.000000000000000       4.050000000000000       0.000000000000000
       0.000000000000000       0.000000000000000       4.050000000000000
%ENDBLOCK LATTICE_CART

%BLOCK POSITIONS_FRAC
  Al   0.0000000000000000   0.0000000000000000   0.0000000000000000
  Al   0.5000000000000000   0.5000000000000000   0.0000000000000000
  Al   0.5000000000000000   0.0000000000000000   0.5000000000000000
  Al   0.0000000000000000   0.5000000000000000   0.5000000000000000
%ENDBLOCK POSITIONS_FRAC

KPOINTS_MP_GRID 15 15 15
SPECTRAL_KPOINTS_MP_GRID 15 15 15

SYMMETRY_GENERATE
```

and `param` file

*Al.param*
```
TASK                   : SPECTRAL
SPECTRAL_TASK          : optics
NEXTRA_BANDS           : 10
```

Run Castep as usual. Then run Optados with the Optados input file

*Al.odi*
```
TASK               : optics

# Sample the JDOS at 0.01 eV intervals
JDOS_SPACING       : 10

# Calculate the JDOS up to 60eV about the valence band maximum
JDOS_MAX_ENERGY    : 30

# Include the intraband term in the calculation of the
# dielectric function
OPTICS_INTRABAND   : true

# Recalculate the Fermi energy using the new DOS
# (discasrd the CASTEP efermi)
EFERMI             : optados

# Since we're recalculating the Fermi energy we do
# a DOS calculation first.
# Sample the DOS at 0.1 eV intervals
DOS_SPACING        : 0.1

# The broadening parameter, A, when using adaptive smearing,
# set by eye to be similar to the linear smearing method
ADAPTIVE_SMEARING  : 0.8  

# The broadening used, (also try linear, or fixed)
BROADENING         : adaptive # Default

# Specify the geometry to be used in the optics calculation
OPTICS_GEOM        : polycrystalline     # Default
```
There is 1 key difference to before: it contains the line `OPTICS_INTRABAND : true`. This is to include the intraband contribution, which is necessary for metals.

We're starting with the high spacing so we get few results (measuring only at 3 energies) - this makes it easier to have a quick look at what's going on.

The file `Al_epsilon.dat` contains the following data:

```
0.0000000000000000        2.1445746376579851        0.0000000000000000     
  15.000000000000000        1.1673581523361172        1.6007294583641971     
  30.000000000000000       0.66031465886470098       0.39432835642313052     


  0.0000000000000000       -32490.319172606654                            NaN
  15.000000000000000       0.37438389209757406        2.7452531094084771E-003
  30.000000000000000       0.84359371433743857        3.4316159432482577E-004


  0.0000000000000000       -32489.174597968995                            NaN
  15.000000000000000       0.54174204443369112        1.6034747114736059     
  30.000000000000000       0.50390837320213944       0.39467151801745531     
```

You can see here that there are 3 columns like before (energy, real and imaginary dielectric), but there's 3 separate sets of them, separated by a double space. The 1st set corresponds to the interband contribution, the 2nd to the intraband, and the 3rd to the total. Though normally it'd be easiest to visualise this data by plotting the `agr` file on xmgrace, this would only give the interband term - if you're interested in other information you will have to use the `.dat` file.

You may choose to use/plot this data in your preferred method, but this tutorial will give you the necessary files to plot it using xmgrace. Firstly, we will create a new file called `Al_epsilon_sep.dat` file that turns separates the data into columns - we can do that with [this Python script](contrubtions_sep.py). The output file looks like

```
0.0000000000000000           2.1445746376579851           0.0000000000000000      -32490.3191726066543197           0.0000000000000000      -32489.1745979689949309           0.0000000000000000
15.0000000000000000           1.1673581523361172           1.6007294583641971           0.3743838920975741           0.0027452531094085           0.5417420444336911           1.6034747114736059
30.0000000000000000           0.6603146588647010           0.3943283564231305           0.8435937143374386           0.0003431615943248           0.5039083732021394           0.3946715180174553
```

It's exactly the same data except all the data is in separate columns now: 1 is still energy, 2 is interband real, 3 is interband imaginary, 4 is intraband real, 5 is intraband imaginary, 6 is total real and 7 is total imaginary dielectric.

Now let's make a graph of more useful data: rerun Optados with `JDOS_SPACING : 0.01` set instead (to get more data for more meaningful graphs), and rerun the Python script. You could plot it with xmgrace (and using any accompanying batch files as you wish), but it is more convenient to plot it using plotly (and Bokeh to add a bit more functionality).



Here we see that, above 1.5eV, the imaginary total dielectric is dominated by the interband contribution, while the real part appears to be influenced slightly moreso by the intraband term. By adjusting the axis limits, or looking at the `Al_epsilon_sep.dat` file

```
0.0000000000000000         111.3599917818530116           0.0000000000000000      -32490.3191726066543197           0.0000000000000000      -32379.9591808248005691           0.0000000000000000
0.0100033344448149         101.2911635864703186          21.4321242322942318      -31756.8048547613216215      208963.9725235598161817      -31656.5136911748522834      208985.4046477921074256
0.0200066688896299          92.9566193519629564          21.9702759743209981      -29742.3723795367077400       97854.5790097907593008      -29650.4157601847437036       97876.5492857650679071
```

we see that the intraband contribution makes the dielectric function at low eV's extremely large, while the interband has little effect.

Running Optados generates the same files as without its contribution, except some of the files are slightly different different.



* The `Al_OPTICS_epsilon.dat` file has the same format as before, but it now contains sequentially the interband contribution, the intraband contribution and the total dielectric function.  The file `Al_OPTICS_epsilon.agr` only contains the interband term.  In the same way, `Al_OPTICS_loss_fn.dat` contains the interband contribution, intraband contribution and total loss function.  All other optical properties are calculated from the total dielectric function and the format of the output files remains the same.

* In the case where the dielectric tensor is calculated and the intraband term is included, only the `Al_OPTICS_epsilon.dat` file is generated.  As before it contains each component, but this time it lists sequentially the interband contribution, intraband contribution and total dielectric function for each component.   

* This time, if additional broadening for the loss function is included by using the key word `optics_lossfn_broadening`,  `AL_OPTICS_loss_fn.dat` will contains four sequential data sets.  These are the interband contribution, the intraband contribution, the total loss function without the additional broadening and the broadened total loss function.  
