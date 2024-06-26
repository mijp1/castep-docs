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
3. Run CASTEP for a range of kpoint meshes (say 2, 4, 6, 8, 10)
4. Examine/plot the convergence of the chemical shielding.

  The convergence plot should look like this:
  <br>
  <img alt = "Diamond convergence" src = "../../img/diamond_convergence.png" width="500"/>

The computational cost scales linearly with the number of kpoints (i.e. the number of points in the irreducible Brillouin Zone). For a large unit cell (i.e. a small BZ) it may be possible to get converged results using a single k-point. But which kpoint should we choose?

For diamond we will look at 3 different k-points (0,0,0), (½,½,½) (¼,¼,¼). Specify the kpoint in the cell file using
```
%BLOCK KPOINTS_LIST
0.25 0.25 0.25 1.0
%ENDBLOCK KPOINTS_LIST
```
Which gives a result closest to the converged answer?
(as the diamond unit cell is rather small the 1 kpoint answer is not too close to converged. However, the observation holds true for all orthorhombic cells)
