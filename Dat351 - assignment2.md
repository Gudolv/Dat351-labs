# Lab 2 - Grid systems

### Background 

In this lab-assignment we'll explore different aspects of submitting jobs to a Grid system. We'll first set up credentials
for the ARC-system, then submitt several different jobs. Last we'll do similar tasks using the AliEn-system. 

### Tools 

To do the tasks we'll be connecting to systems using ARC and AliEn. To use these we'll be using ssh through the gateway 
pc eple.hib.no. 


### Setup 

Before being able to do the tasks in the assignment we need to set up our ARC credentials. For this lab, this only 
involves requesting a certificate and getting it signed. To do this we logged into **arcclient.hvl.no** with the command

```
grid-cert-request
```

into the terminal, then giving a pass-phrase. This creates three files in the **.globus** folder. Out of the three files
we are interested in the file named **usercert_request.pem** as this contains the request and it needs to be sendt to 
**gram.hvl.no** with:

```
scp ~/.globus/usercert_request.pem gram.hvl.no:/tmp/arc/usercert_request_${USER}.pem
```

Remembering to swap USER out with our username. (I also forgot the last _ in the command but it was luckily fixed). 
After its been approved, copy it back to the **.globus** folder and rename it to **usercert.pem**. We can check if the signature
is ok by running 

```
openssl verify $HOME/.globus/usercert.pem
```

This gave "OK" as a reply and we were good to run 

```
arcproxy
```

to create a proxy certificate, giving us access to do our tasks on the ARC network. 

### Task 1

The first task was to submit a job to the ARC-grid, the job was basically to write at script telling the grid to use 
the **echo** program native to linux (it sends what you type back to you). To do this we had to write a small job-script
telling the grid how to solve this task and saving it to a **xRLS** file: 

```

&  (executable="/bin/echo")
(arguments= "Hello" "World")
(jobname =  "helloworld" )
(stdout = "stdout")(stderr = "stderr")
(gmlog = "gmlog")

```

and sending it to be executed with: 

```
arcsub --computing-element=https://arcce.dat351/arex test1.xrls
```

To check the status of the job you can run the command **arcstat helloworld**, "helloworld" being the name of the job.
When the job is done running you can check the output, error output and save the result with 

```
arccat -o helloworld
arccat -e helloworld

arcget --dir=result helloworld
```

The output of this particular task is, no suprise, "Hello World" 


### Task 2

In this task we'll explore how to query the ARC grid and computing centers for information. 

** --computing-element** followed by an adress directs your query to an individual computing element.

**archer-manage** querys the ARC-grid information system. So if you for example want to find all endpoints in the ARC grid 
you would write **archery-manage -s archery:dat351 -o endpoints**.

Next we tried our hand on using the LDAP endpoints to find some information. We chose to utilize **ldapsearch** in the 
terminal of **eple.hib.no**. Typing:

``` 
ldapsearch -H "ldap://arcce.dat351:2135" -b "Mds-Vo-Name=local,o=grid" -x

```
and scanning through the output we tried finding the answer to these questions. 

First up was trying to find information on finished jobs: 


Second was to find how many CPUs each worker node had: 

It appears that they have 2 CPUs. 


Thirdly was to find what batch system the cluster was running: 


Next was finding what CPU and OS the cluster batch was using: 

The cluster has 2 CPUs and runs on CentOS-8.4.2015


Finaly finding how many CPUs that are available for the queue batch: 

The queue batch has 1 CPU and runs CentOS-8.4.2015


### Task 3

To specify the computing element, type of information endpoint and submission endpoint you would run the command:

```
arcsub --submission-endpoint-type=org.nordugrid,arcrest --info-endpoint-type=org.nordugrid.arcrest / 
--computing-element=https://arcce.dat351/arex test1.xrls

```

but I couldn't get this to run, getting an error. By running the below command, however, it ran just fine but I'm not 
sure if this is correct:

```
arcsub --computing-element=https://arcce.dat351/arex test1.xrls --submission-endpoint-type=arcrest --info-endpoint-type=arcrest 
```

### Task 4

**--registry=dat351 test1.xrls** tells arcsub to locate a computing element in the dat351 DNS domain, then submitting the 
job. This can be combined with **--info-endpoint-type** and **--submission-endpoint-type** that selects a computing 
center that supports those enpoints. Below is our attempt at doing this:

```
arcsub --info-endpoint-type=arcrest --submission-endpoint-type=arcrest --registry=dat351 test1.xrls
```


### Task 5

For this task we had to either use a premade C-program or make our own, compile it and submit it as a job. We tried to 
write a C++ variant that looked like this:  

```
#include <iostream>

using namespace std;

int main(int argc, char** argv){
    
    for (int i = 1; i < argc; i++){
        printf("%s ", argv[i]);

    }
    printf("\n");


}
```
saving it as my_echo.cpp and compiling it with **g++ -o my_echo my_echo.cpp**. Next was a job description program that 
looked like this:

``` 

& (executable = "my_echo")
  (arguments = "my" "echo")
  (jobname = "my_echo")
  (stdout = "stdout")
  (stderr = "stderr)
  (gmlog = "gmlog")
```

Saving this as **my_echo_job.xrls** and submiting the job with **arcsub my_echo_job.xrls**. After it had run its course, we 
retrived it with **arcget --dir=echoResult my_eco** and it's out put was as expected (my echo).

### Task 6 - Staging 

In this task we are testing file staging, which involves the transfer of input and output files across the grid. The 
goal is use a node in the grid as a remote compiler. To do this we must copy the source code from one grid node to 
another, where it will be compiled and then delete the source from the destination node. To do this we created a new 
job description file called staging1.xrls with: 

```
& (rsl_substitution = ("GRIDFTP" "gsiftp://gridftp.hvl.no"))
  (executable = "/usr/bin/gcc")
  (arguments = "-o" "my_echo" "my_echo.c"
  (jobname = "compile_echo")
  (stdout = "stdout")
  (stderr = "stderr")
  (gmlog = "gmlog")

  (inputFiles = ("my_echo.c" $(GRIDFTP)/home/tester/my_echo.c))
  (outputFiles = ("my_echo" "")
```

and submitting it with:

```
arcsub --registry=dat351 --info-endpoint-type=arcrest staging1.xrls
```

This, after some initial trouble, worked fine and with the **arcget** command we were able to retrive the compiled 
program. 

# AliEn 

For these tasks we switch from the ARC system to the AliEn system, and to use it we have to follow the same steps as we 
did for the ARC system. Creating a key, sending it, getting it approved and copying it back. 


### Task 7

AliEn only runs files found in specific directories, specificily the **/bin** folder. So, we created a small bash-program 
that returns the first prime larger than the given argument. We then started the AliEn shell with **aliensh** and uploaded 
the batch-program, after making the **/bin** folder. 

The program: 
```
#!/bin/bash

#read number 
number=$1

let x=$number+1
let y=$x+50

for ((x;x<=y;x++)) 
do
    let num_fac=`factor $x|wc -w`

    if [ $num_fac -eq 2 ]
    then
        echo $x
        break
    else
        continue
    fi
done
```

A small validation program that is also needed, for this task it was enough to make one that assumes that we are 
allways correct. 

```
#!/bin/bash

exit 0
```

To upload the programs we typed, changing the file name for each program: 

```
cp file:testprime.sh alien:bin/testprime.sh@HVLSLC6::ALIENSLC6::SEALIENSLC6 

```

To run the program we had to make a job description file, giving the data needed to run the job. We also had to create 
the folder "prime" to store the output.

```
Executable = "testprime.sh";
Arguments = "65566";
Validationcommand  = "bin/validate.sh";
Output = { "stdout,stderr" };
OutputDir = "prime";
```

### Task 8

The first thing we did in this task, was submitting the job made in task 7 with: 

```
submit testprime.sh
```

To follow the status of the job, we can use **ps** or **top**. After succsesfully running the job and verifying the output, we 
had to edit the job description file through the **aliensh** shell. To do this we type:

```
edit testprime.sh
```

Then we change the **Arguments** variable from 65566 to 65566548. Next we have to delete the output (or change the file name) 
to allow new results to be written. This seemed to work just fine and we got the new output stored in the prime folder. 
