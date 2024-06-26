## Files

*diamond.cell*

```
%block LATTICE_CART
0 1.7 1.7
1.7  0 1.7
1.7 1.7 0
%endblock LATTICE_CART

%block POSITIONS_FRAC
C   0.000000   0.000000   0.000000
C   0.250000   0.250000   0.250000
%endblock POSITIONS_FRAC


kpoints_mp_grid  4 4 4

symmetry_generate
```
*diamond.param*

```
comment         = nmr testing
iprint          = 1
xcfunctional = LDA
task : magres
fix_occupancy = true
opt_strategy : speed
cut_off_energy  =  30 Ry
```

## Objectives

Examine the convergence of the chemical shielding as the sampling of the electronic Brillouin zone (BZ) is increased.

## Instructions

1. Look at the files diamond.cell and diamond.param
2. We have specified the kpoints in the cell file using the keyword
`kpoints_mp_grid 4 4 4`
3. Run CASTEP for a range of kpoint meshes (say 2,4,6,8,10)
4. Examine (plot?) the convergence of the chemical shielding.

The computational cost scales linearly with the number of kpoints (i.e. the number of points in the irreducible Brillouin Zone). For a large unit cell (i.e. a small BZ) it may be possible to get converged results using a single k-point. But which kpoint should we choose?
For diamond we will look at 3 different k-points (0,0,0), (½,½,½) (¼,¼,¼). Specify the kpoint in the cell file using
```
%BLOCK KPOINTS_LIST
0.25 0.25 0.25 1.0
%ENDBLOCK KPOINTS_LIST
```
Which gives a result closest to the converged answer?
(as the diamond unit cell is rather small the 1 kpoint answer is not too close to converged. However, the observation holds true for all orthorhombic cells)

### Part 2

We now look at some more realistic examples.

**Oxygen-17**

Oxygen is a component of many geological materials. Oxygen is
also important element in organic and biological molecules since it is often intimately involved in hydrogen bonding. Solid State ^17^O NMR should be a uniquely valuable probe as the chemical shift range of ^17^O covers almost 1000 ppm in organic molecules. Furthermore ^17^O has spin I = 5/2 and hence a net quadrupole moment. As a consequence of this the solid state NMR spectrum is strongly affected by the electric
field gradient at the nucleus.

Because the isotopic abundance of ^17^O is very low (0.037%) and the NMR linewidths due to the electric field gradient relatively large, only limited Solid State NMR data is
available. This is particularly true for organic materials. First principles calculations of ^17^O NMR parameters have played a vital role in assigning experimental spectra, and developing empirical rules between NMR  parameters and local atomic structure.
