MPPNP-modular2 - INSTALLATION


1. Let's install OpenMPI from the website. Follow the instructions and test your MPI. I installed the openmpi-5.0.3 version in my /opt directory.
Installed tar file from https://docs.open-mpi.org/en/v5.0.x/installing-open-mpi/quickstart.html
```
$mkdir /opt/mpi
$tar xf openmpi-5.0.3.tar.bz2
$cd openmpi-5.0.3
$./configure --prefix=/opt/mpi 2>&1 | tee config.out
*incase of any missing libraries : sudo apt install libevent-dev libhwloc=dev
*sudo apt install build-essential
$ make -j 8 all 2>&1 | tee make.out
$ sudo make install 2>&1 | tee install.out
```
Dont forget to test :
```
$opt/mpi/bin/ompi_info
```

2.Next download the previous release, HDF5-1.8.3 from here. I am not sure if you can try the latest version but I didn't.
(https://support.hdfgroup.org/ftp/HDF5/prev-releases/hdf5-1.8/hdf5-1.8.3/src/)
Similarly,
```
$mkdir /opt/hdf5
$tar -xvzf hdf5-1.8.3.tar.bz2
$cd hdf5-1.8.3
$./configure --prefix=/opt/hdf5
$make 
$make check 
$sudo make install
```

Additional info from Joshua Issa: He encountered installation issues and found a solution. I've added it here in case others experience similar problems

**hdf5 step issues**
 - This might cause a problem with the configure step if the architecture of your machine is not recognized because of the config.guess and config.sub files may be out of date (welcome to the future!). You'll have to update config.guess and config.sub, and these specific ones worked for me with aarch64
   
```$wget http://savannah.gnu.org/cgi-bin/viewcvs/*checkout*/config/config/config.guess -O bin/config.guess```

```$wget http://savannah.gnu.org/cgi-bin/viewcvs/*checkout*/config/config/config.sub -O bin/config.sub```

- you want to do make install and then make check in that order

se step issues
- you might run into the same problem as the hdf5 step

```$wget http://savannah.gnu.org/cgi-bin/viewcvs/*checkout*/config/config/config.guess -O build-aux/config.guess```

```$wget http://savannah.gnu.org/cgi-bin/viewcvs/*checkout*/config/config/config.sub -O build-aux/config.sub```


- you want to do make install and then make check in that order


3. Next, we need to git clone NuPPN from the NuGrid Gitlab. However, currently, Gitlab requires your token to access git clone from the terminal instead of using your password.

a. Add ssh key to you gitlab account.
b. Then in your terminal,
```
$git clone https://gitlab.ppmstar.org/NuGrid/nuppn.git --branch modular2 --single-branch
```

Once you have NuPPN in your home directory, there are a few things that need to be modified.
save this in your .bashrc to export your mpi. Replaced the path for your installed mpi
```export PATH=$PATH:/opt/mpi/bin```

Please edit the NuPPN/frames/mppnp/source/makefile. Add the last line in the configuration of the SE with your path to HDF5 like this. The reason is that we want SE to be built with the same hdf5 version as the mppnp. Conflicts would occur if you have several hdf5 versions in your global paths:
```$(SE_PATH)/build/lib/libse.so:
rm -rf $(SE_PATH)
git clone https://github.com/NuGrid/NuSE.git $(SE_PATH)
mkdir $(SE_PATH)/SE/build
cd $(SE_PATH)/SE; ./configure --prefix=$(SE_PATH)/SE/build F77=gfortran --with-hdf5=/opt/hdf5; sudo make; sudo make install
```
Note: If you encounter problems installing the NUSE using the default makefile (which I did), I suggest installing it manually from Nugrid-NuSE into your home directory. If you change the location for the SE files, set the SE_PATH in the makefile to read your SE, too.
```
$git clone https://github.com/NuGrid/NuSE.git
$./configure --prefix=$(SE_PATH)/SE/build F77=gfortran --with-hdf5=/opt/hdf5
$sudo make
$sudo make install
```


4. Prepare your Make.local following the tutorial from here https://www.youtube.com/watch?v=9MAWjzhP3_M 

   The only thing I did differently was to assign
   ```BLASLIB = -lopenblas``` (you may need to sudo apt install blas first. Check where is your lopenblas is!else install blas, ```sudo apt install libopenblas-dev```)

This is an example of my Make.local 
```
# identifier for this local

LOCAL = wan18

#Architecture

ARCH = Linux_x86_64_gfortran

# make sure PHYSICS and SOLVER are absolute path names, for example

PPN = /home/wan18/nuppn

# you don't need to change the following two but you can
PHYSICS = $(PPN)/physics
SOLVER = $(PPN)/solver
UTILS = $(PPN)/utils

# where is mpi?
MPIHOME = /home/wan18/opt/mpi

USE_SUPERLU = YES
BLASLIB = -lopenblas
```

5.Make sure you have input files ready for testing. You can download my SE files to try https://drive.google.com/drive/folders/1pBOQO-9fIPi4TcpX3J5_wxXuYb1T3mcq?usp=sharing . Save these files in the USEEPP directory.(Update: the link is unusable.Email me if you need the input files)

These input files can be obtained from archives or evolution codes like MESA or GENEVA. Simply use NuGrid SE tools to generate the SE files. However form the SE MESA files, you'll have to manually create the index file on your own. Use the command below to generate the .index file in the HDF5 directory(MESA).
```
ls *se.h5  > name_files.idx
```
   
6. Important starts
 ```  
 cp -r run_template run_test
 cd run_test 
```  
   - Edit ppn_frame. Here are some inputs I made with the SE files (I tested the uploaded files, and they were corrupted. Please email me if you want to get input files). You can refer to the definitions in the documentation and change them later based on your work. ( Make sure you have ppn_physics,ppn_solver,ppn_frames, and isotopedatabase.txt in the run directory. Also, you want to check ifthe  NPDATA file exists in NuPPN/physics directory )
```
&ppn_frame
	iabuini = 11    ! initialisation
        iolevel = 1     ! how much output do you want, >4 is for debugging

        ini_filename = '/home/wan/NuPPN/frames/mppnp/USEEPP/iniab2.0E-02GN93.ppn'

        !sig_term_limit = 1d+10  ! upper limit for diffusion coefficients similar to that in MESA

        modstart = 1            ! start model for post-processing (check how many SE cycles you want to compute)
        modstop  = 2000         ! Assuming you try with my SE files input 

        igrid  = 2      ! set grid option
        dxm    = 1.d-2  ! grid size for static grid

        xmrmin = 0.     ! min mass coordinate for pp
        xmrmax = 30.0    ! max mass coordinate

        code_source = 'MES' ! which stellar code was used ?
        !datdir = '../USEEPP'
        datdir = '/home/wan/NuPPN/frames/mppnp/USEEPP/HDF5_mm30' #Path to the SE input files
        prefix = 'M30.00000Z0.014' #This refers to index file created before "name_files.idx"

        msl = 5000   ! maximum number of spatial zones
        nrefmax = 23 ! refinement level
        gfdim = 20   ! max num of refinement species

```

Finally, in you run_test directory
```
make distclean
make superclean
make clean
make blasclean
make
```
This will create the mppnp.exe file. If you encounter other errors, particularly the "Error: Missing actual argument for argument time...", it's likely due to two lines in mppnp.F90 (line 724 and 1610). For now, I've only commented out these lines as I haven't yet figured out how to solve them. Another problem can be due to openblas was not set properly. So check your make debug. I did get help from Marco Pignatari a lot!

**UPDATE** It seems that mppnp.F90 called a subroutine from solver/solver.F90 and decay.F90 which doesnt match with the 'time' argument. So I replaced the solver.f90 using the solver.f90 file from **NuPPN-modify_D_term** (From NuPPN -branch) which is almost identical except the 'time' dependencies, and finally the code works. The replacement was to remove the subroutine 'equilibrium check' in the solver module. I shared the replacement files here too. However keep in mind, this will make the PPN module encounter errors. I'm guessing the new routine, 'equilibrium check' is implemented for PPN and isn't ready for MPPNP yet.
Now 
 ```
mpirun -np 16 ./mppnp.exe
 ```

Another issue with running old Fortran code with the latest architecture is that you might encounter problems such as mismatched arguments. Please add the mismatched argument in the flags. Edit in ARCH makefile debug,

```-fallow-argument-mismatch```


If everything works out, you will see 3 output SE files. You are good to go to install mppnp in the HPC cluster to run them more efficiently. Ask your IT to install openMPI into your workspace. The rest is the same. Use any job scripts to send jobs to your HPC.
*A bit of a reminder to myself.I used this in Eddie HPC cluster, BLASLIB=-L/lib64/libopenblas then export OPENBLAS_NUM_THREADS=2 in the terminal to compile.This is because the setting limits the number of processes that can be run by a user in the Eddie cluster.
```
qsub run_eddie.sh !This is for Eddie. Replace with your HPC command and job script
```

```
#run_eddie.sh lines
#!/bin/sh
#Grid Engine options (lines prefixed with #$)
#$ -N MPPMP_test
#$ -cwd
#$ -l h_rt=23:00:00
#$ -l h_vmem=2G
#$ -pe mpi 64
#$ -R y
# These options are:
#  job name: -N
#  use the current working directory: -cwd
#  runtime limit: -l h_rt
#  memory limit: -l h_vmem
#  number of cores to use: -pe mpi
#  Resource Reservation: -R y

# Initialise the environment modules
. /etc/profile.d/modules.sh

# load modules,please load the module
#module load /exports/applications/modulefiles/SL7/Libraries/openmpi/1.10.1
#module load /exports/applications/modulefiles/SL7/Compilers/openmpi/1.10.1
module load intel

# Run the program
mpiexec -np 64 /exports/csce/eddie/ph/groups/np/Aisha/NuPPN/frames/mppnp/source/mppnp.exe
```

```
qstat !To check your status
qstat -u '*' !All jobs
```



