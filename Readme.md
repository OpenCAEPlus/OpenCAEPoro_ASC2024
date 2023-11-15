# OpenCAEPoro Installation Guide

This guide provides detailed instructions for installing and configuring OpenCAEPoro, a software package for numerical simulations of multi-component fluid flows in porous media.

## Requisite

This version of OpenCAEPoro requires the following external open-source libraries, which are also included in this repository:

- **lapack-3.11:** Provides linear algebra routines.
- **parmetis-4.0.3:** Offers parallel graph partitioning algorithms.
- **hypre-2.28.0:** Supplies high-performance preconditioners and solvers.
- **petsc-3.19.3:** A suite for solving partial differential equations.
- **petsc_solver:** Custom solver built on top of PETSc.


## Recommended Environment

For smooth installation and optimal performance, we recommend using the Intel compiler (e.g., oneAPI-2022.1 version), gcc 7.3.0 and cmake 3.17.

## Installation Steps

First, unzip all software packages. Then, follow these steps to install them in the specified order:

```[bash]
1. build lapack:
    S1. cd lapack-3.11
    S2. make blaslib
    S3. make cblaslib
    S4. make lapacklib
    S5. make lapackelib

2. build parmetis:
    S1. cd parmetis-4.0.3
    S2. modify the path in script "build-parmetis.sh", such as "make config
        cc=mpiicc prefix=ROOT_DIR/parmetis-4.0.3/parmetis-install"
    S3. sh build-parmetis.sh

    ROOT_DIR is the root directory of the repository, which is set by the user
    according to their own directory (similarly hereinafter).

3. build hypre
    S1. cd hypre-2.28.0
    S2. modify the path in script "build-hypre.sh", such as 
        "./configure --prefix=ROOT_DIR/hypre-2.28.0/install" --with-MPI --enable-shared"
    S3. sh build-hypre.sh

4. build petsc
    S1. cd petsc-3.19.3
    S2. modify the path in script "build-petsc.sh", such as
        export PETSC_DIR=ROOT_DIR/petsc-3.19.3
        export PETSC_ARCH=petsc_install
        ./configure CC=mpiicc CXX=mpiicpc \
        --with-fortran-bindings=0 \
        --with-hypre-dir=ROOT_DIR/hypre-2.28.0/install \
        --with-debugging=0 \
        COPTFLAGS="-O3" \
        CXXOPTFLAGS="-O3" \
        make -j 20 PETSC_DIR=ROOT_DIR/petsc-3.19.3 PETSC_ARCH=petsc_install all
    S3. sh build-petsc.sh  

5. build petsc_solver
    S1. cd petsc_solver
    S2. modify the path in script "build-petscsolver.sh", such as
        export CPATH=ROOT_DIR/lapack-3.11/CBLAS/include:ROOT_DIR/lapack-3.11/LAPACKE/include:$CPATH
        export LD_LIBRARY_PATH=ROOT_DIR/lapack-3.11:$LD_LIBRARY_PATH
    S3. modify the path in file "CMakeLists" on lines 18 and 19
        set(PETSC_DIR "ROOT_DIR/petsc-3.19.3/")
        set(PETSC_ARCH "petsc_install")
    S4. sh build-petscsolver.sh

6. build OpenCAEPoro
    S1. cd OpenCAEPoro
    S2. modify the path in script "mpi-build-petsc.sh", such as
        export PARMETIS_DIR=ROOT_DIR/parmetis-4.0.3
        export PARMETIS_BUILD_DIR=ROOT_DIR/parmetis-4.0.3/build/Linux-x86_64
        export METIS_DIR=ROOT_DIR/parmetis-4.0.3/metis
        export METIS_BUILD_DIR=ROOT_DIR/parmetis-4.0.3/build/Linux-x86_64
        export PETSC_DIR=ROOT_DIR/petsc-3.19.3
        export PETSC_ARCH=petsc_install
        export PETSCSOLVER_DIR=ROOT_DIR/petsc_solver
        export CPATH=ROOT_DIR/petsc-3.19.3/include/:$CPATH
        export CPATH=ROOT_DIR/petsc-3.19.3/petsc_install/include/:ROOT_DIR/parmetis-
        4.0.3/metis/include:ROOT_DIR/parmetis-4.0.3/include:$CPATH
        export CPATH=ROOT_DIR/lapack-3.11/CBLAS/include/:$CPATH
    S3. sh mpi-build-petsc.sh
```

## Testing the Installation

After installation, you can test the setup by running the following command in the terminal under the OpenCAEPoro main directory:

`mpirun -n p ./testOpenCAEPoro ./data/test/test.data`

Replace `p` with the number of processes you want to use. Check the output on the screen and the newly generated files in `./data/test/`. You should find files like `SUMMARY.out` and `FastReview.out`. If you run this command with more than one process, `statistics.out` will also be generated. Use `SUMMARY_ref.out` as a reference to verify the results.
