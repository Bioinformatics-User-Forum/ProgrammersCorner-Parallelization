# swarm on Biowulf

The [`swarm` utility on Biowulf](https://hpc.nih.gov/apps/swarm.html) is a handy way to manage many submissions.

## Many jobs without swarm

This directory has a number of jobs that we want to run. Without `swarm` we need 20 files and 20 `sbatch` commands to run them all.

```
[me@biowulf ~]$ ls
master.R   run09      run18
run01      run10      run19
run02      run11      run20
run03      run12
run04      run13
run05      run14
run06      run15
run07      run16
run08      run17

[me@biowulf ~]$ more run01
#!/bin/sh
module load samtools
module load R
R --no-save < master.R --args nodeID=1
```

Submit all 20 jobs:

```
[me@biowulf ~]$ sbatch run01
6077144
[me@biowulf ~]$ sbatch run02
6077145
[me@biowulf ~]$ sbatch run03
6077147
.
.
.
[me@biowulf ~]$ sbatch run20
6077168
```

Check run status:

```
[me@biowulf ~]$ sjobs
User     JobId     JobName  Part  St  Reason  Runtime  Walltime  Nodes  CPUs  Memory    
========================================================================================
me       6077144   run01    norm  PD  ---        0:00   2:00:00      1     2  1GB/node                      
me       6077145   run02    norm  PD  ---        0:00   2:00:00      1     2  1GB/node                      
me       6077147   run03    norm  PD  ---        0:00   2:00:00      1     2  1GB/node                      
.
.
.
me       6077168   run20    norm  PD  ---        0:00   2:00:00      1     2  1GB/node                      
========================================================================================
cpus running = 0
cpus queued = 20
jobs running = 0
jobs queued = 20
```

Lots of output files:

```
[me@biowulf ~]$ ls | tail -n 5
slurm-6077163.out
slurm-6077164.out
slurm-6077165.out
slurm-6077166.out
slurm-6077168.out
```

## Many jobs with swarm

When we use `swarm`, we can consolidate all of our runs into one script.

```
[me@biowulf ~]$ ls
master.R
runAll.swarm

[me@biowulf ~]$ head -n 4 runAll.swarm
R --no-save < master.R --args nodeID=1
R --no-save < master.R --args nodeID=2
R --no-save < master.R --args nodeID=3
R --no-save < master.R --args nodeID=4
```

Job submission and tracking is much nicer!

```
[me@biowulf ~]$ swarm -f runAll.swarm --module samtools,R
6077234

[me@biowulf ~]$ sjobs
User     JobId            JobName  Part  St  Reason  Runtime  Walltime  Nodes  CPUs  Memory    
==============================================================================================
me       6077234_[0-19]   runAll   norm  PD  ---        0:00   2:00:00      1     2  1GB/node                      
==============================================================================================
cpus running = 0
cpus queued = 20
jobs running = 0
jobs queued = 20
```

We still get lots of output files. You may want to concatenate them all.

```
[me@biowulf ~]$ ls | tail -n 5
swarm_6077234_7.o
swarm_6077234_8.e
swarm_6077234_8.o
swarm_6077234_9.e
swarm_6077234_9.o

[me@biowulf ~]$ cat *.o > run6077234.o
[me@biowulf ~]$ cat *.e > run6077234.e
[me@biowulf ~]$ rm swarm_*
[me@biowulf ~]$ ls
master.R
run6077234.e
run6077234.o
runAll.swarm
```

## Many many jobs with swarm (bundling)

If you are running a lot of jobs (e.g. 10,000) using swarm, you will want to bundle them. This runs several of them sequentially in a single run on an assigned node. This saves the batch scheduler some work and results in fewer output files for you to deal with.

```
[me@biowulf ~]$ ls
master.R
runAll.swarm

[me@biowulf ~]$ wc -l runAll.swarm
10000 runAll.swarm

[me@biowulf ~]$ swarm -f runAll.swarm -b 50 --module samtools,R
6077356
```

This command will run 200 jobs, each consisting of 50 commands from `runAll.swarm`.

## Additional help

For additional help, see the [Biowulf web page](https://hpc.nih.gov/apps/swarm.html). They cover additional options and have some good examples.