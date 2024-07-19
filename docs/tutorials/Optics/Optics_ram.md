# Optics

Optados is capable of getting numerous optical properties of different structures, all of which depend on the wavelength (energy) of light interacting with it. In this tutorial, we will perform an Optados optics calculation on rutile (TiO~2~) - a birefringent crystal with anisotropic optical properties, examining its single-crystal and polycrystalline (isotropic) properties.

We will use the cell file

*rut.cell*

```
%BLOCK LATTICE_ABC
  4.6257000   4.6257000   2.9806000
 90.0000000  90.0000000  90.0000000
%ENDBLOCK LATTICE_ABC
%BLOCK POSITIONS_FRAC

Ti      0.0000000   0.0000000   0.0000000
O       0.2821000   0.2821000   0.0000000

%ENDBLOCK POSITIONS_FRAC

SYMMETRY_GENERATE

KPOINTS_MP_GRID 10 10 10
SPECTRAL_KPOINTS_MP_GRID 14 14 14
```

This `cell` file was obtained using cif2cell using the structure with the COD ID 1010942, found on the [Crystallography Open Database](www.crystallography.net)

We will first run a castep calculation using the above cell with the `param` file

*rut.param*

```
TASK                   : SPECTRAL
SPECTRAL_TASK          : OPTICS
CUT_OFF_ENERGY         : 200
```

Once that is done, we can perform the optical Optados calculation. This is done by running Optados calculation with the Optados input file

*rut.odi*

```
TASK               : optics

# Sample the JDOS at 0.01 eV intervals
JDOS_SPACING       : 0.01

# Calculate the JDOS up to 60eV about the valence band maximum
JDOS_MAX_ENERGY    : 60

# Recalculate the Fermi energy using the new DOS
# (discasrd the CASTEP efermi)
EFERMI             : optados

# Since we're recalculating the Fermi energy we do
# a DOS calculation first.
# Sample the DOS at 0.1 eV intervals
DOS_SPACING        : 0.1

# The broadening used, (also try linear, or fixed)
BROADENING         : adaptive # Default

# The broadening parameter, A, when using adaptive smearing,
# set by eye to be similar to the linear smearing method
ADAPTIVE_SMEARING  : 0.4     # Default

# Specify the geometry to be used in the optics calculation
OPTICS_GEOM        : tensor     # Default
# Include additional broadening for the loss function
OPTICS_LOSSFN_BROADENING : 0.0    # Default
```

The line `TASK : optics` is key here, as that is what tells us to perform an optical calculation. The other crucial line is `OPTICS_GEOM : tensor` - this tells it to calculate the full dielectric tensor of rutile. This produces 2 output files: `rut.odo` and `rut_epsilon.dat` - we are interested in the latter. 
