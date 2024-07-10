# Introduction

In this tutorial we will look at exactly what castep GA is doing when it is used, examining what the input files do what the output files tell us. We will go through a silicon example to do so - starting from a heavily distorted diamond cell and ending in a diamond structure.

In my run, the "family tree" of the final structure found looks like this:

![Family tree](Family_tree.png)

We will have a look at how this happened.

!!! note
    Yours is unlikely to be exactly the same - castep GA is heavily based on randomness and thus each run is going to be different, likely even if using the same seed

## Input Files

We will use the `cell` file

*Si.cell*

```
%block LATTICE_ABC
ang
5.4 5.4 5.4
90  90  90
%endblock LATTICE_ABC

%block POSITIONS_FRAC
Si  0.2   0.01   0.04
Si  0.21   0.29   0.30
Si  0.59   0.43   0.03
Si  0.9   0.81   0.30
Si  0.53   0.03   0.58
Si  0.72   0.21   0.79
Si  0.04   0.45   0.4
Si  0.28   0.85   0.69
%endblock POSITIONS_FRAC

%BLOCK SPECIES_POT
QC5
%ENDBLOCK SPECIES_POT

symmetry_generate
symmetry_tol : 0.05 ang
```

Note that this is just a heavily distorted version of a ideal diamond silicon cell - `LATTICE_ABC` defines its cubic structure with the lattice parameter that is expected, while the fractional positions of all the Si's are slightly moved from where they'd be expected for diamond - for example the first line

`Si 0.2 0.01 0.04`

is only a bit off from where it'd be in a perfect diamond, which is

`Si 0 0 0`

In a normal castep GA run the positions of all the Si's, and even the starting unit cell, is completely arbitrary, and in most cases it should have little impact on the run. However, in this case we are trying to get it to get to diamond quickly, so that it is easier to analyse what is going on.

For the `param` file we will use

```
task             = genetic algor # Run the GA
ga_pop_size      = 12           # Parent population size
ga_max_gens      = 6            # Max number of generations to run for
ga_mutate_amp    = 0.05          # Mutation amplitude (in Angstrom)
ga_mutate_rate   = 0.1          # Probability of mutation to occur
ga_fixed_N       = true          # Fix number of ions in each member based on input cell

rand_seed        = 129189 # Random seed for replicability


# continuation = Si.xyz

#################################################################
# CASTEP Geomtery Optimisation Parameters		        #
# These will be used for each cells geom opt/fitenss evaluation #
#################################################################

comment          = Si_Convex_hull

#rand_seed        = 100003 # Random seed for replicability
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
    NUM_CHILDREN=11
    RSC=T
    RSN=1

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

There is a lot more to discuss here. For more information on all the key words look at the documentation (ADD LINK LATER). However, let's look at the key information here:

- `ga_max_gens` defines the number of generations it will run for. 6 is a very low amount, but it does the job here
- `ga_mutate_amp` describes how severe the mutations are. Again, this is using a low number because we're starting not too far off the final result
- `ga_mutate_rate` tells you the probability of different parts of the cell mutating - 0.1 is a fairly ordinary value to use

A key line for this tutorial is

`rand_seed        = 129189`

This means that you may be able to replicate similar results, though that isn't guranteed [[CHECK IF EVEN TRUE]]

## Running

We can now run castep GA on the Si. This should take about 3 minutes to run, but look at what is happening as it is running - you can use `tail -f Si.castep` to keep track of how the `castep` output file is being generated. Also, refresh your directory regularly to see what's being generated.

The first files to be generated will have the same format as `Si.gen_000_mem_001.cell` and `Si.gen_000_mem_001.param` - this is the 1st member of the 0th generation, which is generated randomly using the `Si.cell` as well as the mutation values specified in the `Si.param` file

!!! note
    I will refer to future files as being `<seed>.file_ending`in the interest of  brevity, specifying details where necessary

You should get a total of 12 members like that, up to `mem_012`. As it's running, the 0th generation will start to get `<seed>.geom` and `<seed>-out.cell` outputs for each member. These are obtained as fast geometry optimisations (ADD LINK) are performed on the randomly generated cells.

After the last member finishes its geometry optimisation, the next generation is generated: you'll get `<seed>.cell` and `<seed>.param` files.

This is the point where the `castep` output file gets interesting. You have a table that starts like 
