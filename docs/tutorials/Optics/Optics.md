### Optics

In this tutorial, we will use Optados to examine the optical properties of silicon and aluminium.

## Silicon Properties

For Silicon, we will use the input `cell` file

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
SPECTRAL_KPOINTS_MP_GRID 14 14 14
```

and the `param` file

*Si.param*
```
TASK                   : SPECTRAL
SPECTRAL_TASK          : OPTICS
CUT_OFF_ENERGY         : 200
IPRINT                 : 2

```
Run Castep as usual. When it is done, run Optados on Si with the Optados input file

<a id="odi"></a>
*Si.odi*
```
# Choose the task to perform
TASK               : optics

# Sample the JDOS at 0.01 eV intervals
JDOS_SPACING       : 0.01

# Calculate the JDOS up to 60eV about the valence band maximum
JDOS_MAX_ENERGY    : 30

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
OPTICS_GEOM        : polycrystalline     # Default

# Include additional broadening for the loss function
OPTICS_LOSSFN_BROADENING : 0.0    # Default

```

After running Optados, we get several output `.dat` and `.agr` files. We will now examine them.

### Epsilon.dat

First let's look at the file containing the (polycrystalline) dielectric function - `Si_epsilon.dat`. The contents of the file look like

```
0.0000000000000000        14.107263205387552        0.0000000000000000     
1.0003334444814937E-002   14.107363765236455        0.0000000000000000     
2.0006668889629875E-002   14.107665455817445        0.0000000000000000     
3.0010003334444812E-002   14.108168310237140        0.0000000000000000    
```

The 1st column corresponds to the energy (of the photon interacting with it), the 2nd is the real component of the dielectric function, and the 3rd is the imaginary component.

The header also contains the results of the sum rule $\int_0^{\omega'} \textrm{Im} \epsilon(\omega) \mathrm{d}\omega = N_\textrm{eff}(\omega')$

`# Result of sum rule: Neff(E) =     7.0939380864459736`

Where $N_\textrm{eff}$ is the effective number of electrons contributing to the absorption process, and is a function of energy.

Let's plot it using xmgrace. Although you could plot the `dat` file (after configuring a bit to get it to plot everything properly), but luckily Optados has also generated an equivalent `agr` file that contains the same data, but also formats the xmgrace graph nicely. So let's use

`xmgrace Si_epsilon.agr`

and get an output looking like

![Standard dielectric plot of Si](Si_dielectric.png){width="60%"}

This is a very neat result: it looks just like what you'd expect from the [Debye equations](https://en.wikipedia.org/wiki/Dielectric#Debye_relaxation)

This file gives us all the data necessary for the calculation of any optical property of the cell examined. As we will see in the remainder of the tutorial, it is the basis of the generation of all the other files.

### Absorption

Next let's look at the absorption data. The `dat` file is very similar, this time with 2 columns: the 1st is the energy of the photon, and the 2nd is the absorption coefficient. It doesn't look particularly different, but if you wish to check, the data table should start like

```
0.0000000000000000        0.0000000000000000     
1.0020040080160320E-002   0.0000000000000000  
```

You can plot it with xmgrace the same way as before (again the `agr` file is prepared by Optados). It should look like this

<a id="absorption_graph"></a>
![Si absorption graph](absorption.png){width="40%"}

Let's examine how this is calculated. The absorption coefficient is calculated as

$$
\alpha(\omega) = \frac{2 \omega \kappa(\omega)}{c}
$$

Where $\omega$ is the energy divided by $\hbar$, $\kappa$ is the imaginary refractive index and $c$ is the speed of light. $\kappa$ can be easily obtained from the real and imaginary dielectric functions, knowing that the complex refractive index is the square root of the complex dielectric function - this leads to the equation

$$
\kappa(\omega) = \sqrt{\frac{1}{2} \left[ \sqrt{\epsilon_1(\omega)^2 + \epsilon_2(\omega)^2} - \epsilon_1(\omega) \right]}
$$

To check that this is really what's going on in the absorption calculation, we can use the Python script

```python
import numpy as np
c = 3e8  
hbar = 6.582119569e-16  
def calculate_property(energy, epsilon_1, epsilon_2):
    omega = energy / hbar  
    kappa = np.sqrt(0.5 * (-epsilon_1 + np.sqrt(epsilon_1**2 + epsilon_2**2)))
    alpha = (2 * omega * kappa) / c  
    return alpha
data = np.loadtxt('Si2_OPTICS_epsilon.dat')

energy = data[:, 0]  
epsilon_1 = data[:, 1]
epsilon_2 = data[:, 2]  
result = calculate_property_coefficient(energy, epsilon_1, epsilon_2)
output_data = np.column_stack((energy, result))
np.savetxt('predicted_abs.dat', output_data, fmt='%e', delimiter=' ')
```

If plot this together with `Si_absorption.dat`, you should get an identical result to [above](Optics.md#absorption_graph).

For the next properties we will calculate, we will also see where they come from using an almost identical Python script: simply change the `calculate_property` function.

<a id="refractive_index"></a>
### Refractive Index

Next we will look at the (real and imaginary) refractive index. This data is found in the `Si_refractive_index.dat` (and `.agr`) files. The file is similar to the previous, this time having 3 columns again - the 1st is energy, the 2nd is the real refractive index and the 3rd is the imaginary refractive index. We have already looked at how to calculate the imaginary refractive index [above](Optics.md#Absorption) (multiply it by some constants and you have the absorption coefficient). The calculation of the real refractive index is very similar, instead becoming

$$
n(\omega) = \sqrt{\frac{1}{2} \left[ \sqrt{\epsilon_1(\omega)^2 + \epsilon_2(\omega)^2} + \epsilon_1(\omega) \right]}
$$

and the accompanying Python script to verify if this is right is also very similar: the function becomes

```python
def calculate_property(energy, epsilon_1, epsilon_2):
    omega = energy / hbar  # Convert energy (eV) to angular frequency (rad/s)
    kappa = np.sqrt(0.5 * (epsilon_1 + np.sqrt(epsilon_1**2 + epsilon_2**2)))
    alpha = (2 * omega * kappa) / c  # Absorption coefficient (1/m)
    return alpha
```
If you're not interested in the Python output, just plot the `Si_refractive_index.agr` file using xmgrace, but if you wish to plot them together it is easiest to use

<a id="batch"></a>
`xmgrace -batch double.bat`

on the batch file

*double.bat*
```
READ BLOCK "predicted_refractive_indices.dat"
BLOCK XY "1:2"
S0 LEGEND "Predicted real"
BLOCK XY "1:3"
S1 LEGEND "Predicted imaginary"

READ BLOCK "Si2_OPTICS_refractive_index.dat"
BLOCK XY "1:2"
S2 LEGEND "Optados real"
BLOCK XY "1:3"
S3 LEGEND "Optados imaginary"

LEGEND 0.9, 0.9
```
This gives a graph that looks like this:

![Refractive index](refractive_index.png){width="40%"}

Once again, the values derived from the dielectric function dataset are identical, so they overlap - confirming for us how the properties are calculated. You may note that the imaginary refractive index's shape is [identical to that of the absorption](Optics.md#absorption_graph) - as we've seen, it's just that multiplied by a constant.

## Reflectivity

Next we will look at the reflectivity. This can be found in `Si_reflection.dat` (and `.agr` again). The 1st column represents the energy, as usual, while the 2nd is the corresponding reflection coefficient. The reflectivity is calculated using the [Fresnel equations](https://en.wikipedia.org/wiki/Fresnel_equations), using normal incidence and assuming the incoming light ray is coming out of air/vacuum (real and imaginary refractive index n~1~ and n~2~ of the first medium = 1). The imaginary component was also neglected. Under these conditions, the equation for reflectivity becomes

$$
R = \left( \frac{\tilde{n} - 1}{\tilde{n} + 1} \right)^2
$$

Where $\tilde{n}$ is the complex refractive index (real + imaginary part). We can implement this in the Python script more easily by separating it into functions to find the refractive indices (like we did [before](Optics.md#refractive_index)), and then use that to find the reflectivity.

```python
def calculate_refractive_indices(epsilon_1, epsilon_2):
    n = np.sqrt((epsilon_1 + np.sqrt(epsilon_1**2 + epsilon_2**2)) / 2)
    kappa = np.sqrt((-epsilon_1 + np.sqrt(epsilon_1**2 + epsilon_2**2)) / 2)
    return n, kappa
def calculate_property(n, kappa):
    n_complex = n + 1j * kappa
    reflectivity = np.abs((n_complex - 1) / (n_complex + 1))**2
    return reflectivity
```

Once again, we plot the output file of the script along with the `.dat` file and see that they're identical:

![Reflectivity](reflection.png){width="40%"}

If you are interested, you could change the Fresnel equation used and appropriately adjust the script to include the imaginary part, see how it behaves coming from a different medium, different angle etc. - there is a lot you could look at in terms of reflectivity.

### Conductivity

Next we will look the optical conductivity of silicon in `Si_conductivity.dat`. The 1st column is the energy, while the 2nd and 3rd columns are the real and imaginary parts of conductivity, both in the SI units Siemens per meter. The complex optical conductivity can be found by [approximating in the high frequency limit](https://en.wikipedia.org/wiki/Optical_conductivity#High_frequency_limit) and rearranging the equation to:


$\sigma_1(\omega) = \epsilon_0 \omega \epsilon_2(\omega)$

$\sigma_2(\omega) = -\epsilon_0 \omega (\epsilon_1(\omega) - \epsilon_\infty)$

where $\sigma_1$ is the real part and $\sigma_2$ is the imaginary. As usual $\epsilon_1$ corresponds to the real dielectric and $\epsilon_2$ to the imaginary. In this calculation, $\epsilon_\infty$ is approximated as 1. This can be implemented in our Python script as

```python
def calculate_conductivity(epsilon_1, epsilon_2, energy):
    omega = energy / hbar  
    sigma_1 = epsilon_0 * omega * epsilon_2  
    sigma_2 = -epsilon_0 * omega * (epsilon_1 - epsilon_inf)
    return sigma_1, sigma_2

```

As in previous cases, make sure all constants are defined and any file/variable names adjusted as you implement it.

Since there are 3 columns, it is easier to plot them together - we can simply repurpose our [previous batch file](Optics.md#batch) - just make sure to change file names as appropriate. The output should look like:

![Conductivity](conductivity.png){width="40%"}

### Loss Function

Lastly, we will examine the loss function in `Si_loss_fn.dat`. The 1st column is the energy, while the 2nd is the loss function for that energy. The header of the file shows the results of the two sum rules associated with the loss function:

$\int_0^{\omega'} \textrm{Im} -\frac{1}{\epsilon(\omega)}\omega \mathrm{d}\omega = N_\textrm{eff}$

and

$\int_0^{\omega'} \textrm{Im} -\frac{1}{\epsilon(\omega)}\frac{1}{\omega} \mathrm{d}\omega = \frac{\pi}{2}$

The loss function itself is calculated rather simply from the dielectric: it is simply the imaginary component of the inverse of the complex dielectric:

$\text{Loss}(\omega) = \text{Im}\left(\frac{1}{\epsilon(\omega)}\right)$

This leads to

$\text{Loss}(\omega) = \frac{\epsilon_2(\omega)}{\epsilon_1(\omega)^2 + \epsilon_2(\omega)^2}$

Let's implement this into our Python script by changing the function to

```python
def calculate_property(epsilon_1, epsilon_2):
    loss_function = epsilon_2 / (epsilon_1**2 + epsilon_2**2)
    return loss_function
```

Plotting it together with `Si_loss_fn.dat` on xmgrace gives us the graph

![Loss graph](loss.png){width="40%"}

## Changing Parameters

Now that we know what Optados optics does and how it works, let's try changing some parameters in the [Optados input file](Optics.md#odi) `Si.odi`, and re-running Optados, to see what effects it has.

### JDOS Parameters

First let's have a look at the effect of changing the line `JDOS_MAX_ENERGY : 30`. We'll set it to 10 and 100 and plot `Si_epsilon.agr` using xmgrace. Doing it for 10 gives the graph:



* Change parameters `JDOS_SPACING` and `JDOS_MAX` and check the effect on the optical properties.  Note: all of the other optical properties are derived from the dielectric function.  
*  The `optados` input file has been set up to calculate the optical properties in the polycrystalline geometry (`optics_geom = polycrystalline`).  It is possible to calculate either polarised or unpolarised geometries, or to calculate the full dielectric tensor.  To calculate the full dielectric tensor set `optics_geom = tensor`.  This time only the file `Si2_OPTICS_epsilon.dat` is generated.  The format of this file is the same as before (the columns are the energy and the real and imaginary parts of the dielectric function respectively), but this time the six different components of the tensor are listed sequentially in the order $\epsilon_{xx}$, $\epsilon_{yy}$, $\epsilon_{zz}$, $\epsilon_{xy}$, $\epsilon_{xz}$ and $\epsilon_{yz}$.

* Additional broadening can be included in the calculation of the loss function.  This is done by including the keyword `optics_lossfn_broadening` in the optados input file.  If you include this keyword and re-run optados, you will find that the file `Si2_OPTICS_loss_fn.dat` now has three columns.  These are the energy, unbroadened spectrum and broadened spectrum respectively.  

#### Aluminium

* Aluminium is a metal so we need to include both the interband and intraband contributions to the dielectric function.  To include the intraband contribution `optics_intraband = true` must be included in the `optados` input file.  When you run optados, the same files are generated as when only the interband term is included.  

* The `Al_OPTICS_epsilon.dat` file has the same format as before, but it now contains sequentially the interband contribution, the intraband contribution and the total dielectric function.  The file `Al_OPTICS_epsilon.agr` only contains the interband term.  In the same way, `Al_OPTICS_loss_fn.dat` contains the interband contribution, intraband contribution and total loss function.  All other optical properties are calculated from the total dielectric function and the format of the output files remains the same.

* In the case where the dielectric tensor is calculated and the intraband term is included, only the `Al_OPTICS_epsilon.dat` file is generated.  As before it contains each component, but this time it lists sequentially the interband contribution, intraband contribution and total dielectric function for each component.   

* This time, if additional broadening for the loss function is included by using the key word `optics_lossfn_broadening`,  `AL_OPTICS_loss_fn.dat` will contains four sequential data sets.  These are the interband contribution, the intraband contribution, the total loss function without the additional broadening and the broadened total loss function.  
