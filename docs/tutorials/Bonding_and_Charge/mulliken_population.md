
By default castep will calculate the Mulliken Population analysis at the end of every calculation (the keyword `popn_calculate` is set to `true` by default).

## Silicon

Here is a cell file. You can use the icon in the top right of the box to copy and paste the text. Save it in a file `silicon.cell`

```
%block lattice_abc
3.8 3.8 3.8
60 60 60
%endblock lattice_abc
!
! Atomic co-ordinates for each species.
! These are in fractional co-ordinates wrt to the cell.
!
%block positions_frac
Si 0.00 0.00 0.00
Si 0.25 0.25 0.25
%endblock positions_frac
!
! Analyse structure to determine symmetry
!
symmetry_generate
!
! Specify M-P grid dimensions for electron wavevectors (K-points)
!
kpoint_mp_grid 4 4 4

```

Here is a param file. You can use the icon in the top right of the box to copy and paste the text. Save it in a file `silicon.param`

```
xc_functional : LDA
cutoff_energy : 500 eV
spin_polarised : false
```

Run castep. Toward the end of the file `silicon.castep` you will find


```
Atomic Populations (Mulliken)
-----------------------------
Species          Ion     s       p       d       f      Total   Charge (e)
==========================================================================
Si              1     1.356   2.644   0.000   0.000   4.000     0.000
Si              2     1.356   2.644   0.000   0.000   4.000     0.000
==========================================================================

            Bond                   Population      Length (A)
======================================================================
         Si 1 -- Si 2                   2.99        2.32702
======================================================================
```
The atoms are not charged overall, and there is a large bond population. This is all indicative of a strong covalent bond.

### Comparing to Other Diamond Structures

To understand the results of this further, we will compare the results of this to 2 other diamond structures - GaAs and diamond (as in carbon-diamond).

For both of the cases, we will use an identical .param file (just rename them to ```diamond.param``` and ```GaAs.param```). The cell files will be slightly different - for diamond we will change the ```lattice_abc``` block lattice dimensions with 2.52 (rather than 3.8 - the structure is the same but the cell size is different). Naturally the ```Si```'s in the ```positions_frac``` block need to be replaced with ```C```'s

Similarly, towards the end of ```diamond.castep``` we find

```
Atomic Populations (Mulliken)
-----------------------------
Species          Ion     s       p       d       f      Total   Charge (e)
==========================================================================
C               1     1.076   2.924   0.000   0.000   4.000     0.000
C               2     1.076   2.924   0.000   0.000   4.000     0.000
==========================================================================

            Bond                   Population      Length (A)
======================================================================
          C 1 -- C 2                    3.00        1.54318
======================================================================
```
The results are rather similar but there are some interesting things to note:

- The ratio of the populations in s and p orbitals is closer to 1:3, which is expected for sp^3^ hybridization - this indicates that the bonds are much more perfectly hybridized, meaning it's more overlapped: this indicates stronger bonding
- The population of electrons in the bonds is the same (indicating that the same type of bonding is present), but the bond length is smaller - again indicating more overlap and thus stronger bonding.
- The charge on the atoms is 0 for both of them - this is expected given that it's 2 identical atoms.

Now we will compare it with GaAs. The same procedure is used, except the lattice length is now 3.93 and the atoms in ```positions_frac``` with Ga and As (it doesn't matter which one goes in which position/line).

We get an output looking like this:

```
Atomic Populations (Mulliken)
-----------------------------
Species          Ion     s       p       d       f      Total   Charge (e)
==========================================================================
Ga              1     1.180   1.743   9.994   0.000  12.917     0.083
As              1     1.488   3.595  10.000   0.000  15.083    -0.083
==========================================================================

            Bond                   Population      Length (A)
======================================================================
         Ga 1 -- As 1                   0.34        2.44652
======================================================================
```

Despite having the same structure, the results are very different now -

- Unlike before, the Ga and As ions have charges on them - this is indicative of ionic/polar character
- There is now a much smaller population in the Ga-As bond. This indicates that the covalent bonds between them aren't particularly strong.
- Especially in the case of Ga, the s:p ratio is far off from 3 - this again indicates that the bonding is not like in the cases above

## Diatomic cases

Next we will examine diatomic molecules - HF, HCl and HBr. We will examine the results given what we know about how electronegativity varies across the periodic table.

Here is the ```HF.cell``` file

```
%block lattice_abc
5 5 5
90 90 90
%endblock lattice_abc

%block positions_abs
H 2 2 2
F 2.91 2 2
%endblock positions_abs
```

Since we are just trying to look at a single diatomic molecule, the cell is defined rather simply - an arbitrarily large cube for the cell (making it too large would make the calculation take longer, but too small and it won't be accurate), 1 atom placed about in the middle, and the 2^nd^ atom placed a bond length away from it. The bond lengths can be found on a [database](https://cccbdb.nist.gov/), or you may perform a [geometry optimisation](../../..//documentation/Geometry_Optimisation/overview) to find it yourself if you wish.
