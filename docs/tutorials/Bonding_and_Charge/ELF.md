[ELF](../../documentation/Groundstate/ELF.md) (Electron Localization Function) is another way to measure electron density. In this tutorial, we will use it to roughly demonstrate lone pairs/hydrogen bonding in HF, H~2~O and NH~3~.

## HF

For HF we will use the `cell` file:

*HF.cell*
```
%block lattice_abc
5 5 5
90 90 90
%endblock lattice_abc

%block positions_abs
H 2 2 2
F 2.91 2 2
%endblock positions_abs

kpoints_mp_grid 4 4 4
```
and `param` file:

*HF.param*
```
xc_functional : LDA
cutoff_energy : 500 eV
spin_polarised : false
CALCULATE_ELF : TRUE
WRITE_FORMATTED_ELF : TRUE
```

!!! note
    For this tutorial we will simply assume that everything here is converged and stick to the LDA functional
