---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "How to Write Batch Script for Ansys Fluent"
subtitle: ""
summary: ""
authors: ['admin']
tags: []
categories: []
date: 2020-06-16T10:22:52-06:00
lastmod: 2020-06-16T10:22:52-06:00
featured: false
draft: false

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []
---

When I first started out working on my Masters, there were very little
resources on how to run Ansys Fluent using a batch script for a "proper" HPC
machine, such as [Clemson's Palmetto
cluster](https://www.palmetto.clemson.edu/palmetto/). So I've decided to
document the script(s) that I developed over my time doing my Masters.

## Overview

The script I'll be overviewing was developed for the [PBS job scheduling
system](https://en.wikipedia.org/wiki/Portable_Batch_System), though it can
more than likely be changed to adapt to any job scheduling system. The script
I'll be discussing here is a combination of two different scripts that I have
developed. They can be found on my GitHub
[here](https://github.com/jrwrigh/msResearchTools/blob/master/clusterUtils/fluentSST.sh)
and
[here](https://github.com/jrwrigh/msResearchTools/blob/master/clusterUtils/fluentSBES.sh).

The architecture is based as a bash script. Fluent is controlled by a journal
file that is created by the script itself. The script is structured such
that all of the user inputs are clustered as high in the file as possible,
allowing them to A) be easily changed and B) quickly checked to make sure the
script will do what you want it to do. Without further ado, let's break down
this script.

The script does the following:

 1. Sets PBS directives and performs software setup
 2. Prints out useful information to the batch job output
 3. Takes in user input
 4. Manipulates data to be in a useful form (ie. getting resource numbers, job
    ID number, etc.)
 5. Creates the Fluent journal file
 6. Copies all required data to the nodes
 7. Runs Fluent
 8. Copies all the data back


```bash
#!/bin/bash
#PBS -N p136L0260_S050_SST
#PBS -l select=1:ncpus=28:mpiprocs=28:mem=32gb
#PBS -l walltime=04:00:00
#PBS -j oe
#PBS -m abe
#PBS -M [email address]

module purge
module add ansys/19.0
module add intel/17.0

set -o xtrace

echo ""
echo "#########################################################################"
echo "###                       SCRIPT FILE START                           ###"
echo "#########################################################################"
echo ""
cat $0
echo ""
echo "#########################################################################"
echo "###                        SCRIPT FILE END                            ###"
echo "#########################################################################"
echo ""

cd $PBS_O_WORKDIR

#####################################################################
#                         Parameters

casePath=p136L0260_S050_SST.cas
dataFileName=p136L0260_S050_SST

num_iterations=800

SWITCH_ROTATIONAL_VELOCITY=false
WRITE_CFDP_FILE=true
rotationalVelocity=0

# Fluent parameters
mpi=intel # MPI options are [ibmmpi, intel, openmpi, cray]
fluentType=3ddp

#####################################################################
#                     Name Wrangling

num_nodes=$(cat $PBS_NODEFILE | sort -u | wc -l)
tot_cpus=$(cat $PBS_NODEFILE | wc -l )
jobid_num=$(echo $PBS_JOBID | grep -Eo "[0-9]{3,}")

caseFile="$(basename $casePath)"
dataFileName=${jobid_num}_${dataFileName}
outFilePath="$PBS_O_WORKDIR/${dataFileName}.log"
journalFile="$jobid_num"_SST.jou
outDirName=$dataFileName

#####################################################################
#                     Fluent Journal File Logic

if $SWITCH_ROTATIONAL_VELOCITY; then
    rotationalVelocityParameter="/define/parameters/input-parameters/edit \"rotationalVelocity\"
    $rotationalVelocity"
else
    rotationalVelocityParameter=""
fi

if $WRITE_CFDP_FILE; then
    writeCFDPParameters="/file/export/cfd-post-compatible $dataFileName * () * ()
density
viscosity-lam
x-velocity
y-velocity
z-velocity
velocity-magnitude
x-wall-shear
y-wall-shear
z-wall-shear
pressure
absolute-pressure
dynamic-pressure
cell-volume
cell-volume-change
cell-wall-distance
curv-corr-fr
turb-kinetic-energy
turb-diss-rate
y-plus
()
yes
no "
else
    writeCFDPParameters=""
fi
#              ^^END Fluent Journal File Logic END^^
#####################################################################

#####################################################################
#                     Journal File Creation
cat <<EOT >$journalFile
/file/set-batch-options
; confirm file overwrite?
yes
; Exit on Error?
yes
; Hide Questions?
no
/file/read-case $caseFile
$rotationalVelocityParameter
/solve/monitors/residual/convergence-criteria
.00001
.00001
.00001
.00001
!date
report summary no
/server/start-server server_info.txt
/solve/iterate $num_iterations
!date
/file/write-data $dataFileName
$writeCFDPParameters
exit
yes
EOT
#              ^^END Journal File Creation END^^
#####################################################################

#####################################################################
#                     Building Fluent Arguements

fluent_args="-t${tot_cpus} $fluent_args -cnf=$PBS_NODEFILE"

fluent_args="-g -i $journalFile -mpi=$mpi $fluent_args"

#####################################################################
#                     Printing Job Information

echo "
+--------------------------+
|       START SOLVER       | 
+--------------------------+
+-------------------------------------+
|                                     | 
Start Time : $(date)
Job Information:
----------------
Num. CPUS = $tot_cpus
Num. Nodes = $num_nodes
Job ID # = $jobid_num
Inputs:
----------------
Journal File = $journalFile
Case File = $caseFile
Fluent Verison = $fluentType
Output Files:
-----------------
Data File = ${dataFileName}.dat
Log File = ${dataFileName}.log
Directories:
------------
PBS_O_WORKDIR = $PBS_O_WORKDIR
TMPDIR = $TMPDIR
Other:
-------
Nodes = $(uniq $PBS_NODEFILE)
|                                     | 
+-------------------------------------+
"

#####################################################################
#                     Running the Job Itself

for node in `uniq $PBS_NODEFILE`
do
	ssh $node "cp $PBS_O_WORKDIR/$caseFile $TMPDIR"
	ssh $node "mv $PBS_O_WORKDIR/$journalFile $TMPDIR"
done

cd $TMPDIR
printf "Contents of TMPDIR\n"
ls -la
printf "\n\n"

fluent $fluentType $fluent_args > $outFilePath

printf "Simulation Completed\n"

for node in `uniq $PBS_NODEFILE`
do
    ssh $node "mkdir $PBS_O_WORKDIR/$outDirName"
    ssh $node "cp -r $TMPDIR/* $PBS_O_WORKDIR/$outDirName"
done
```

## PBS Frontmatter and System-Specific Software Setup

```bash
#!/bin/bash
#PBS -N p136L0260_S050_SST
#PBS -l select=1:ncpus=28:mpiprocs=28:mem=32gb
#PBS -l walltime=04:00:00
#PBS -j oe
#PBS -m abe
#PBS -M [email address]


module purge
module add ansys/19.0
module add intel/17.0
```

I highlight both the PBS section and the software setup together as they'll be
the things you'll most likely have to change to make work for your application.
These PBS directives (`$PBS ...`) simply tell PBS information about the job,
such as what resources to use, how long it should run, and other useful
information. 

The lines starting with `module` are how the software environment is loaded.
`purge` is used to create a clean-slate environment, and then `add` is used to
add Ansys and the Intel library that Ansys depends on. 

For either of these sections, check with the documentation for your machines on
how to setup batch simulations. Even if you're computing system uses PBS or
`module`, there's a good chance that they're setup differently and possibly
require different options to run.

## Setup Useful Debugging Things
```bash
set -o xtrace

echo ""
echo "#########################################################################"
echo "###                       SCRIPT FILE START                           ###"
echo "#########################################################################"
echo ""
cat $0
echo ""
echo "#########################################################################"
echo "###                        SCRIPT FILE END                            ###"
echo "#########################################################################"
echo ""

cd $PBS_O_WORKDIR
```

Here, we setup some settings that are useful for debugging runs, or just
generally for the inevitable "Wait, what settings were used for these
results?" a few days/weeks/months after you run a simulation.

First, `set -o xtrace` is for the debugging side of things. It will print every
command that is run to the job output *before* it actually runs, with a `+`
prepended to the line. So if I have this script:

```bash
#!/usr/bin/env bash
set -o xtrace
echo "Hello World!"
```

the output will be:

```text
+ echo "Hello World!"
Hello World!
```

It will also expand variable values before it prints, so it's quite nice to
determine what exactly the script is doing when, how far the script has
advanced, etc.

The important part of the other section is the `cat $0`. For bash, the `$0`
variable stores the name of the currently running file. So this command simply
prints out the contents of this script. This is quite nice as it allows you to
know what state the script was in before it was run.

{{% alert warning %}} 
Note that I will sometimes have job scheduling systems that set `$0` to a value
that isn't the currently running script. I'm not sure why they do this, but be
forewarned that this may not work correctly. The next best thing to do is to
store the name of the script in the script and `cat` that instead of `$0`. This
has some robustness issues (it requires the user to update the name inside the
script if they change the name of the script itself), but it's better than not
tracking the script file.
{{% /alert %}}

And last but not least, `cd $PBS_O_WORKDIR` goes to the "original" directory,
which means the directory where the `qsub [jobScript]` was submitted at. This
is another PBS dependent parameter, but it is important as it will ensure that
path to the files is correct.

## User Parameters

These are where the use enters in information about the run they want to use.
This includes paths to required files, names for output files, options to be
set in the Fluent journal file, and more external settings that Fluent
requires.

```bash
#####################################################################
#                         Parameters

casePath=p136L0260_S050_SST.cas
dataFileName=p136L0260_S050_SST
```

These two variables set the path to the case file that Fluent with run and the
name of the data file that Fluent will output the results to. Note that the
`casePath` variable needs to be a path relative to the `$PBS_O_WORKDIR`. That
said, it can be a path, not just the name of a file. The `$dataFileName` acts
as a prefix for the actual name of the data file that will be output.

```bash
num_iterations=800

SWITCH_ROTATIONAL_VELOCITY=false
WRITE_CFDP_FILE=true
rotationalVelocity=0
```

These variables are responsible for settings within the Fluent journal file.
The lowercase variables (`$num_iterations` and `$rotationalVelocity`) will be
directly copied into the journal file. The all caps variables
(`$SWITCH_ROTATIONAL_VELOCITY` and `$WRITE_CFDP_FILE`) trigger if statements
later in the script that will add or remove chunks of Fluent journal script.
Skip forward to **Somewhere else!!!!!** to get more detail on that process and
to this other place for the journal file itself.

```bash
# Fluent parameters
mpi=intel # MPI options are [ibmmpi, intel, openmpi, cray]
fluentType=3ddp
```

Lastly, these two variables are actually for Fluent outside of the journal
file. They're added in with the Fluent command line options. `$mpi` obviously
controls the MPI type and `$fluentType` controls which version of Fluent to
run. In this case it is 3D (`3d`) double precision (`dp`). See Ansys
documentation for the available options

## Data Wrangling

Here, some of the outputs from PBS and user inputs are cleaned up such that
they can easily be used by the rest of the script.

```bash
#####################################################################
#                     Name Wrangling

num_nodes=$(cat $PBS_NODEFILE | sort -u | wc -l)
tot_cpus=$(cat $PBS_NODEFILE | wc -l )
jobid_num=$(echo $PBS_JOBID | grep -Eo "[0-9]{3,}")
```

These three are explicitly for cleaning up PBS outputs. The `$PBS_NODEFILE`
keeps track of which node have been assigned the job. The format of the file is
to repeat the name of each node for however many processes are allocated to it.
So if you had a job on 2 nodes using 2 cores each, the node file would look
like:

```text
node1
node1
node2
node2
```

For the number of processors, simply counting the number of lines in the node
file is sufficient. This is done using `wc` (or "**w**ord **c**ount") with the
"lines" flag `-l`. To count the number of nodes, we can do the same thing, but
eliminate the duplicated node names by first running `$PBS_NODEFILE` through
`sort -u` to only leave the unique lines.

Lastly, the `$PBS_JOBID` variable usually has some extra server information
appended to it, so this is removed using `grep`. All `grep` does is return the
any consecutive numbers longer than 3 digits. So we can turn
`1234567.pbs1.palmetto.clemson.edu` into just `1234567`.

```bash
caseFile="$(basename $casePath)"
dataFileName=${jobid_num}_${dataFileName}
outFilePath="$PBS_O_WORKDIR/${dataFileName}.log"
journalFile="$jobid_num"_SST.jou
outDirName=$dataFileName
```

Next we have some manipulation of the user inputs. First, we use `basename` to
get the actual file name of the case file set in `$casePath`. So
`$casePath=../caseDir/test.cas` turns into `$caseFile=test.cas`. Next, job ID
number is prepended to the name of the output data file is. This is then used
to set the file paths for the Fluent output log (`$outFilePath`) and the
directory that will store the results of the simulation (`$outDirName`).
Lastly, the name of the journal file is set by the PBS job ID number.

## Fluent Journal File Logic

These lines of the script use the all caps parameters set in the [user inputs section](#user-parameters).

```bash
#####################################################################
#                     Fluent Journal File Logic

if $SWITCH_ROTATIONAL_VELOCITY; then
    rotationalVelocityParameter="/define/parameters/input-parameters/edit \"rotationalVelocity\"
    $rotationalVelocity"
else
    rotationalVelocityParameter=""
fi

if $WRITE_CFDP_FILE; then
    writeCFDPParameters="/file/export/cfd-post-compatible $dataFileName * () * ()
density
viscosity-lam
x-velocity
y-velocity
z-velocity
velocity-magnitude
x-wall-shear
y-wall-shear
z-wall-shear
pressure
absolute-pressure
dynamic-pressure
cell-volume
cell-volume-change
cell-wall-distance
curv-corr-fr
turb-kinetic-energy
turb-diss-rate
y-plus
()
yes
no "
else
    writeCFDPParameters=""
fi
#              ^^END Fluent Journal File Logic END^^
#####################################################################

#####################################################################
#                     Journal File Creation
cat <<EOT >$journalFile
/file/set-batch-options
; confirm file overwrite?
yes
; Exit on Error?
yes
; Hide Questions?
no
/file/read-case $caseFile
$rotationalVelocityParameter
/solve/monitors/residual/convergence-criteria
.00001
.00001
.00001
.00001
!date
report summary no
/server/start-server server_info.txt
/solve/iterate $num_iterations
!date
/file/write-data $dataFileName
$writeCFDPParameters
exit
yes
EOT
#              ^^END Journal File Creation END^^
#####################################################################

#####################################################################
#                     Building Fluent Arguements

fluent_args="-t${tot_cpus} $fluent_args -cnf=$PBS_NODEFILE"

fluent_args="-g -i $journalFile -mpi=$mpi $fluent_args"

#####################################################################
#                     Printing Job Information

echo "
+--------------------------+
|       START SOLVER       | 
+--------------------------+
+-------------------------------------+
|                                     | 
Start Time : $(date)
Job Information:
----------------
Num. CPUS = $tot_cpus
Num. Nodes = $num_nodes
Job ID # = $jobid_num
Inputs:
----------------
Journal File = $journalFile
Case File = $caseFile
Fluent Verison = $fluentType
Output Files:
-----------------
Data File = ${dataFileName}.dat
Log File = ${dataFileName}.log
Directories:
------------
PBS_O_WORKDIR = $PBS_O_WORKDIR
TMPDIR = $TMPDIR
Other:
-------
Nodes = $(uniq $PBS_NODEFILE)
|                                     | 
+-------------------------------------+
"

#####################################################################
#                     Running the Job Itself

for node in `uniq $PBS_NODEFILE`
do
	ssh $node "cp $PBS_O_WORKDIR/$caseFile $TMPDIR"
	ssh $node "mv $PBS_O_WORKDIR/$journalFile $TMPDIR"
done

cd $TMPDIR
printf "Contents of TMPDIR\n"
ls -la
printf "\n\n"

fluent $fluentType $fluent_args > $outFilePath

printf "Simulation Completed\n"

for node in `uniq $PBS_NODEFILE`
do
    ssh $node "mkdir $PBS_O_WORKDIR/$outDirName"
    ssh $node "cp -r $TMPDIR/* $PBS_O_WORKDIR/$outDirName"
done
```
