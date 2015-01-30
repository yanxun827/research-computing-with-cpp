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

### Hardware considerations

So can we do that?  Well of course there are a number of complexities we want to think about.

For a start, we need to worry about whether it's actually faster - in the end, on many traditional file systems, X processes writing out to the same disk/file will take more time than one process writing out the same data.

However on modern HPC systems like Legion, we have parallel file-systems and file-system hardware which are designed to allow many writes at once to happen accross the multiple disks which make up the filesystem.

Because these systems are connected to the compute nodes by a network, you also gain the advantage of having multiple network links worth of bandwidth to do the file reads/writes rather than the network bandwidth of the node that process zero is running on.

As is common with this sort of problem in HPC, there were a bunch of competeing vendor specific libraries for doing this, until MPI-IO was added into the MPI standard.

### MPI-IO

So what is MPI-IO?
