# Coreloss

## Calculation of core-loss spectra for hBN

Core-loss calculations effectively calculate the probability of an electron being excited from 1 state into the conduction band. This is useful for calculating the core-loss (ionisation edge) peaks - by performing these calculations, you can get an approximate simulation of what you would see in an experimental [EELS](https://en.wikipedia.org/wiki/Electron_energy_loss_spectroscopy) or [XANES](https://en.wikipedia.org/wiki/X-ray_absorption_near_edge_structure) spectrum.

In this tutorial, we will look at the results of such a calculation on hexagonal boron nitride (h-BN). We will use the `cell` file

*hbn.cell*
```
%block lattice_abc
2.5 2.5 2.5
60 60 60
%endblock lattice_abc

%block positions_frac
B 0.00 0.00 0.00
N 0.25 0.25 0.25
%endblock positions_frac

kpoints_mp_grid 12 12 12

symmetry_generate


spectral_kpoint_mp_grid 10 10 10
```

with the `param` file

*hbn.param*
```
task: spectral
spectral_task: coreloss
xc_functional: LDA
opt_strategy: speed
```

Note that, for now, the `species_pot` block doesn't change anything - the value inside is the same as the default pseudopotential you'd get. Run castep. After it is done, run Optados on hbn with the Optados input file

*hbn.odi*
```
TASK               : core
DOS_SPACING       : 0.01
BROADENING         : adaptive # Default
ADAPTIVE_SMEARING  : 0.4  # Default
CORE_GEOM          : polycrystalline  # Default
CORE_LAI_BROADENING : true  # Default
LAI_GAUSSIAN_WIDTH : 1.0
```

The line `TASK : core` is what determines that a core-loss calculation will be performed. `CORE_GEOM : polycrystalline` could also be replaced by a different value, such as `TENSOR`, to get other results: here we are finding the results of the polycrystalline (uniform) case. To compare with experiment we can include lifetime and instrument broadening effects - this is done by the lines

```
CORE_LAI_BROADENING : true  # Default
LAI_GAUSSIAN_WIDTH : 1.0
```

which adds some Gaussian broadening to simulate instrument effects.

Running Optados should generate 2 files of interest: `hbn_B1K1_core_edge.dat` and `bn_N1K1_core_edge.dat` - these are the results of the core-loss calculations. We will focus on the first `dat` file - let's look at the boron part specifically. The file starts off like

```
-15.700479247265910        0.0000000000000000        8.6070320322269327E-005
-15.690478675557825        0.0000000000000000        8.6103943275722382E-005
```

The 1st column is the energy, the 2nd is the standard Gaussian-broadened core-loss and the 3rd column is the instrumentation broadened core-loss. Let's try plotting this with xmgrace - you could use `xmgrace -nxy hbn_B1K1_core_edge.dat`, but to easily add legends I'd use `xmgrace -batch plot.bat` on the batch file

*plot.bat*
```
READ BLOCK "bn_B1K1_core_edge.dat"

BLOCK XY "1:2"
S0 LEGEND "No Instrumentation Broadening"

BLOCK XY "1:3"
S1 LEGEND "Instrumentation Broadened"

WORLD XMIN 0
```

This creates a graph that looks like:

![Result of above steps](basic_result.png){width="50%"}

While the instrumentation broadening does make the results more similar to what you'd expect in experiment, it also leads to information being lost from the graph (for example, the peak right after 40eV is completely missing) and the instruments used in practice are likely more realistic than that: let's try adding to the `odi` file

```
LAI_LORENTZIAN_WIDTH : 1
LAI_LORENTZIAN_SCALE : 0.01
```

and rerunning Optados. Plotting the results again yields this graph:

![Less instrumentation broadening](less_lai_broaden.png){width="50%"}

In practice, both the instrumentation and adaptive (or linear of fixed) broadening are adjusted until the results are close to experiment.

### Including a core-hole

The above was effectively calculating the probability of an electron being able to be excited into the conduction band, corresponding to that same energy being lost from an X-ray/electron and thus XANES/EELs data. However, when calculating that, it was not accounting for the fact that there'd be a core-hole as a result (which naturally will affect energy, DOS and thus probability of occuring): that must be factored in for more realistic results.

This is done rather simply by specifying the missing electron when describing the potential in the `cell` file. If you look at the `hbn.castep` file generated earlier, you may see that the pseudopotential report contains the lines

```
"2|1.2|12|14|16|20:21(qc=8)"
```

This tells us what kind of pseudopotential is used for the boron. To specify that there is a 1s electron missing, all you have to do is add `{1s1.00}` at the end: with only 1 electron in the 1s shell, there is a core electron missing: a core hole.

Go into `hbn.cell`, and add the lines

```
%block species_pot
B 2|1.2|12|14|16|20:21(qc=8){1s1.00}
%endblock species_pot
```

to calculate the core edge data factoring in the missing 1s electron.
!!! note
    Your potential may be different, depending on your version of Castep etc. - but don't worry as the procedure is the same.

Before we re-run Castep, add the line

`CHARGE : +1`

in the `hBN.param` file - this must be done to maintain charge neutrality.  Next, re-run castep and Optados. 

The periodic images of the core-hole will interact with one another.  As this is unphysical, we need to increase the distance between the core-holes. This is done by creating a supercell.  Create a 2x2x1 supercell (talk to one of the tutors if you’re unsure about how to do this) and carry out another core-loss B K-edge simulation.  Compare the spectra to that from the primitive cell.  Construct larger and larger unit cells until the spectrum stops changing with increasing separation between the periodic images.  

Other things to try include:

* Include the core-hole on the N atom rather than the B
* Compare your simulated spectra to experimental data (the EELS database is a good place to find experimental data)
* Compare to spectra from cubic BN
* Calculating spectra from graphite (graphene) and diamond


We begin by running a CASTEP calculation using the files provided.  Note that we specify a pseudopotential file for one B atom and both N atom, and use an on-the-fly pseudopotential for the other B atom.  This looks a bit weird!  It is simply a way to only compute the EELS for one atomic site (core-loss spectra can only be computed for atoms described by on-the-fly potentials). An h-BN [cell](h-BN.cell) and [param](h-BN.param) file are provided.
