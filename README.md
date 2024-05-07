# MPPNP-modular2 - INSTALLATION
Installing MPPNP into your personal computer can be quite tricky due to the many dependencies that need to be set. The way I provide this instruction is to remind myself and help people step by step.

I made this resource on 5/7/2024.

First, make sure you have access and permission to NuGrid repositories.

Download and install the stable version of Ubuntu 18.04. You do not want the latest Ubuntu because the MPPNP code will not work with the latest compiler. I did try to compile with Ubuntu 22.04, but it gave so many errors. Well, you could bypass this error by adding this "-fallow-argument-mismatch" to FFLAGS in the MuPPN/frames/mppnp/source/ARCH/<your ARCH>/Makefile. but it still did not work well for me. So I used Ubuntu 18.04 to avoid the headache.
https://www.public-health.uiowa.edu/it/support/kb48549/

2.Downgrade the compilers (gcc, g++, and gfortran) in Ubuntu 18.04. By default, Ubuntu 18.04 runs on gcc-7, but it still won't compile the MPPNP code. I tried using gcc-4.8, g++-4.8, and gfortran-4.8, and it works. (Ubuntu 22.04 does not support the old compilers.)

To change the compiler in Ubuntu, follow these steps:
```
$sudo apt-get install gcc-4.8.5 g++-4.8.5
$sudo apt-get install gfortran-4.8
$sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 60
$sudo update-alternatives --install /usr/bin/g++ g++ /usr/bin/g++-4.8 70
$sudo update-alternatives --install /usr/bin/gfortran gfortran /usr/bin/gfortran-4.8 100
```
configure and choose your compiler.You may choose your compilers version here:
```
$sudo update-alternatives --config gcc 
$sudo update-alternatives --config g++
$sudo update-alternatives --config gfortran
```

Test: All must run with the same version
```
$gcc --version
$g++ --version
$gfortran --version
```

3.If all okay, then lets install OpenMPI from website.Follow the instruction and test your mpi.I installed openmpi-5.0.3 version in my /opt directory.
Installed tar file from https://docs.open-mpi.org/en/v5.0.x/installing-open-mpi/quickstart.html
```
$mkdir /opt/mpi
$tar xf openmpi-5.0.3.tar.bz2
$cd openmpi-5.0.3
$./configure --prefix=/opt/mpi 2>&1 | tee config.out
$ make -j 8 all 2>&1 | tee make.out
$ make install 2>&1 | tee install.out
```
Dont forget to test :
```
$opt/mpi/bin/ompi_info
```
save this in your .bashrc to export your mpi.
```export HDF5_MPI="ON"```


4.Next download the previous release, HDF5-1.8.3 from here. I am not sure if you can try the latest version but I didn't.
(https://support.hdfgroup.org/ftp/HDF5/prev-releases/hdf5-1.8/hdf5-1.8.3/src/)
Similarly,
```
$mkdir /opt/hdf5
$tar -xvzf hdf5-1.8.3.tar.bz2
$cd hdf5-1.8.3.tar.bz2
$./configure --prefix=/opt/hdf5
$make 
$make check 
$make install
```
Next, we need to git clone NuPPN from the NuGrid GitHub. However, currently, GitHub requires your token to access git clone from the terminal instead of using your password.

Go to your GitHub settings:
Developer settings
Personal access tokens
Access classic
Generate a new token.
Do not download manually from the source because some git clone repositories are hardwired in the NuPPN.git (NuSE).
Save that token as your password. Now, whenever you want to git clone anything, use this token instead
```
$git clone https://github.com/NuGrid/NuPPN.git) --branch modular2 --single-branch
```

Username: your username
Password : your token

Once you have NuPPN in your home directory, there are a few things that need to be modified.

Please edit the NuPPN/frames/mppnp/source/makefile. Add the last line in the configuration of the SE with your path to HDF5 like this:
```$(SE_PATH)/build/lib/libse.so:
rm -rf $(SE_PATH)
git clone git@github.com:NuGrid/NuSE.git $(SE_PATH)
mkdir $(SE_PATH)/SE/build
cd $(SE_PATH)/SE; ./configure --prefix=$(SE_PATH)/SE/build F77=gfortran --with-hdf5=/opt/hdf5; sudo make; sudo make install
```
Note: If you encountered problem installing the NUSE by the default makefile, I suggest to install it manually from Nugrid-NuSE into your home directory.Set the SE_PATH in the makefile to read your SE too.

2. Prepare your Make.local following the tutorial from here https://www.youtube.com/watch?v=9MAWjzhP3_M 

   The only thing I did differently was to assign
   ```BLASLIB = -lopenblas``` (you may need to sudo apt install blas first.Check where is you lopenblas is!)
4.Make sure you have input files ready for testing. You can download my SE files to try. Save these files in the USEEPP directory.

These input files can be obtained from archives or evolution codes like MESA or GENEVA. Simply use NuGrid SE tools to convert LOG history data into SE files. You'll need to manually create the index file on your own. Use command below to generate the .index file:
```
ls *se.h5  > name_files.idx
```
   
5. Important starts
 ```  
   - cp run_template run_test
   - cd run_test ( Make sure you have have ppn_physics,ppn_solver,ppn_frames in the folder )
```  
   - Edit ppn_frame.Here some inputs I made with the SE files.You can refer to the definitions in the documentation to change them later based on your work.( Make sure you have have ppn_physics,ppn_solver,ppn_frames,istopedatabase in the run directory.Also you want to check if NPDATA file exist in NuPPN/physics directory )
```
&ppn_frame
	iabuini = 11    ! initialisation
        iolevel = 1     ! how much output do you want, >4 is for debugging

        ini_filename = '/home/wan/NuPPN/frames/mppnp/USEEPP/iniab2.0E-02GN93.ppn'

        !sig_term_limit = 1d+10  ! upper limit for diffusion coefficients similar to that in MESA

        modstart = 1            ! start model for post-processing (check how many SE cycles you want to compute)
        modstop  = 79000

        igrid  = 2      ! set grid option
        dxm    = 1.d-2  ! grid size for static grid

        xmrmin = 0.     ! min mass coordinate for pp
        xmrmax = 3.0    ! max mass coordinate

        code_source = 'MES' ! which stellar code was used ?
        !datdir = '../USEEPP'
        datdir = '/home/wan/NuPPN/frames/mppnp/USEEPP/set1_update'
        prefix = 'M3.00Z.0200'

        msl = 10000   ! maximum number of spatial zones
        nrefmax = 23 ! refinement level
        gfdim = 20   ! max num of refinement species    
       ```
Finally, in you run_test directory
 ```make distclean
make superclean
make clean
make blasclean
make
 ```
This will create the mppnp.exe file. If you encounter other errors, particularly the "Error: Missing actual argument for argument time...", it's likely due to two lines in mppnp.F90 (line 724 and 1610). For now, I've only commented out these lines as I haven't yet figured out how to solve them yet. Another problem can be due to openblas was not set properly.So check your make debug. I did get help from Marco Pignatari lot!

Now 
 ```
mpirun -np 8 ./mppnp.exe
 ```
If everthing works out. You will see 3 output SE files. You are good to go to install mppnp in the HPC cluster to run them much effieciently. Ask your IT to install openMPI into your work space.The rest would be the same.Use any jobscripts to send job to your HPC
 ```
./jobs.ssh !replace with you job script
 ```




