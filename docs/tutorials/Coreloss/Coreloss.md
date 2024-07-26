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

kpoints_mp_grid 2 2 2

symmetry_generate

%block species_pot
B 2|1.2|12|14|16|20:21(qc=8)
%endblock species_pot

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
-15.685787277124911        0.0000000000000000        4.3994742417499804E-005
-15.675786930280168        0.0000000000000000        4.4820011439245385E-005
```

The 1st column is the energy, the 2nd is the standard Gaussian-broadened core-loss and the 3rd column is the instrumentation broadened core-loss. Let's try plotting this with xmgrace - you could use `xmgrace -nxy hbn_B1K1_core_edge.dat`, but to easily add legends I'd use `xmgrace -batch plot.bat` on the batch file

*plot.dat*
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

```
LAI_LORENTZIAN_WIDTH : 0.5
LAI_LORENTZIAN_SCALE : 0.1
```

Now try running CASTEP and OptaDOS to produce a N K-edge

### Including a core-hole

First we will include a core-hole on atom B:1.  To do this we add a modified pseudopotential string into the hBN.cell file

```
B:1  2|1.2|12|14|16|20:21{1s1.00}(qc=8)
```

The core-hole is created by removing a 1s electron from the electronic configuration used in the generation of the pseudopotential.  Information about the pseudopotential is included at the top of the `hBN.castep` file

```
   ============================================================
   | Pseudopotential Report - Date of generation 17-08-2018   |
   ------------------------------------------------------------
   | Element: B Ionic charge:  3.00 Level of theory: PBE      |
   | Atomic Solver: Koelling-Harmon                           |
   |                                                          |
   |               Reference Electronic Structure             |
   |         Orbital         Occupation         Energy        |
   |            2s              2.000           -0.347        |
   |            2p              1.000           -0.133        |
   |                                                          |
   |                 Pseudopotential Definition               |
   |        Beta     l      e      Rc     scheme   norm       |
   |          1      0   -0.347   1.199     qc      0         |
   |          2      0    0.250   1.199     qc      0         |
   |          3      1   -0.133   1.199     qc      0         |
   |          4      1    0.250   1.199     qc      0         |
   |         loc     2    0.000   1.199     pn      0         |
   |                                                          |
   | Augmentation charge Rinner = 0.838                       |
   | Partial core correction Rc = 0.838                       |
   ------------------------------------------------------------
   | "2|1.2|12|14|16|20:21(qc=8)"                             |
   ------------------------------------------------------------
   |      Author: Chris J. Pickard, Cambridge University      |
   ============================================================
```

The line

```
   | "2|1.2|12|14|16|20:21(qc=8)"                             |
```

specifies the parameters used to create the OFT B pseudopotential. We use this as a starting point and remove one of the core 1s electrons to create the core-hole pseudopotential by including `{1s1.00}`.  

To maintain the neutrality of the cell, we include

`CHARGE : +1`

in the `hBN.param` file.  Run the calculation.  Compare the K-edge from the core-hole calculation with the previous non-core-hole calculation.  

The periodic images of the core-hole will interact with one another.  As this is unphysical, we need to increase the distance between the core-holes. This is done by creating a supercell.  Create a 2x2x1 supercell (talk to one of the tutors if you’re unsure about how to do this) and carry out another core-loss B K-edge simulation.  Compare the spectra to that from the primitive cell.  Construct larger and larger unit cells until the spectrum stops changing with increasing separation between the periodic images.  

Other things to try include:

* Include the core-hole on the N atom rather than the B
* Compare your simulated spectra to experimental data (the EELS database is a good place to find experimental data)
* Compare to spectra from cubic BN
* Calculating spectra from graphite (graphene) and diamond


We begin by running a CASTEP calculation using the files provided.  Note that we specify a pseudopotential file for one B atom and both N atom, and use an on-the-fly pseudopotential for the other B atom.  This looks a bit weird!  It is simply a way to only compute the EELS for one atomic site (core-loss spectra can only be computed for atoms described by on-the-fly potentials). An h-BN [cell](h-BN.cell) and [param](h-BN.param) file are provided.
