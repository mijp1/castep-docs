## Projected Density of States

We assume the reader is familiar with the previous section on Density of States calculations and is now familiar with running `optados`.

### Outline
This is a simple example of using optados for calculating electronic density of states of 2 atoms of crystalline silicon projected onto LCAO basis states.

### Input Files
* `examples/Si2_PDOS/Si2.cell` - The castep `.cell` file containing information about the simulation cell.

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
SPECTRAL_KPOINTS_MP_GRID 2 2 2
```

* `examples/Si2_PDOS/Si2.param` - The castep `.param` file containing information about the parameters for the SCF and spectral calculations.

```
TASK                   : SPECTRAL
%BLOCK devel_code
spectral: check_output: true :endspectral
%ENDBLOCK devel_code

SPECTRAL_TASK          : DOS
PDOS_CALCULATE_WEIGHTS : TRUE
CUT_OFF_ENERGY         : 200
IPRINT                 : 1
```

* `examples/Si2_PDOS/Si2.odi` - The optados input file, containing the parameters necessary to run optados.
```
###########################################################
#       OptaDOS example file -- AJ Morris 18/V/11
###########################################################

TASK              : pdos

# Decompose into angular momentum channels
# (also try species_ang, species, sites)
PDOS : angular

# Or choose the projectors by hand...

# The DOS on Si atom 1 and the DOS on the s-channel
# of Si atom 2 (2 proj)
#PDOS : Si1;Si2(s)
# The sum of the s-channels on the two Si atoms (1 proj)
#PDOS : sum:Si1-2(s)
# The p-channel on each Si atom 1. (1 proj)
#PDOS : Si1(p)

# Recalculate the Fermi energy using the new DOS
# (discasrd the CASTEP efermi)
EFERMI            : optados

# Sample the DOS at 0.1 eV intervals
DOS_SPACING       : 0.1

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
### Instructions:

* Examine the optados input file noting `TASK : pdos`. We choose to decompose the DOS into angular momentum channels `PDOS : angular` and as in the previous example we choose to recalculate the Fermi level using the calculated DOS, rather than use the Fermi level suggested by castep.

* Execute optados.

* The output can be found in `Si2.pdos.dat`.

	```
	################################################################
	#
	#                  O p t a D O S   o u t p u t   f i l e
	#
	#  Generated on 13 Feb 2012 at 10:15:10
	################################################################
	#+-------------------------------------------------------------+
	#|                    Partial Density of States -- Projectors  |
	#+-------------------------------------------------------------+
	#| Projector:    1 contains:                                   |
	#|           Atom            AngM Channel                      |
	#|          Si   1                 s                           |
	#|          Si   2                 s                           |
	#+-------------------------------------------------------------+
	#| Projector:    2 contains:                                   |
	#|           Atom            AngM Channel                      |
	#|          Si   1                 p                           |
	#|          Si   2                 p                           |
	#+-------------------------------------------------------------+
	#| Projector:    3 contains:                                   |
	#|           Atom            AngM Channel                      |
	#|          Si   1                 d                           |
	#|          Si   2                 d                           |
	#+-------------------------------------------------------------+
	#| Projector:    4 contains:                                   |
	#|           Atom            AngM Channel                      |
	#|          Si   1                 f                           |
	#|          Si   2                 f                           |
	#+-------------------------------------------------------------+
	```
	The header shows that there are four projectors described below. The first containing the s-channels of both silicon atoms, the second the p-channels etc.

* The output is easily plotted using `xmgrace`.

* Setting `DOS_SPACING : 0.001` gives a high quality plot, as shown in the figure below.

	| ![Si DOS](opt1.png) |
	|:--:|
	| <b>Density of States of Silicon generated by adaptive broadening projected onto LCAO momentum states</b>|

* Other projections to try are:
	* `PDOS : Si1;Si2(s)`  -- Output the PDOS on Si atom 1 and the PDOS on the s-channel of Si atom 2. (Resulting in two projectors)
	* `PDOS : sum:Si1-2(s)`  --  Output the sum of the s-channels on the two Si atoms. (Resulting in one projector)
	* `PDOS : Si1(p)` -- Output the p-channel on Si atom 1. (Resulting in one projector)
