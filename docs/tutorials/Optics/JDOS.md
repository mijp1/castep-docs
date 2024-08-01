# JDOS

The JDOS (Joint Density of States) is effectively the density of states for an energy range which is equal to the difference between conduction and valence bands - this leads to being able to calculate the probability of an excitation occurring for a given photon energy. One can intuitively understand how this leads to the calculation of optical properties: photons that have energies that correspond to gaps between conduction and valence bands will interact with the material and give rise to its optical properties.

In this tutorial, we will calculate the JDOS for silicon, and then relate that to the imaginary and real part of the dielectric function.

## JDOS Calculation

Let's start by actually getting the JDOS results. Run Castep on the `cell` file

*Si.cell*
```
%BLOCK LATTICE_CART
2.73  2.73 0.00
2.73  0.00 2.73
0.00  2.73 2.73
%ENDBLOCK  LATTICE_CART

%BLOCK POSITIONS_FRAC
Si 0.0     0.0     0.0
Si 0.25    0.25    0.25
%ENDBLOCK POSITIONS_FRAC

SYMMETRY_GENERATE

KPOINTS_MP_GRID 10 10 10  
SPECTRAL_KPOINTS_MP_GRID 10 10 10
```

and `param` file

*Si.param*
```
TASK                   : SPECTRAL
SPECTRAL_TASK          : DOS
PDOS_CALCULATE_WEIGHTS : TRUE
CUT_OFF_ENERGY         : 200
```

Afterwards, run Optados with the Optados input file

*Si.odi*
```
TASK              : jdos

JDOS_MAX_ENERGY   : 30
JDOS_SPACING      : 0.01
EFERMI            : optados
DOS_SPACING       : 0.1
BROADENING        : adaptive # Default
ADAPTIVE_SMEARING : 0.4      # Default
FIXED_SMEARING    : 0.3      # Default
SET_EFERMI_ZERO : true       # Default
NUMERICAL_INTDOS      : false  # Default
FINITE_BIN_CORRECTION : true  # Default
```

The line `TASK : jdos` is what tells it to calculate the JDOS in which we are interested. This will generate the file `Si.jadaptive.dat` (as well as the accompanying `agr` file). The `dat` file data table should start like this

```
0.0000000000000E+00    0.3375085335870E-14
0.1000333444481E-01    0.3869643667676E-14
```

The 1st column is the energy, and the 2nd is the JDOS. The `agr` file allows us to make a nice plot with xmgrace (though  of course you could plot the results with your software of choice) - let's run

`xmgrace Si.jadaptive.agr`

 The result should look like this:

 ![JDOS Result](jdos_result.png){width="50%"}

### Joint DOS
See  `examples/Si2_JDOS`. This is a simple example of using `optados` for calculating joint electronic density of states.  We choose to recalculate the Fermi level using the calculated DOS, rather than use the Fermi level suggested by castep, and so `EFERMI: OPTADOS` is included in the `Si2.odi` file.  

* Execute castep and optados using the example files.  The JDOS is written to `Si2.jadaptive.dat`. A file suitable for plotting using `xmgrace` is written to `Si2.jadaptive.agr`.
* Check the effect of changing the sampling by increasing and decreasing the value of `JDOS_SPACING` in the `Si2.odi` file.
