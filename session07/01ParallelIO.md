---
title: Parallel I/O
---

## Overview of parallel I/O

### Amdahl's Law

So back in session 4, you learned about Amdahl's law, and you saw how the scalability of a parallel code is limited by the proportion of the code that is serial (impossible to parallelise).

When you look at the parts of the code that are serial, one of the biggest contributors to the serial run time is reading from and writing to disk.

So can we parallelise this too?

### Traditional MPI program

So when we look at the kind of MPI programs we have written we tend to see the following pattern.

At the start of the program, we read in our input file on process zero, communicate that to the other processes (either one by one or broadcasting, it doesn't really matter), the other processes do a bunch of computation/communication, and then at some time-step we decide to write out our state to disk, either to keep a trajectory of our simulation (or whatever) or simply to checkpoint the program so that we can restart it.

All the processes send their data back to process zero and wait while it writes out to disk, and then they go into a cycle of parallel computation and serial I/O until the end of the program.  This is clearly wasteful!

What we want to do is have each process write out its data simultaneously in some way which keeps it:

* coherrant

* without errors(!)

* is fast

As is common with this sort of problem in HPC, there were a bunch of competeing vendor specific libraries for doing this, until MPI-IO was added into the MPI standard.
