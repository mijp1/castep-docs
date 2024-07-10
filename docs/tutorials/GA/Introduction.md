# Introduction

In this tutorial we will look at exactly what castep GA is doing when it is used, examining what the input files do what the output files tell us. We will go through a silicon example to do so - starting from a heavily distorted diamond cell and ending in a diamond structure.

In my run, the "family tree" of the final structure found looks like this:

![Family tree](Family_tree.png)

We will have a look at how this happened.

!!! note
    Yours is unlikely to be exactly the same - castep GA is heavily based on randomness and thus each run is going to be different, likely even if using the same seed
