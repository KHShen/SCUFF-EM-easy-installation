# How to install and use SCUFF-EM on cluster

**Author**: *Kunhong Shen*, **Date**: *July-25-2024*

Purdue rcac provides many different clusters. Usually, we have access to **Negishi**, **Bell**, and **Gilbreth** in Physics Department. Remember those clusters have an average lifetime of 6 years and they will retire in the future. Once you want to build SCUFF-EM on a new cluster, just follow the blew instructions. If you still find some problems, just ask **RCAC-HELP** for help.

## Installation

Here is the instruction from Official website of SCUFF-EM.

[Installation - SCUFF-EM documentation (homerreid.github.io)](https://homerreid.github.io/scuff-em-documentation/reference/Installing/)

Read it and find some useful tips.

As we cannot install external packages beyond your `${HOME}` folder, we need to load a proper compiler and its environment. Let's take **Negishi** as an example.

### Load intel compiler

Open a terminal in the RemoteDesk of any cluster.

```bash
module load intel
```

This command will load the intel compiler and Math Kernel Library (MKL). Check the user manual [RCAC - Knowledge Base: Negishi User Guide: Intel MKL Library (purdue.edu)](https://www.rcac.purdue.edu/knowledge/negishi/compile/intel_mkl) for details. The default C compiler is gcc. Every time you want to run SCUFF-EM after installation, run this command at the most beginning. 

### Install HDF5

The Hierarchical Data Format version 5 (HDF5), is an open source file format that **supports large, complex, heterogeneous data**. We don't need to understand how it works. Because the HDF group released version 1.14 may not support SCUFF-EM, we should find the legacy version like [1.12.3](https://hdf-wordpress-1.s3.amazonaws.com/wp-content/uploads/manual/HDF5/HDF5_1_12_3/src/hdf5-1.12.3.tar.gz). The downloaded file `hdf5-1.12.3.tar.gz` should first be unarchived by running:

```bash
cd ~/Downloads
tar -xvzf hdf5-1.12.3.tar.gz
# Then
cd hdf5-1.12.3
./configure --prefix=${HOME}/lib/hdf5-1.12.3
make -j 8 install
```

### Install libGDSII

```bash
cd ~/Downloads
git clone http://github.com/HomerReid/libGDSII
cd libGDSII
sh autogen.sh --prefix=${HOME}/lib/libGDSII
make -j 8 install
```

### Clone SCUFF-EM

```bash
cd ~
git clone https://github.com/HomerReid/scuff-em.git
```

### Configure its dependencies and install

```bash
cd scuff-em
# Compile without python and link INTEL mkl, self-compiled HDF5 and GDSII library
sh autogen.sh --without-python --prefix=${HOME}/scuff-em-installation CPPFLAGS="-I${HOME}/lib/hdf5-1.12.3/include -I${HOME}/lib/libGDSII/include -I${MKL_HOME}/include" LDFLAGS="-L${HOME}/lib/hdf5-1.12.3/lib -L${HOME}/lib/libGDSII/lib -L${MKL_HOME}/lib/intel64 -lmkl_intel_lp64 -lmkl_intel_thread -lmkl_core -liomp5 -lpthread"
# Wait for some time
make -j 8 install
```

Now, you have installed SCUFF-EM on a cluster.

### How to test it

For checking whether you have successfully install SCUFF-EM, follow

```bash
# Remember load intel compiler and its environment
module load intel
cd ~
# Add the location of the scuff-em binary executables to your execution path by running
export PATH=${PATH}:${HOME}/scuff-em-installation/bin
# Test scuff-analyze
scuff-analyze
```

You will some suggestions like below. Now you successfully install SCUFF-EM.

```bash
error: either --geometry or --meshfile option must be specified (aborting)

usage: scuff-analyze [options]

 options: 

  --geometry xx       (geometry file)
  --mesh xx           (mesh file)
  --meshfile xx       (mesh file)
  --PhysicalRegion xx (index of surface within mesh file)(-1)
  --TransFile xx      (list of transformations)
  --EPFile xx         (list of points)
  --WriteGnuplotFiles (write gnuplot visualization files)
  --WriteGMSHFiles    (write GMSH visualization files )
  --WriteGMSHLabels   (write GMSH labels)
  --Neighbors xx      (number of neighboring cells to plot)(0)
  --EEPs xx xx        (analyze equivalent edge pairs for surfaces xx, xx)
  --RegionVolumes     (compute volumes of closed regions)
  --WhichRegion xx xx xx (identify region in which the given point lives)
```

### Notes

Once more, the job file that is uploaded to the cluster should add the folloing command at the head to ensure the cluster can run it.

```bash
module load intel
export PATH=${PATH}:${HOME}/scuff-em-installation/bin
```

### Submit jobs to cluster

check the manual here.

[RCAC - Knowledge Base: Negishi User Guide: Running Jobs (purdue.edu)](https://www.rcac.purdue.edu/knowledge/negishi/run).
