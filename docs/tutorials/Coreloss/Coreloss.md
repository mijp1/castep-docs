# Coreloss

## Calculation of core-loss spectra for cBN

Core-loss calculations effectively calculate the probability of an electron being excited from 1 state into the conduction band. This is useful for calculating the core-loss (ionisation edge) peaks - by performing these calculations, you can get an approximate simulation of what you would see in an experimental [EELS](https://en.wikipedia.org/wiki/Electron_energy_loss_spectroscopy) or [XANES](https://en.wikipedia.org/wiki/X-ray_absorption_near_edge_structure) spectrum.

In this tutorial, we will look at the results of such a calculation on cubic boron nitride (cBN). We will use the `cell` file

*cbn.cell*
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

*cbn.param*
```
task: spectral
spectral_task: coreloss
xc_functional: LDA
opt_strategy: speed
```

Note that, for now, the `species_pot` block doesn't change anything - the value inside is the same as the default pseudopotential you'd get. Run castep. After it is done, run Optados on cBN with the Optados input file

*cbn.odi*
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

Running Optados should generate 2 files of interest: `cbn_B1K1_core_edge.dat` and `cbn_N1K1_core_edge.dat` - these are the results of the core-loss calculations. We will focus on the first `dat` file - let's look at the boron part specifically. The file starts off like

```
-15.700479247265910        0.0000000000000000        8.6070320322269327E-005
-15.690478675557825        0.0000000000000000        8.6103943275722382E-005
```

The 1st column is the energy, the 2nd is the standard Gaussian-broadened core-loss and the 3rd column is the instrumentation broadened core-loss. Let's try plotting this with xmgrace - you could use `xmgrace -nxy cbn_B1K1_core_edge.dat`, but to easily add legends I'd use `xmgrace -batch plot.bat` on the batch file

<a id="plot_bat"></a>

*plot.bat*
```
READ BLOCK "cbn_B1K1_core_edge.dat"

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

This is done rather simply by specifying the missing electron when describing the potential in the `cell` file. If you look at the `cbn.castep` file generated earlier, you may see that the pseudopotential report contains the lines

```
"2|1.2|12|14|16|20:21(qc=8)"
```

This tells us what kind of pseudopotential is used for the boron. To specify that there is a 1s electron missing, all you have to do is add `{1s1.00}` at the end: with only 1 electron in the 1s shell, there is a core electron missing: a core hole.

Go into `cbn.cell`, and add the lines

```
%block species_pot
B 2|1.2|12|14|16|20:21(qc=8){1s1.00}
%endblock species_pot
```

to calculate the core edge data factoring in the missing 1s electron.
!!! note
    Your potential may be different, depending on your version of Castep etc. - but don't worry, as the procedure is the same.

Before we re-run Castep, add the line

`CHARGE : -1`

to the `cbn.param` file - this must be done to maintain charge neutrality.  Next, re-run Castep. Let's have a quick look at the pseudopotential report of boron in `cbn.castep`

```
============================================================                
| Pseudopotential Report - Date of generation 29-07-2024   |                
------------------------------------------------------------                
| Element: B Ionic charge:  4.00 Level of theory: LDA      |                
| Atomic Solver: Koelling-Harmon                           |                
|                                                          |                
|               Reference Electronic Structure             |                
|         Orbital         Occupation         Energy        |                
|            2s              2.000           -0.865        |                
|            2p              1.000           -0.654        |                
|                                                          |                
|                 Pseudopotential Definition               |                
|        Beta     l      e      Rc     scheme   norm       |                
|          1      0   -0.865   1.199     qc      0         |                
|          2      0    0.250   1.199     qc      0         |                
|          3      1   -0.654   1.199     qc      0         |                
|          4      1    0.250   1.199     qc      0         |                
|         loc     2    0.000   1.199     pn      0         |                
|                                                          |                
| Augmentation charge Rinner = 0.838                       |                
| Partial core correction Rc = 0.838                       |                
------------------------------------------------------------                
| "2|1.2|12|14|16|20:21(qc=8){1s1.00}"                     |                
------------------------------------------------------------                
|      Author: Chris J. Pickard, Cambridge University      |                
============================================================
```

You should notice that the energies in the 2s and 2p orbitals are lower, the beta values are all different, and, most importantly, that the pseudopotential used is the one we manually wrote in: the one with only 1 electron in the 1s shell.

Now, re-run Optados. This create the same files as before. Again, let's focus on the boron result. The output file `bn_B1K1_core_edge.dat` now starts like:

```
-16.879627404257498        0.0000000000000000        4.3418325679994708E-005
-16.869627177952179        0.0000000000000000        4.3433042280034070E-005
```

The columns represent the same information as before. Plotting it with the same batch file as [above](Coreloss.md#plot_bat) yields us this graph:

![1s missing core edge](1s_missing.png){width="50%"}

## Supercell

The periodic images of the core-hole will interact with one another.  As this is unphysical, we need to increase the distance between the core-holes. This is done by creating a supercell.  To do this, we will create a 2x2x2 supercell. There are multiple ways of doing this, but this tutorial will cover how it can be done using [Vesta](https://jp-minerals.org/vesta/en/). First, upload the `cell` file we used to Vesta. From the top of the toolbar, go into `Edit -> Edit Data -> Unit cell...`. This should open up the window below

![Edit unit cell window](cell_window.png)

Click `Transform...`. This opens up a new window

![Transformation window](transform_window.png)

To create the 2x2x2 supercell, the transformation matrix is rather simple: make the diagonal values 2 like in the figure above (so it becomes 2x larger in all directions) and click `Ok`. Select `Search atoms in the new unit-cell and add them as new sites` in the next pop-up window.

Now that the supercell has been generated, we must save it and turn it into a cell file. Click `File -> Export Data` and save it as  `cbn.cif` file (saving it as a `cell` file is not an option). We can use `cif2cell cbn.cif` to get information on how to make the new cell. We change `cbn.cell` to:

*cbn.cell*
```
%block lattice_abc
5 5 5
60 60 60
%endblock lattice_abc

%block positions_frac
B:exi       0.0000000   0.0000000   0.0000000
B       0.0000000   0.0000000   0.5000000
B       0.0000000   0.5000000   0.0000000
B       0.0000000   0.5000000   0.5000000
B       0.5000000   0.0000000   0.0000000
B       0.5000000   0.0000000   0.5000000
B       0.5000000   0.5000000   0.0000000
B       0.5000000   0.5000000   0.5000000
N       0.1250000   0.1250000   0.1250000
N       0.1250000   0.1250000   0.6250000
N       0.1250000   0.6250000   0.1250000
N       0.1250000   0.6250000   0.6250000
N       0.6250000   0.1250000   0.1250000
N       0.6250000   0.1250000   0.6250000
N       0.6250000   0.6250000   0.1250000
N       0.6250000   0.6250000   0.6250000
%endblock positions_frac

kpoints_mp_grid 6 6 6

symmetry_generate

%block species_pot
B:exi 2|1.2|12|14|16|20:21(qc=8){1s1.00}
%endblock species_pot

spectral_kpoint_mp_grid 5 5 5
```

With double the size of the supercell, you may also halve the kpoints: this allows it to be calculated faster without losing accuracy. However, it will still take significantly longer to calculate.

Specifying 1 of the boron atoms to be called `B:exi` and making changing the potential block to only affect that means that we simulate only 1 of the boron atoms losing that electron - by doing this we prevent the interaction problem mentioned above. Re-run Castep and Optados. There will now be 16 output files, rather than just 2 - there is a core edge output for every atom - `cbn_ B 1    K1     B:exi_core_edge.dat` is the core edge result for the boron with the missing 1s electron. The spaces in the file name can be a bit awkward so let's rename it to `cbn_BExi.dat`. Let's plot it on xmgrace, using the same method as before (include the lines `WORLD XMIN 10` and `WORLD XMAX 40` in the batch file to look at a more reasonable range). This is the output:

![Supercell 1s output](222_super.png)

## Comparison to Experiment

To compare properly to experiment, we will need to adjust the lifetime broadening; the supercell EELS results we just obtained are unrealistic as you cannot measure the spectrum so precisely. To account for that, we can add both Lorentzian and Gaussian broadening - add these lines to the `cbn.odi` file

```
LAI_LORENTZIAN_WIDTH : 0.5
LAI_LORENTZIAN_SCALE : 0.5
LAI_LORENTZIAN_OFFSET : 18

LAI_GAUSSIAN_WIDTH : 1
```

And plot the lifetime broadened result (we're not interested in the un-broadened result so remove the `1:2` block from the batch file) with xmgrace.

!!! note
    You can combine Gaussian and Lorentzian broadening, as is done above. Off-setting Lorentzian broadening can also be done to adjust the appearance of the graph - this is all done to try make it look more like experimental data. Feel free to tinker with the broadening.

The result should look like this:

![Attempt to make similar to experiment](broadened_output.png)

Compare this output with the one from the [EELS Database](https://eelsdb.eu/spectra/cubic-boron-nitride/). You should see that they look reasonably similar.

You may wish to plot the 2 results together. To do this, first download the `msa` file from the above link. Looking at the graph or the file, you can immediately see a problem - the energies are shifted (the region of interest starts at 190eV) and the intensities are much higher (order of thousands rather than 0.001). Therefore, we will use a script to shift and scale our results to be more like what you'd see in experiment.

We can do this by using a Python script

```py
with open('bn_ B 1    K1     B:exi_core_edge.dat', 'r') as infile, open('bn_BExi_ss.dat', 'w>
    for line in infile:
        columns = line.split()
        col1 = (float(columns[0]) * 3) + 140
        col5 = float(columns[2]) * 750000
        outfile.write(f"{col1} {col5}\n")

```

and then plotting the 2 sets of results together by using xmgrace on the batch file

```
READ BLOCK "bn_BExi_ss.dat"

BLOCK XY "1:2"
S0 LEGEND "Castep"

READ BLOCK "Dspec.60967.1.msa"

BLOCK XY "1:2"
S1 LEGEND "EELS Database"
```

The output should look like this:

![Experimental and Castep plotted together](together.png)

You can see that they are reasonably similar - the peaks are in similar positions. However, in a real real EELS spectrum, the spectrum doesn't fall to 0 but rather a finite value, unlike what was calculated here. 

Other things to try include:

* Include the core-hole on the N atom rather than the B
* Compare your simulated spectra to experimental data (the EELS database is a good place to find experimental data)
* Compare to spectra from cubic BN
* Calculating spectra from graphite (graphene) and diamond
