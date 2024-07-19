## Calculation of core-loss spectra for hBN

We begin by running a CASTEP calculation using the files provided.  Note that we specify a pseudopotential file for one B atom and both N atom, and use an on-the-fly pseudopotential for the other B atom.  This looks a bit weird!  It is simply a way to only compute the EELS for one atomic site (core-loss spectra can only be computed for atoms described by on-the-fly potentials). An h-BN [cell](h-BN.cell) and [param](h-BN.param) file are provided.

Execute optados using the optados input file provided and the file `hBN_core_edge.dat` will be created. The file contains two columns, the first is the energy and the second is the spectrum. This file contains the following edges:

```
B 1    K1       B:1
```

OptaDOS has also written a grace file

```
$ xmgrace hBN_core_edge.agr
```

This spectrum has lots of fine detail.  To compare with experiment we can include lifetime and instrument broadening effects. First let's add some Gaussian broadening to simulate instrument effects. Add the following the odi file and re-run.

```
CORE_LAI_BROADENING : true
LAI_GAUSSIAN_WIDTH : 1.0
```

The file `hBN_core_edge.dat` now has three columns. The third column is the broadened spectrum. To compare the broadened and un-broadened spectra

```
xmgrace hBN_core_edge_broad.agr hBN_core_edge.agr
```

Now add some lifetime broadening (look up the meaning of the keywords in the optados user guide)

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
