
# Castep GA for silicon

Let's assume we don't know the ground state Stillinger-Weber silicon structure and set out to discover it using the CASTEP GA!


## Input Files

We will use the `cell` file
*Si.cell*

```
%block LATTICE_ABC
ang
3.8 3.8 3.8
 90  90  90
%endblock LATTICE_ABC

%block POSITIONS_FRAC
Si  0.203  0.617  0.209
Si  0.844  0.442  0.350
Si  0.964  0.379  0.096
Si  0.762  0.524  0.941
Si  0.544  0.605  0.781
Si  0.238  0.597  0.531
Si  0.728  0.914  0.742
Si  0.209  0.929  0.435
%endblock POSITIONS_FRAC

%BLOCK SPECIES_POT
QC5
%ENDBLOCK SPECIES_POT

symmetry_generate
symmetry_tol : 0.05 ang
```

The definition of the lattice and positions of the atoms is fairly inconsequential to the kind of result you will get: in the 0th generation it gets effectively randomised. The only thing that matters is the amount of Si atoms defined in the `POSITIONS_FRAC` block - here it is 8 so future structures will also have 8 atoms. Another thing worth noting is the QC5 potential being used - it is used due to its speed, which is essential considering how many calculations will be done.

For the `param` file we will use
*Si.param*

```
task             = genetic algor # Run the GA
ga_pop_size      = 12            # Parent population size
ga_max_gens      = 12            # Max number of generations to run for
ga_mutate_amp    = 1.00          # Mutation amplitude (in Angstrom)
ga_mutate_rate   = 0.15          # Probability of mutation to occur
ga_fixed_N       = true          # Fix number of ions in each member based on input cell

rand_seed        = 101213 # Random seed for replicability
opt_strategy     = SPEED  # Run quick

geom_max_iter    = 211 # Can have a large max iter as using pair potentials

# Don't write most output files for each population member
write_checkpoint = NONE
write_bib        = FALSE
write_cst_esp    = FALSE
write_bands      = FALSE
write_cell_structure = TRUE

######################################
# Any extra devel code options	     #
# & required GA specific devel flags #
######################################

%block devel_code

  # Command used to call castep for each population member
  # If not given this defaults to castep.serial
  CMD: castep.serial :ENDCMD

  GA:

    PP=T   # Using a pair potential
    IPM=M  # Randomly mutated initial population

    CW=24  # Num gens for convergence

    NI=F   # No niching
    FW=0.5 # Fitness weighting

    # Asynchronous running options
    # Required for asynchronous running, without this all geom opts will be run
    # one after another
    AS=T   # Run geometry optimisations asynchronously
    MS=3   # Run 3 geometry optimisations at once

    # Random symmetry children
    NUM_CHILDREN=12
    RSC=F

    CORE_RADII_LAMBDA=0.8 # Core radii 0.8 pseudopotenital radii

    SCALE_IGNORE_CONV=T   # Ignore convergence in fitness calcualtion

  :ENDGA

  # Use pair potentials in geometry optimisations and perform a final snap to symmetry
  GEOM: PP=T SNAP=T :ENDGEOM

  # Use the Stillinger-Weber pair potential
  PP=T
  PP:
    SW=T
  :ENDPP

%endblock devel_code
```

It is important to understand what is going on in this file. The line `task = genetic algor` is what tells is to actually run GA. The GA parameters tell the GA how to run- unlike in the previous tutorial (ADD LINK), all of these values are reasonable (the mutation amplitude and rate shouldn't be far off those values in most cases). As you can see, we will have 12 parents in each generation (meaning 12 cells will be chosen to breed), and it will run for 12 generations. `ga_fixed_N` is what ensures that there are fixed 8 ions in the cell (the more information is fixed, the faster it'll get a decent result as it'll have fewer things to try). `op_strategy = SPEED` and `geom_max_iter = 211` help ensure that the geometry optimisations (of which there are 12 per generation - so 144 in total for this first run!) are fast but reasonably accurate.

A very important thing to note are the lines

```
NUM_CHILDREN = 12
RSC = F
```

This is highly atypical and would not be used in most actual calculations: instead, it would look like this
```
NUM_CHILDREN=11
RSC=T
RSN=1
```

With the general rule that `NUM_CHILREN` + `RSN` = `ga_pop_size` (defined earlier). What that does is, in every generation, it creates 1 child as a random high-symmetry structure (with the other 11, `NUM_CHILDREN`, being bred). The reason this is not used here is because Silicon is a very simple example: it is likely to simply randomly guess that it's diamond, meaning the actual GA was inconsequential and the result is uninteresting.

!!! note
    Though not a gurantee, using an identical `RAND_SEED` makes it likely that you'll get identical results to this tutorial, so you can follow along more easily

The `devel_code` block is a bit more complex. The fact that pair potentials are used are defined in both the `GA` sub-block and `GEOM` sub-block, again necessary for speed. The `CMD` sub-block is there so that geometry optimisations are performed on all members: what happens is `cell` files are generated (initially randomly and in the 1st generation onwards by breeding + mutation), as well as respective `param` files that tell them to geometry optimise, for each member, and then they are run.

Like standard CASTEP, the CASTEP GA requires both a cell and a param file input. However, we likely do not know what the best cell structure is. As such, we give a cell that contains multiple ions of the species that we are interested in. The number of ions given in this cell is used as a baseline for the number of ions in cells created & bred over the course of the GA, where at least one ion of each species will always be represented at least once in each cell. We can also fix the number of ions in each population member to match that in the input cell. We should not add too few ions, as cells with more ions contain more \`genetic information', meaning smaller cells don't benefit from breeding operations as much.

The exact positions of the ions in the input cell file does not matter, as they will be randomised during the initial population generation. However, the cell given should be a \`reasonable' guess w.r.t. the cell density (as the density of the input cell is used when creating the initial randomised population). I.e. the total density of the input cell should not be unreasonable, though the exact ionic positions do not matter.

For example, looking at the `Si.cell` in `./1_Si_SW_GA/Si.cell` we see the cell is cubic with a not unreasonable volume for 8 Si ions, though the positions of those 8 ions is random.

The param file is similar to a standard CASTEP input file, with some extra GA specific parameters. The GA copies all of the parameters to each bred population member, changing only the `task` to `geometry optimization`, before performing geometry optimisations for fitness evaluation. This means we can use any of the tools CASTEP has available during the populations geometry optimisation. N.B. any GA parameters passed to a standard CASTEP geometry optimisation, *e.g.* `ga_pop_size`, are ignored by CASTEP.

Looking at the `Si.param` file we see the task is set to `genetic algor` and we are running with a population size of 12 over 12 generations. There are a few GA specific parameters given in the devel code block, the main ones are

-   `CMD: mpirun -n 1 castep.mpi :ENDCMD`
    -   This gives the command that will be used to call CASTEP to perform the geometry optimisations on each population member. This can be changed to any valid call, for example to run over many cores.
    -   A serial calculation is used here as we are using cheap pair potentials.
-   `AS=T` and `MS=3`
    -   This allows asynchronous parellisation, where 3 population members will have geom-opts performed at the same time.

These parameters are chosen due to computational resources available in this workshop, but can be trivially increased based on available resources.

The large comment block at the bottom of the input file lists a lot of parameters that can be used with the various GA methods currently implemented. However, for now let's run a GA!


## Running the CASTEP GA (Approx. 5-6 mins runtime)

First lets check our CASTEP and CASTEP GA binaries are accessible by running:

    which castep_GA && which castep.mpi

This should return something like

    /tmp/castep_GA_workshop_venv/bin/castep_GA
    /tmp/castep_GA_workshop_venv/bin/castep.mpi

Your paths may differ but as long as this command completes without error everything we need is (hopefully) set up correctly.

To run CASTEP GA simply call

    castep_GA Si

and the GA will start, which will take approximately 5-6 minutes to run. While this is running have a look at the next subsection for a description of the output files that are being created.


## CASTEP GA Output Files

The CASTEP GA generates a lot of output files; as well as a main GA output `<seed>.castep` there are also input and output files for each population member. The population member seeds are labeled with the generation and population number, for example

    Si.gen_002_mem_006.cell

is the input cell file for the 6<sup>th</sup> member of generation 2. You should see input files being generated periodically (a generation at a time). In-between the output file generation geometry optimisations are automatically performed on each population member (here asynchronously 3 at a time).

If the Si CASTEP run we started in the last section is still running we can have a look at the main output file. In a ***new*** terminal window navigate to

    ./CASTEP_workshop_GA/1_Si_SW_GA

and run

    tail -f Si.castep

This will show us the output of the main GA `Si.castep` file being generated in real time.

The main output file contains a lot of information and can be a bit overwhelming. It gives all the parent structures and the pre and post geometry optimisation child structures as well as information about mutation operations. Once this GA run has completed move on to the next section to analyse our output files.


## Analysing the GA Output Files

The GA saves the post geometry optimisation structures from all evaluated population members in the file `<SEED>.xyz` in standard xyz format. We can view a nice animation of all of the structures the GA explored with jmol by opening this xyz file in jmol with

    Jmol.jar Si.xyz

then going to `tools -> animate -> once`.

In order to analyse the structures created by the GA in more detail we could look at just the enthalpy of the structures, where we may be most interested in the lowest enthalpy structure found. The GA always takes the lowest enthalpy cell into the next generation, so we can look at the last generation to find the lowest enthalpy cell with

    grep Si.castep -e "GA: gen # 12" | tail -n 24 | sort -k10,10g

which gives the final generation in order of enthalpy (in eV/ion).

However, we gain a lot of information about the phase space with the other cells the GA finds on the way to the minimum enthalpy cell. To explore this a bit more run

    ./get_data.sh

You may need to make this executable by running `chmod u+x get_data.sh`. This script generates an output file `out.put` that contains a summary of the data for each population member in 9 columns:

1.  Generation number.
2.  Parent or child.
3.  Member number.
4.  Error file detected (T or F).
5.  Geometry optimisation converged (T or F).
6.  Number of ions in the cell.
7.  Enthalpy per ion of converged cell (in eV/ion).
8.  Volume of converged cell (in Angstrom/ion).
9.  Member filename.

We can then plot the population as the enthalpy (per ion) against the cell volume (per ion) by running the python script

    python plot_results.py

This will produce two plots; one is an animation of the population members in each generation, the other is every explored population member over all generations with the lowest enthalpy structure labeled. It will also save these to the current directory.

From this representation we can gain information about the phase space of SW Si, where the lower enthalpy structures represent structures on the energy-volume curve for SW Si. There are also structures found by the GA above the energy-volume curve that are likely unstable.

We can have a look at the structure of the lowest enthalpy cell (as labeled on your graph) with jmol (or your crystal structure viewing program of choice). If using jmol run

    Jmol.jar <SEED>-out.cell

where `<SEED>` is the seed of the lowest enthalpy population member given on the output graph and the `-out.cell` postfix gives us the structure post geometry optimisation (it is important to look at this cell rather than the input cell). For example, for me (though your seed may be different than this) this is

    Jmol.jar Si.gen_006_mem_012-out.cell

It can sometimes be difficult to figure out the exact space group of crystals from a visual inspection. So, we can use `c2x` to find the space group of our lowest enthalpy cell within a tolerance with

    c2x --int -e=0.1-0.0001 <SEED>-out.cell

which for my lowest enthalpy member is

    c2x --int -e=0.1-0.0001 Si.gen_006_mem_012-out.cell

For me this gives the output

    Tol=0.001    International symmetry is Fd-3m

which is the diamond structure space group we would expect for Si.


## GA Continuation (Approx. 2 mins runtime)

It is possible you won't find the $\text{Fd}\overline{3}\text{m}$ structure (or one close to it, remember the GA is a coarse search) as there is some randomness in a GA run and our population size and number of generations is small. But don't worry, it is possible to restart a GA run from exactly where you left off. Let's do this now (even if you did find the $\text{Fd}\overline{3}\text{m}$ structure).

To perform a continuation open the `Si.param` file in your text editor of choice. We then tell CASTEP GA we want to continue from where we left off. We do this by giving the location of the `.xyz` file that contains all explored structures by uncommenting the following in the `Si.param` file

    # continuation = Si.xyz

so it reads

    continuation = Si.xyz

Also, we want to run for more generations, so change the value for `ga_max_gens` from 12 to 18 (to run 6 more generations) *i.e.* in the `Si.param` file change the line

    ga_max_gens      = 12            # Max number of generations to run for

to

    ga_max_gens      = 18            # Max number of generations to run for

**Important: Do not change the `ga_pop_size` as this will cause an error.**

We can then call CASTEP GA as before with

    castep_GA Si

and it will read in the members from generation 12 of our last run and start generation 13 from these. Running for generations 13 to 16 should take approximately 1-2 minutes.

Have a look at the outputs of this run by re-running

    ./get_data.sh

and then plotting with

    python plot_results.py

You may want to have a look at the lowest enthalpy structure with Jmol. You may also want to have a look at some of the other structures that are low on the energy/volume plot (a good way to find their seed names will be from the `out.put` file).
