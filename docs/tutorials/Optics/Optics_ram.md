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
