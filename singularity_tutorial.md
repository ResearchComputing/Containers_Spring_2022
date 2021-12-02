# Exercise: Running Containers with Singularity on RMACC Summit

## Logging into RMACC Summit

#### _Option 1:_ If you are using a temporary account: In a terminal or Git Bash window, type the following:
```
ssh -X userNNNN@tlogin1.rc.colorado.edu
Password: <temporary-password-you-received-from-me>
```
_Note that `userNNNN` is a temporary account that I have assigned to you (where "NNNN" is a number like "0012"). If you do not have a temporary account or password, let me know._

#### _Option 2:_ If you are using your regular RC account (e.g., ‘monaghaa’ for me), log in as you normally would, e.g.:
 ```
ssh -X monaghaa@login.rc.colorado.edu
```

### Getting on a compute node

#### Navigate to scompile
```
ssh scompile
```

#### Load singularity and set up environment
```
module load singularity/3.6.4
export SINGULARITY_TMPDIR=/projects/$USER
export SINGULARITY_CACHEDIR=/projects/$USER 
```

#### Change to your `/projects` directory and make a sandbox directory you can experiment in:
```
cd /projects/$USER 
mkdir sandbox
cd sandbox
```

## Running a container from Singularity Hub

#### Pull an existing container image that someone else posted:
```
singularity pull --name mytranslator.sif shub://monaghaa/mytranslator
```

#### Now run it:
```
singularity run mytranslator.sif
```

#### Now look at the script inside the container that is executed when you use _`singularity run`_:
```
singularity inspect --runscript mytranslator.sif
```

You'll notice that the runscript calls a script in `/opt` called `text_translate.py`. 

#### Next, let's make a writable _local_ copy of `text_translate.py` in our present working directory and edit it to change the output language from German to French:

_first, copy the file_
```
singularity exec mytranslator.sif cp /opt/text_translate.py .
```
_second, edit the file (and change output language to French. You can use the "nano" editor. Type cntl^x to exit and save when done_
```
nano text_translate.py
```
_Finally, execute your newly modified _local_ copy of the script_
```
singularity exec mytranslator.sif python ./text_translate.py
```
You just used the containerized version of python to run a local version of a python script.

 #### We can also invoke a simple python shell from the container:
```
singularity exec mytranslator.sif python
```

Type `exit()` to exit python.

#### Now shell into the container and look around:
```
singularity shell mytranslator.sif
```

Try typing _`ls /`_. What directories do you see?:
 
Now exit the container by typing `exit`

## Binding directories to a container

On Summit, most host directories are “bound” (mounted) by default. But on other systems, or in some instances on Summit, you may want to access a directory that is not already mounted. Let’s try it.

Note that the “/mnt" directory in ”mytranslator.sif” empty. But say we want to bind the Summit ”/opt” directory to it...  

#### Let’s bind it:
```
singularity shell --bind /opt:/mnt mytranslator.sif
```

Now from within the container type "ls -l /mnt" and see if it matches what you see from the outside of the container if you type the same thing. When you are done, type `exit` to get out of the container.


_Note: If your host system does not allow binding, you will need to create the host directories you want mounted when you build the container (as root on, e.g., your laptop)_

## Running an MPI container

MPI-enabled Singularity containers can be deployed on RMACC Summit, with the caveat that the MPI software within the container must be consistent with MPI software available on the system. This requirement diminishes the portability of MPI-enabled containers, as they may not run on other systems without compatible MPI software. Regardless, MPI-enabled containers can still be a very useful option in many cases.   

Here we provide an example of that uses a gcc compiler with OpenMPI.  

#### Let’s pull the MPI container from Singularity Hub first:

```
singularity pull hello_openmpi.sif shub://monaghaa/hello_openmpi_summit
```

In order to use it, we load the gcc and openmpi modules on Summit (the openmpi version is consistent with the openmpi version installed in the container).

#### Load the gcc and openmpi modules:
```	
module load gcc/6.1.0
module load openmpi/2.0.1
```

#### Now run the container. Note that you preface the command with _`mpirun –n <numprocs>`_:

```
mpirun -n 2 singularity exec hello_openmpi.sif mpi_hello_world
```

You should see 2 processes, one with rank '0' and the other with '1', indicating that this MPI program is working propery.

That concludes tutorial on running containers with Singularity.  Now return to the [course slides](https://github.com/ResearchComputing/CONTAINER_TUTORIAL_FALL_2020/blob/main/Containers-Pt2-Fall2020.pdf) and we will finish out this mini course with some commentary and demos on building containers with Singularity. 
