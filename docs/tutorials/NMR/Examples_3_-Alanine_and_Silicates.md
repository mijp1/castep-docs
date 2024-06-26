<h1>Examples 3 - Alanine and Silicates</h1>

We now look at some more realistic examples. These are fairly large crystals - to get them
to complete in a short time we will run them on a cluster.
**Use the following number of
cpus:**

**analine - 4 cristoballite - 6 quartz - 13**

(these numbers are chosen to give efficient k-point scaling)

<p style="border-width:3px; border-style:solid;"><i>
<b>Oxygen-17 NMR</b>
<br>
Oxygen is a component of many geological materials. Oxygen is
also important element in organic and biological molecules since it is often intimately involved in hydrogen bonding. Solid State ^17^O NMR should be a uniquely valuable probe as the chemical shift range of ^17^O covers almost 1000 ppm in organic molecules. Furthermore ^17^O has spin I = 5/2 and hence a net quadrupole moment. As a consequence of this the solid state NMR spectrum is strongly affected by the electric
field gradient at the nucleus.

Because the isotopic abundance of ^17^O is very low (0.037%) and the NMR linewidths due to the electric field gradient relatively large, only limited Solid State NMR data is
available. This is particularly true for organic materials. First principles calculations of ^17^O NMR parameters have played a vital role in assigning experimental spectra, and developing empirical rules between NMR  parameters and local atomic structure.</i>
</p>

# Alanine

## Files

* [alanine.cell](alanine/alanine.cell)
* [alanine.param](alanine/alanine.param)
* [alanine.pdb](alanine/alanine.pdb)

## Objectives

1. Compute the chemical shift and Electric field gradient for alanine
2. Assign the ^17^ NMR spectrum

<img alt="Fig3. Solid-State O17 NMR spectrum of L-alanine" src="../../img/nmr_tut3.png" width = "300"/>
<figure fig3>
  <figcaption>Fig3. Solid-State O17 NMR spectrum of L-alanine. (b) is from MAS (magicangle- spinning) (c) is from DOR (double-orientation rotation)</figcaption>
</figure>


## Instructions

1. Look at the cell and param files. The geometry for alanine was obtained by neutron diffraction and was downloaded from the [Cambridge Crystallographic database](https://www.ccdc.cam.ac.uk/). View the original pdb file note the hydrogen bonding

2. Run the example - the calculation is not fully converged. However, the relative shift between the two sites is fairly converged.

3. The experimental ^17^O NMR spectrum shows two peaks (Fig 3 (b)) - they are very broad due to the quadrupolar coupling, and overlap. The experimental parameters are given in Table 1.

4. Assign the two resonances A and B. Do all three computed parameters support this assignment?

| | |
|--|--|
|&#948;(A)-&#948; (B) (ppm)| 23.5|
|C<sub>Q</sub> (A) (MHz)| 7.86|
|&#951;<sub>Q</sub> (A)| 0.28|
|C<sub>Q</sub>(B) (MHz)| 6.53|
|&#951;<sub>Q</sub>(B)| 0.70|
| **Table 1: Experimental ^17^O NMR parameters for alanine. The two resonances are labeled A and B. Isotropic chemical shift &#948;, quadrupolar coupling C<sub>Q</sub>, and EFG asymmetry &#951;<sub>Q</sub>.**||

# Silicates - Quartz and Cristoballite

## Files

* [quartz.cell](silicates/quartz.cell)
* [quartz.param](silicates/quartz.param)
* [crist.cell](silicates/crist.cell)
* [crist.param](silicates/crist.param)



## Objectives

1. Compute the chemical shift and Electric field gradient for two silicates.
2. Assign the ^17^O NMR spectrum

## Instructions

1. The ^17^O parameters for two silicates are reported in Table 2. From the values you compute can you tell which one is quartz? (a suitable &#963;<sub>ref</sub> is 263ppm)




| | &#948; (ppm) | C<sub>Q</sub> (MHz) | &#951;<sub>Q</sub> |
|---|---|---|---|
|Material A| 37.2 | 5.21 | 0.13 |
|Material B| 40.8 | 5.19 | 0.19 |
| **Table 2: Experimental ^17^O NMR parameters for two silicates. Isotropic chemical shift &#948; , quadrupolar coupling C<sub>Q</sub>, and EFG asymmetry &#951;<sub>Q</sub>.** |
