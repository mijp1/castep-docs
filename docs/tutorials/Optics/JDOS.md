### Joint DOS
See  `examples/Si2_JDOS`. This is a simple example of using `optados` for calculating joint electronic density of states.  We choose to recalculate the Fermi level using the calculated DOS, rather than use the Fermi level suggested by castep, and so `EFERMI: OPTADOS` is included in the `Si2.odi` file.  

* Execute castep and optados using the example files.  The JDOS is written to `Si2.jadaptive.dat`. A file suitable for plotting using `xmgrace` is written to `Si2.jadaptive.agr`.
* Check the effect of changing the sampling by increasing and decreasing the value of `JDOS_SPACING` in the `Si2.odi` file. 
