# Dat 351 - Assignment 1

## Bakground and Tools 

The tools that will be used and referenced in this report are:

**SSH** - ssh stands for Secure Socket Shell and is a network protocol that allows
you to securely access and communicate with remote machines. Usualy used in a 
terminal. 

**HTCondor**, **TORQUE** and **Slurm** are, in the context of this lab, software
used to manage workloads on clusters of computers.

**eple.hib.no** is the gateway that allows us to connect to the computer grid 
through the internet. 

## Setup

All tasks for this assignment have been done using SSH on a linux machine 
running **Endeavour OS**, connecting to **eple.hib.no** through ssh and gaining access 
to the computer grid. The grid consists of a small cluster consisting of 10 
computers running CentOS7 and CentOS8 on virtual machines.  

## Tasks and goals

The goal of this lab is to learn the basics of submitting a job to a computer 
resource through a local scheduler. This entails writing some bash scripts,
submitting these to the intended targets and saving the output. 


## Process 

Since we are connecting through **eple.hib.no** we had to first create a key 
using the simple command ``ssh-keygen `` in the terminal. This key was then 
approved and we gained access to eple. From here we could connect to the 
different machines.

The computer cluster is set up to use a shared filesystem and a shared password
for database login, thus we had to pick one of 20 preordained users and change 
the password. For this a simple ``passwd`` in the terminal was enough. 

The next task was to enable moving files between nodes. This is required for 
**TORQUE** to be able transfer files using **rcp**. To allow **rcp** to move files we 
created a file in the home-directory called .rhosts. What node this file is 
created on does not matter. In linux this can be done with ``touch .rhost``.
In the assignment it says only to add the **TORQUE** computers but we only managed 
to get this to work by adding all the computers. 

![Example of .hosts](https://github.com/Gudolv/Dat351-labs/blob/main/Screenshots/Oblig1/hosts.png)

After the **.rhost** whas created and contained the necessary text we ran 
``rcp /tmp/afile.txt torquew1:/tmp ``. This copied the file from the current node
to the target **torquew1**. 

![Files moved](https://github.com/Gudolv/Dat351-labs/blob/main/Screenshots/Oblig1/movefilesux.png)

##Slurm 

Next it was time to try out Slurm. Jobs has to be submitted from a computer 
called **slurmmaster**. To submitt a job to Slurm you need to use the ``sbatch``
command. It takes a batch script as an argument and assings resources to the 
task at hand. This batch script uses the ``srun`` command to run a selection of 
programs. After checking what nodes and partitions that are available with 
``sinfo`` and tryinga small script, we created a job called **myfirstjob.sh**. It looked 
like this:
```bash

#!/bin/bash

sleep 60
data
```
![First slurm](https://github.com/Gudolv/Dat351-labs/blob/main/Screenshots/Oblig1/slurmsuc.png)

This was run with the batch scrip runmyfirstjob.sh, after making it executable 
with chmod u+x myfirstjob, and looked like this:

```bash

#!/bin/bash

#SBATCH --job-name=MyFirstJob
#SBATCH --output=%x%j.out
#SBATCH --error=%x%j.err 

srun myfirstjob.sh

```
%x is standin for the job-name and %j is standin for the job id. This resulted 
in the creation of two files, MyFirstJob505.out and MyFirstJob505.err. 

![Slurm](https://github.com/Gudolv/Dat351-labs/blob/main/Screenshots/Oblig1/myfirstjob.png)

Next up was testing the **scancel** command. This is used to cancel jobs and only
works for your personal jobs. 

![Cancel](https://github.com/Gudolv/Dat351-labs/blob/main/Screenshots/Oblig1/cancelslurm.png)

The next step in the assignment was to compile a small C-program and submitt a 
job to run the program. We used the code provided in the assignment, compiled it
with **cc**. called it **pi** and tested it localy with **./pi**. Then we proceded
to follow the same procedure mentioned above, creating a small batch script to 
run the program, the output was:

![PI](https://github.com/Gudolv/Dat351-labs/blob/main/Screenshots/Oblig1/pijobsuc.png)

1**# of trials= 100000000 , estimate of pi is 4**

## HTCondor

To submit the job to HTCondor we had to log into one of the condor machines, 
make a small script that looked like this:

```bash

##HTCondor submit-file for pi job 

Universe=vanilla
Executable = pi 
Log = pi_HTC.out 
Output = pi_HTC.out
Error = pi_HTC.error

Queue 
```
![Condor](https://github.com/Gudolv/Dat351-labs/blob/main/Screenshots/Oblig1/condorjobproc.png)

We chose vanilla as universe as this is the recommended environment for running
compiled languages. It was submitted using **condor_submit condor_job**, this 
succeded and we got the same output as with HTCondor. You can check the queue 
with **condor_q**.

![Result](https://github.com/Gudolv/Dat351-labs/blob/main/Screenshots/Oblig1/pihtcout.png)

## TORQUE

To submit one has to ssh into the machine called **torquemaster**. One can use 
**qsub** to submit the jobscript. We did this using a script that looked like 
this:
```bash

#!/bin/bash
#PBS -k o 
#PBS -N student7torquejob
#PBS -j oe

cd $HOME/

./pi 
```
![Torque](https://github.com/Gudolv/Dat351-labs/blob/main/Screenshots/Oblig1/torquesuc.png)

One can check the status of the job using **qstat**, running the script gave the 
same output as the two other attempts at running the pi-program. 

## Running with and without #!bin/bash 

Running the script **torque_job_nob.sh** thats exactly the same as the one above 
without the #!/bin/bash runs just fine on **TORQUE** but running the same script
that worked fine with **Slurm** earlier after removing **#!/bin/bash** gets an error. 

I assume that the difference lies in the way that jobs are submitted, for **Slurm** 
you submitt a script that creates a fork, then executes the job while in **TORQUE** 
you submitt the job directly and thus don't need to specifiy that you want it ran 
as a bash script. 


