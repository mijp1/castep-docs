## JDOS and OPTICS

### Joint DOS
See  `examples/Si2_JDOS`. This is a simple example of using `optados` for calculating joint electronic density of states.  We choose to recalculate the Fermi level using the calculated DOS, rather than use the Fermi level suggested by castep, and so `EFERMI: OPTADOS` is included in the `Si2.odi` file.  

### Input Files:
* `examples/Si2_JDOS/Si2.cell` - The castep cell file containing information about the simulation cell.

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


* `examples/Si2_JDOS/Si2.param` - The castep param file containing information about the parameters for the SCF and spectral calculations.

```
TASK                   : SPECTRAL
SPECTRAL_TASK          : DOS
PDOS_CALCULATE_WEIGHTS : TRUE
CUT_OFF_ENERGY         : 200
IPRINT                 : 1
```
* `examples/Si2_JDOS/Si2.odi` - The optados input file, containing the parameters necessary to run optados.
```
###########################################################
#       OptaDOS example file -- AJ Morris 18/V/11
###########################################################

# Choose the task to perform
TASK              : jdos

# Calculate the JDOS up to 30 eV
JDOS_MAX_ENERGY   : 30

# Sample the JOOS at 0.01 eV intervals
JDOS_SPACING      : 0.01

# Recalculate the Fermi energy using the new DOS
# (discasrd the CASTEP efermi)
EFERMI            : optados

# Since we're recalculating the Fermi energy we do
# a DOS calculation first.
# Sample the DOS at 0.1 eV intervals
DOS_SPACING       : 0.1

# Give progress reports as it goes
IPRINT : 2

###########################################################
#            A D V A N C E D   K E Y W O R D S
###########################################################

# The keywords below are all at their default value
# They are presented here to indicate the internal
# workings of OptaDOS and allow you to tweak the
# output

# The broadening used, (also try linear, or fixed)
BROADENING        : adaptive # Default

# The broadening parameter, A, when using adaptive smearing,
# set by eye to be similar to the linear smearing method
ADAPTIVE_SMEARING : 0.4      # Default

# The Gaussian broadening parameter for fixed smearing,
# in electron Volts
FIXED_SMEARING    : 0.3      # Default

# Set the Fermi energy to zero on the output plots
SET_EFERMI_ZERO : true       # Default

# Normalise the DOS with the volume of the simulation
# cell
DOS_PER_VOLUME  : false      # Default


###########################################################
#            C O M P A T I B I L I T Y
###########################################################

# Perform numerical integration of the DOS, instead of
# semi-analytic (useful to compare with LinDOS)
NUMERICAL_INTDOS      : false  # Default

# When performing numerical integration of the DOS make
# sure that no Gaussians are smaller than the dos_spacing.
# (Should always be true, but useful for comparison with
# LinDOS)
FINITE_BIN_CORRECTION : true  # Default
```

* Execute castep and optados using the example files.  The JDOS is written to `Si2.jadaptive.dat`. A file suitable for plotting using `xmgrace` is written to `Si2.jadaptive.agr`.
* Check the effect of changing the sampling by increasing and decreasing the value of `JDOS_SPACING` in the `Si2.odi` file.
