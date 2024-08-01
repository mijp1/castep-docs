# JDOS

The JDOS (Joint Density of States) is effectively the density of states for an energy range which is equal to the difference between conduction and valence bands - this leads to being able to calculate the probability of an excitation occurring for a given photon energy. One can intuitively understand how this leads to the calculation of optical properties: photons that have energies that correspond to gaps between conduction and valence bands will interact with the material and give rise to its optical properties.

In this tutorial, we will calculate the JDOS for silicon, and then relate that to the imaginary and real part of the dielectric function.

### Joint DOS
See  `examples/Si2_JDOS`. This is a simple example of using `optados` for calculating joint electronic density of states.  We choose to recalculate the Fermi level using the calculated DOS, rather than use the Fermi level suggested by castep, and so `EFERMI: OPTADOS` is included in the `Si2.odi` file.  

* Execute castep and optados using the example files.  The JDOS is written to `Si2.jadaptive.dat`. A file suitable for plotting using `xmgrace` is written to `Si2.jadaptive.agr`.
* Check the effect of changing the sampling by increasing and decreasing the value of `JDOS_SPACING` in the `Si2.odi` file.
