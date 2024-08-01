# JDOS

The JDOS (Joint Density of States) is effectively the density of states for an energy range which is equal to the difference between conduction and valence bands - this leads to being able to calculate the probability of an excitation occurring for a given photon energy. One can intuitively understand how this leads to the calculation of optical properties: photons that have energies that correspond to gaps between conduction and valence bands will interact with the material and give rise to its optical properties.

In this tutorial, we will calculate the JDOS for silicon, and then relate that to the imaginary and real part of the dielectric function.

## JDOS Calculation

Let's start by actually getting the JDOS results. Run Castep on the `cell` file

*Si.cell*
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

and `param` file

*Si.param*
```
TASK                   : SPECTRAL
SPECTRAL_TASK          : DOS
PDOS_CALCULATE_WEIGHTS : TRUE
CUT_OFF_ENERGY         : 200
```

Afterwards, run Optados with the Optados input file

*Si.odi*
```
TASK              : jdos

JDOS_MAX_ENERGY   : 30
JDOS_SPACING      : 0.01
EFERMI            : optados
DOS_SPACING       : 0.1
BROADENING        : adaptive # Default
ADAPTIVE_SMEARING : 0.4      # Default
FIXED_SMEARING    : 0.3      # Default
SET_EFERMI_ZERO : true       # Default
NUMERICAL_INTDOS      : false  # Default
FINITE_BIN_CORRECTION : true  # Default
```

The line `TASK : jdos` is what tells it to calculate the JDOS in which we are interested. This will generate the file `Si.jadaptive.dat` (as well as the accompanying `agr` file). The `dat` file data table should start like this

```
0.0000000000000E+00    0.3375085335870E-14
0.1000333444481E-01    0.3869643667676E-14
```

The 1st column is the energy, and the 2nd is the JDOS. The `agr` file allows us to make a nice plot with xmgrace (though  of course you could plot the results with your software of choice) - let's run

`xmgrace Si.jadaptive.agr`

 The result should look like this:

 ![JDOS Result](jdos_result.png){width="50%"}

## Imaginary Dielectric

Now that we have the results, let's examine how they are used to get the dielectric function. You may immediately notice that the shape of the graph above is completely different from either component of the dielectric function. We will need to process the results for them to be meaningful.

We will start processing the data by using the Python script

```py
energies = []
imaginaries = []

input_file = 'Si.jadaptive.dat'
output_file = 'imaginary.dat'
new_lines = []
with open(input_file, 'r') as infile:
    for line in infile:
        if "#" in line:
            continue
        parts = line.split()
        if len(parts) == 2:
            energy = float(parts[0])
            jdos = float(parts[1])
            energies.append(energy)
            if jdos == 0.0 or energy == 0:
                test = 0
            else:
                test = (jdos/energy**2)
            imaginaries.append(test)
            new_line = f"{energy} {jdos} {test}\n"
            new_lines.append(new_line)

with open(output_file, 'w') as outfile:
    outfile.writelines(new_lines)

```
This creates a new file `imaginary.dat` that contains the energy, JDOS and our first approximation of the imaginary dielectric function in the 3rd column. This is approximated by the relationship

 $\epsilon_2 \propto \frac{\text{JDOS}}{\omega^2}$

 which was implemented in the line

```py
test = (jdos/energy**2)
```

We can plot this `dat` file by using xmgrace on the batch script

*plot.bat*
```
READ BLOCK "imaginary.dat"
BLOCK XY "1:3"
```

to get just the imaginary output. The graph should look like this:

![First Imaginary Approximation](first_approx.png){width="50%"}

* Check the effect of changing the sampling by increasing and decreasing the value of `JDOS_SPACING` in the `Si2.odi` file.
