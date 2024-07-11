

# Introduction

In this practical we will gain hands on experience using the CASTEP GA to find some \`unknown' structures. We will then utilise an on-the-fly machine learned GAP potential to reduce the cost of an *ab-initio* geometry optimisation.

Throughout this practical we will make use of the Stillinger-Weber (SW) pair potential. This allows GA runs to be carried out in a short time on desktops, though all of the approaches translate directly to *ab-initio* methods. Pair potentials (PP) are fast but far from perfect. Specifically, resultant cells from PP GA runs may not match structures found in nature (or with DFT), but instead be the lowest enthalpy cells based in the PP energy landscape.

N.B. The CASTEP GA (specifically the convex hull algorithm) is currently in development. This means that some of the devel code parameters will be promoted to standard input flags. Therefore, the exact keywords used for some of the options in this practical may differ from the release version (though all the functionality will be there). Therefore, if using the GA in future the documentation should be consulted first.

As a quick recap the CASTEP GA has 7 main steps:

1.  Generate an initial randomised population of parent cells.
2.  Perform a geometry optimisation to relax the parents & find their enthalpy based fitness.
3.  Select parents based on their fitness for breeding and breed to create a population of children.
4.  Mutate the children with a given probability (and to ensure no ionic overlaps).
5.  Relax these children with a geometry optimisation & find the enthalpy based fitness of patents and children.
6.  If the population has converged, or we have reach maximum allowed generations, then exit.
7.  Select members of parents and children based of fitness to be the next generation of parents and go to 3.


# Setup

In this workshop we will be running on a remote machine, `teaching0`. However, you can also access all the files in your user directory from the desktop machines. This means, if you would prefer to edit or view files on the desktops rather than the command line you can. All CASTEP commands must however be carried out on the command line on `teaching0`.

We should then log on to `teaching0` using

    ssh -Y USER@teaching0

where `USER` should be replaced with your username.

We then need to load the environment that contains all the programs we will need (`castep.mpi`, `castep_GA` etc) with

    source /tmp/castep_GA_workshop_venv/bin/activate

This should make everything we need available to us and you should now see

    (castep_GA_workshop_venv) USER@teaching0:

on your command line.
**Note:** if the terminal window is closed, or we disconnect from `teaching0` this `source` command will have to be re-ran to re-load all the required packages.

You should be in your home directory, so now extract the GA workshop files into your home directory

    tar -zxf /tmp/CASTEP_workshop_GA.tar.gz

which should create the directory `CASTEP_workshop_GA` containing all the files we will need in this workshop. Move into this directory with

    cd CASTEP_workshop_GA


# CASTEP GA for Ground State Silicon

Let's assume we don't know the ground state Stillinger-Weber silicon structure and set out to discover it using the CASTEP GA!


## Input Files

First navigate to the directory

    cd CASTEP_workshop_GA/1_Si_SW_GA

which should contain a CASTEP `Si.cell` and `Si.param` file, as well as the bash script `get_data.sh` and a python script `plot_results.py` (which we will use for results analysis later).

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


# SiC Stillinger-Weber GA

Now we will look at exploring a more complicated potential energy landscape, to do this we first navigate to the directory

    ./CASTEP_workshop_GA/2_SiC_SW_GA

Luckily CASTEP allows us to add our own parameterisation values for pair potentials. A parameterisation for an SiC SW pair potential has been added to the devel code block of the parameter file `SiC.param`.

This parameterisation leads to a more complex potential energy landscape than we saw with Si. Due to the limitations of this pair potential parameterisation the GA will find different structures than we would find with *ab-inito* calculations as this is a pair potential energy landscape. However, it will run quickly enough to be usable with the computational and time resources available. As such we will use the GA to explore this unknown potential energy landscape as we did with Si. N.B. Everything we are doing here works just the same with DFT.


## Running the CASTEP GA (Approx. 5-10 mins runtime)

The `SiC.param` file is very similar to the one used for Si, except the initial population is no longer completely random but instead generated as random high symmetry cells. The cell file contains an equal number of Si and C ions in random positions in a not unreasonably sized cell for this many ions Si and C ions.

To run CASTEP GA execute

    castep_GA SiC

and the GA will start, which will take approximately 6-7 minutes to run. You may want to look at the outputs as we did before, or take a quick break!


## Analysing the GA Output Files

The GA outputs for this SiC run can be analysed in the same way as in the case of Si. We can use the same scripts as earlier to create energy volume plots by running

    ./get_data.sh && python plot_results.py

Where this energy volume graph can be analysed as with Si. Make a note of the lowest enthalpy cell for use later.

If it looks like would benefit from running a few more generations, we can run a continuation using the same methods as before.

Have a look at the cell of the lowest enthalpy structure. It's likely that it seems a bit different to what we would expect in nature. As such, it would be good if we could run a full DFT calculation on some of the cells that have been found by the GA rather than relying on this pair potential parameterisation. However, we are a bit limited in computer power running, with all of the previous runs using only 3 cores. To overcome this hurdle we will exploit machine learning with Gaussian approximation potentials (GAP).


# Gaussain Approximation Potential Accelerated Geometry Optimisation

A common approach to machine learning is to train a model on some dataset of known solutions to a problem. This presents an issue with the application of machine learning to crystal structure optimisation for materials discovery as by definition the parameter landscape is not explored. One approach to this in CASTEP is to use a number of initial geometry optimisation or molecular dynamics steps ran with *ab-initio* methods as the initial training data. Subsequent geometry optimisation steps are then ran with a machine learned potential using Gaussian approximation potentials (GAP).

Ions move over the course of geometry optimisations (or molecular dynamics calculations), therefore the landscape may change such that the potential needs some extra *ab-inito* data to train on. As such, CASTEP adaptively chooses geometry optimisation steps over the course of the calculation to run with DFT which are then used to re-train the GAP potential. More information about this method is available in the paper by Stenczel *et al* <sup><a id="fnr.1" class="footref" href="#fn.1" role="doc-backlink">1</a></sup>. In general, less and less DFT geometry optimisation or MD steps will be required as the calculation continues and samples the parameter space more thoroughly (again see <sup><a id="fnr.1.100" class="footref" href="#fn.1" role="doc-backlink">1</a></sup>).

Note: All methods that are outlined here for the acceleration of a geometry optimisation can also be directly applied to molecular dynamics calculations. In practice, as the cost savings are generally greater the more steps are in the calculation, the cost savings of GAP are greatest for long running molecular dynamics calculations.


## Using GAP to Accelerate a Geometry Optimisation

We now navigate to the directory

    ./CASTEP_workshop_GA/2_SiC_SW_GA/DFT_polish_GAP

which contains 3 files. The first is the `SiC_GA_best.cell`, we should open this and replace the lattice and positions blocks with those from the best cell from our previous SiC run (that we made a note of earlier, if that note is lost open the image `../all_gens.png`). For me the best cell is `../SiC.gen_009_mem_006-out.cell`, but for you this could be different. The cell file we also specifies the QC5 pseudopotential library, a kpoint grid spacing and that the geometry optimisation will be fixed cell.

The `SiC_GA_best.param` file contains standard CASTEP parameters for a Damped Molecular Dynamics (DMD) geometry optimisation. Along with this is the devel code block required to call `hybrid-md` (which then calls `gap_fit`) to perform the GAP fitting.

Finally, there is an extra input file required for a GAP run, the `<SEED>.hybrid-md-input.yaml` file. This contains specific inputs relating to the GAP fitting. Here we specify a mandatory 10 initial DFT steps for the initial training dataset. After this we specify a minimum of 10 steps with the machine learned potential before re-fitting, whilst allowing CASTEP to vary the number of steps between re-fittings. We also allow GAP to also parellise over 3 cores.

**N.B.** The GAP run we carry out here has rather coarse convergence parameters, designed to allow for a fast run on a desktop in this workshop.


## Running CASTEP with GAP (Approx. 10-15 mins runtime)

To run CASTEP to perform the damped MD geometry optimisation with the aid of GAP call

    mpirun -n 3 castep.mpi SiC_GA_best

Whilst this is running (which will take 10 to 15 minutes) there will be information printed to the terminal window each DMD step. This states if the step is an initial (DFT) step, if the step is using the machine learned potential or if it is being ran with DFT in order to perform a re-fit.

It may be illustrative to open a ***new*** terminal window, navigate to the working directory and tail the `.castep` output file with

    tail -f SiC_GA_best.castep

in order to see the geometry optimisation running. Notice the steps using the machine learned potential are essentially free when compared with the DFT steps that require SCF calculations.


## Looking at the GAP Geometry Optimisation Results

First, let us consider the cost of this geometry optimisation. My calculation ran for a total of 152 DMD steps, though your mileage may vary. Using the following command we can see on which DMD steps CASTEP decided to perform DFT calculations

    grep SiC_GA_best.castep -e "Decided to perform"

We can also see how many of the DMD steps utilised DFT

    grep SiC_GA_best.castep -e "Decided to perform" | wc -l

In my case this showed that 22 of the total DMD steps were using DFT, therefore approximately 85% of the DMD steps were carried out with the machine learned potential. Though there is a cost associated with these steps and with the re-fitting of the GAP potential, it is clear this represents a large cost saving. Very roughly (based on average time for one SCF cycle in my DMD run) without GAP this run would have taken approximately 4 times as long for no extra information.

We can also visualize (using jmol or similar) both the output structure from the DMD run (`SiC_GA_best-out.cell`) and the pre-geometry optimisation structure (`SiC_GA_best.cell`). It is likely these are quite different. This is due to the difference (and lower quality!) of the SW parameterisation compared with any DFT approach.

## (Optional Extension) Running GA with Other Systems

You may also want to try the CASTEP GA with another system or pair potential parameterisation (it is likely not possible to run a full DFT run of the GA in the timescale of this practical). If this is the case you can use the blank input file in

    ./CASTEP_workshop_GA/2.5_GA_blank


## (Optional Extension) Running GAP with Other Systems

If you are interested in applying GAP to your calculations you may want to try GAP with a material of interest to you. If you would like to try with a cell and parameters of your choice there are blank input files in

    ./CASTEP_workshop_GA/2.5_GAP_blank


# Footnotes

<sup><a id="fn.1" href="#fnr.1">1</a></sup> Tamás K. Stenczel, Zakariya El-Machachi, Guoda Liepuoniute, Joe D. Morrow, Albert P. Bartók, Matt I. J. Probert, Gábor Csányi, Volker L. Deringer; Machine-learned acceleration for molecular dynamics in CASTEP. J. Chem. Phys. 28 July 2023; 159 (4): 044803. <https://doi.org/10.1063/5.0155621>
