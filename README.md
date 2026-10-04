# Install invDFT

These install scripts provide a set of executable
functions that install the necessary dependencies
of invGKS branch of [invDFT](https://github.com/dftfeDevelopers/invDFT.git)
on NERSC Perlmutter.

To use these scripts, we assume you have cloned this
repository onto a system where you intend to install invDFT.
For example, I installed it into `$PSCRATCH/install_DFTFE` after 
cloning into the scratch directory

    cd $PSCRATCH
    git clone https://github.com/dftfeDevelopers/installationForInvGKS.git install_invDFT
    cd install_invDFT

## Pre-requisites

Because it's a better shell, the scripts are written
in the [rc](http://doc.cat-v.org/plan_9/4th_edition/papers/rc)
shell language.  Install `rc` by running

    cp src/rcrc $HOME/.rcrc
    . ./bin/getrc.sh $HOME/$LMOD_SYSTEM_NAME
    rc -l

Note that getrc installs into the `$HOME/$LMOD_SYSTEM_NAME/bin`
directory, and adds that to your PATH. Also note that in rc shell, the 
`export` keyword is not used when setting environment variables.

Copying the rcrc startup file to your home directory provides
the module command (in case your lmod version is old,
and doesn't yet recognize the rc shell).

## Module Environment

The module environment intended to run invDFT has been extracted
into `env2/env.rc`.  Edit this file before proceeding any further.
Make sure that your module environment contains some version of the
pre-requisites mentioned there. The above environment file is used both by the install and run
phases of invDFT.

## Running the installation
The installation itself is contained within the functions in
`installInvDFT.rc`.  Source this script using

    . ./installInvDFT.rc

and then run the functions listed in that file manually, in order.
For example, 

    install_blis
    install_libflame
    install_alglib
    install_libxc
    install_spglib
    install_p4est
    install_scalapack
    install_elpa
    install_kokkos
    install_petsc
    install_slepc
    install_numdiff
    install_dealii
    compile_dftfe
    compile_invDFT_invGKS 

Each function follows a standard pattern - download source into `$WD/src`,
patch, compile, and install into `$INST`.  It is HIGHLY recommended
to check all warnings and errors from these installs to be sure
you have not ended up with broken packages.


## Running invDFT


Assuming you have already sourced `env2/env.rc`, an example
batch script running GPU-enabled invDFT on 2 nodes is below:

    #!/bin/bash
    #SBATCH -A m2360_g
    #SBATCH -C gpu
    #SBATCH -q regular
    #SBATCH --job-name test_invDFT
    #SBATCH -t 00:10:00
    #SBATCH -n 8
    #SBATCH --ntasks-per-node=4
    #SBATCH -c 32
    #SBATCH --gpus-per-node=4
    #SBATCH --gpus-per-task=1
    #SBATCH --gpu-bind=none


    export SLURM_CPU_BIND='cores'
    export OMP_NUM_THREADS=1
    export MPICH_GPU_SUPPORT_ENABLED=1


    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$WD/env2/lib
    export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$WD/env2/lib64
    export BASE=$WD/src/dftfe/build/release/real

    srun  $BASE/dftfe dftfe_parameterFile.prm invDFT_parameterFile.prm > output
