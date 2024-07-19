## Density of States

### Outline
This is a simple example of using `optados` for calculating electronic density of states of crystalline silicon in a 2 atom cell. It shows how optados's adaptive broadening can be used to resolve fine spectral features that a fixed broadening scheme will obscure.

The tarball for the tutorial files for all of the optados tutorials are here: [optados-examples.tgz](optados-examples.tgz)

### Input Files:
* `examples/Si2_DOS/Si2.cell` - The castep cell file containing information about the simulation cell.

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


* `examples/Si2_DOS/Si2.param` - The castep param file containing information about the parameters for the SCF and spectral calculations.

```
TASK                   : SPECTRAL
SPECTRAL_TASK          : DOS
PDOS_CALCULATE_WEIGHTS : TRUE
SPIN_POLARIZED         : TRUE
CUT_OFF_ENERGY         : 200
IPRINT                 : 1
```
* `examples/Si2_DOS/Si2.odi` - The optados input file, containing the parameters necessary to run optados.
```
# Choose the task to perform
TASK              : dos

# Recalculate the Fermi energy using the new DOS
# (discasrd the CASTEP efermi)
EFERMI : optados

# Sample the DOS at 0.1 eV intervals
DOS_SPACING       : 0.1

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

1. Perform a castep calculation on the bulk silicon using the  `Si2.cell`  and `Si2.param` input files.

	This should take a couple of seconds to run.

1. Examine the optados input file - `Si2.odi`

	Perform an optados calculation.

	```
	$ optados.mpi Si2
	```

	This generates 3 files:

	* `Si2.odo` -- optados general output file.
	* `Si2.adaptive.dat` -- The adaptive broadened DOS raw output data.
	* `Si2.adaptive.agr` -- The adaptive broadened DOS in a file suitable to be plotted by  xmgrace.

	Examine the `Si2.odo` file. `optados` has performed a Density of States calculation.

	```
	+--------------- Fermi Energy Analysis ------------------------+
	| From Adaptive broadening                                     |
	| Spin Component:1 occupation between 3.99961 and 4.00003 <-Oc |
	| Spin Component:2 occupation between 3.99961 and 4.00003 <-Oc |
	|       Fermi energy (Adaptive broadening) : 5.4109 eV  <- EfA |
	+--------------------------------------------------------------+
	```
	It has used the integrated DOS to work out the Fermi level, and has suggested the error in the integration by indicating the number of electrons at the Fermi level. Since we had 4 up electrons and 4 down in the input file this analysis seems satisfactory.

	```
	+----------------------- Electronic Data ---------------------+
	|  Number of Bands                           :       23       |
	|  Grid size                                 :  10 x 10 x 10  |
	|  Number of K-points                        :           110  |
	|  Spin-Polarised Calculation                :           True |
	|  Number of up-spin electrons               :          4.00  |
	|  Number of down-spin electrons             :          4.00  |
	+-------------------------------------------------------------+
	```
	Since we had `efermi : optados`, `optados` sets the internal value of the Fermi level to the one it has derived from the DOS. This is important for subsequent calculations. Other valid options are `file`, where `optados` uses the value calculated by the electronic structure code that generated the eigenvalues;  `insulator`, where optados uses a value calculated from assuming the system is non-metallic; or a value set by the user.

	optados now performs some analysis of the DOS at the Fermi level,

	```
	+-------------- DOS at Fermi Energy Analysis ----------------+
	|                          Fermi energy used :   5.4109 eV   |
	| From Adaptive broadening                                   |
	| Spin Component:1 DOS at Fermi Energy:0.0011 eln/cell <-DEA |
	| Spin Component:2 DOS at Fermi Energy:0.0011 eln/cell<- DEA |
	+------------------------------------------------------------+
	```
	From this we may assume that there is a band gap.

	Importantly, then optados calculates the band energy from the DOS is has calculated.

	```
	+------------------- Band Energy Analysis -------------------+
	|     Band energy (Adaptive broadening) : 1.3609 eV   <- BEA |
	|     Band energy (From CASTEP) :         1.3622 eV   <- BEC |
	+------------------------------------------------------------+
	```
	As the quality of the optados calculation is increased these two values should converge to the same answer.

	Finally optados shifts the Fermi level to 0 eV, for the output files.

1. The DOS is written to `Si2.adaptive.dat`. This contains 5 columns as described in the header of the file:

	```
	################################################################
	#
	#                  O p t a D O S   o u t p u t   f i l e
	#
	#    Density of States using adaptive broadening
	#  Generated on 12 Feb 2012 at 16:50:37
	# Column        Data
	#    1        Energy (eV)
	#    2        Up-spin DOS (electrons per eV)
	#    3        Down-spin DOS (electrons per eV)
	#    4        Up-spin Integrated DOS (electrons)
	#    5        Down-spin Integrated DOS (electrons)
	#
	################################################################
	```

	This file can be plotted by your favourite graph-plotting software. However, `optados` has made things easy and generated a  `Si2.adaptive.agr` file which is directly plottable using `xmgrace`.

	```
	$ xmgrace Si2.adaptive.agr
	```
1. We now try again with a better sampling of the DOS, by setting `DOS_SPACING : 0.001` and also analyse the band gap, by setting `COMPUTE_BAND_GAP : true`. You can set `IPRINT : 2` to see a progress report in  `Si2.odo`.

	In  `Si2.odo` we now have a new section analysing the band gap in various ways.

	```
	+----------------------------- Bandgap Analysis ---------------+
	|          Number of kpoints at       VBM       CBM            |
	|                   Spin :   1  :      1         1             |
	|                   Spin :   2  :      1         1             |
	|               Thermal Bandgap :   0.6676272107  eV    <- TBg |
	|            Between VBM kpoint :    0.05000    0.05000 0.05000|
	|               and CBM kpoint:   -0.45000    -0.05000 -0.45000|
	|             ==> Indirect Gap                                 |
	+--------------------------------------------------------------+
	|                            Optical Bandgap                   |
	|                   Spin :   1  :     2.5542517447  eV  <- OBg |
	|                   Spin :   2  :     2.5542463024  eV  <- OBg |
	|          Number of kpoints with this gap                     |
	|                   Spin :   1  :           1                  |
	|                   Spin :   2  :           1                  |
	+--------------------------------------------------------------+
	|                            Average Bandgap                   |
	|                   Spin :   1  :     3.8121372691  eV  <- ABg |
	|                   Spin :   2  :     3.8121342659  eV  <- ABg |
	|              Weighted Average :     3.8121357675  eV  <- wAB |
	+--------------------------------------------------------------+
	```

	`optados` is very careful in its band gap analysis. It uses the bare eigenvalues (un-broadened) and works out the nature and size of the thermal gap, optical gap and the average gap over all of the Brillouin zone. In cases of multi-valleyed semiconductors optados will report the number of conduction band minima or valence band maxima with identical energies, but will not report the nature of the gap.

	Increasing the number of integration points has improved the band energy of the adaptive smearing:

	```
	|      Band energy (Adaptive broadening) :  1.3623 eV   <- BEA |
	```

1. We will now compare the DOS with the adaptive broadening scheme with simple Gaussian smearing. In the optados input file (`Si2.odi`) change the value of `BROADENING` to `fixed`.

	Plotting the fixed broadened DOS over the adaptive we see the advantages of the adaptive broadening.

	```
	xmgrace Si2.adaptive.agr Si2.fixed.agr
	```
