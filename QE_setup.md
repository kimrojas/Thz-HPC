# Quantum espresso setup in HPC

In this documentation, I will explain how to install and run Quantum Espresso in HPC. I will use the following software versions:

* Quantum Espresso 7.2
* Python
* Intel compilers
* Intel MPI

## Initializing the environment.

First, we need to load the neccessary modules. By doing `module avail`, we can see all the modules available in the system. For example, in ISSP supercomputer, we can see the following modules:


```bash
>$ module avail

--------------------------------------------- /home/local/env/modulefiles ----------------------------------------------
aocc/2.2.0       aocl/3.1.0-aocc-ilp64  oneapi_compiler/2023.0.0  openmpi/4.0.4-aocc-3.2.0
aocc/3.2.0       aocl/4.0-aocc          oneapi_mkl/2023.0.0       openmpi/4.0.4-gcc-10.1.0
aocc/4.0.0       aocl/4.0-aocc-ilp64    oneapi_mpi/2023.0.0       openmpi/4.1.5-aocc-4.0
aocl/2.2.0-aocc  arm_forge/20.1.1       oneapi_tbb/2023.0.0       openmpi/4.1.5-oneapi-2023.0.0
aocl/3.1.0-aocc  gcc/10.1.0             openmpi/4.0.4-aocc-2.2.0  openmpi/4.1.5-oneapi-2023.0.0-classic
```

In this case, we select  `oneapi_compiler/2023.0.0` and `oneapi_mpi/2023.0.0`, by doing:

```bash
>$ module purge
>$ module load oneapi_compiler/2023.0.0
>$ module load oneapi_mpi/2023.0.0
```

You can check the loaded modules by doing `module list`:

```bash
>$ module list

Currently Loaded Modulefiles:
 1) oneapi_compiler/2023.0.0   2) oneapi_mpi/2023.0.0
```


## Installing Quantum Espresso

Create a `install_script.sh` file with the following content:

```bash
#!/bin/bash

mkdir QE
cd QE

mkdir -p PSEUDO/gbrv
cd PSEUDO/gbrv
wget https://www.physics.rutgers.edu/gbrv/all_pbe_UPF_v1.5.tar.gz
tar -xvf all_pbe_UPF_v1.5.tar.gz
cd ../../

mkdir CODE
cd CODE
wget https://gitlab.com/QEF/q-e/-/archive/qe-7.2/q-e-qe-7.2.tar.gz
tar -xvf q-e-qe-7.2.tar.gz
cd q-e-qe-7.2
echo "--- Configuring QE 7.2 with Intel compilers and Intel MPI ---"  > install_7-2.log
./configure --enable-openmp --with-scalapack=intel  | tee -a install_7-2.log
echo "--- Compiling QE 7.2 with Intel compilers and Intel MPI ---"  >> install_7-2.log
make -j8 pwall | tee -a install_7-2.log
echo "--- COMPILATION DONE ---" >> install_7-2.log
cd ../../

echo "Add to jobscript: export PATH=$(realpath ./CODE/q-e-qe-7.2/bin):\$PATH"
echo "Add to jobscript: export ESPRESSO_PSEUDO=$(realpath ./PSEUDO/gbrv)"
```
