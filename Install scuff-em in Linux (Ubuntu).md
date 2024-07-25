# Install SCUFF-EM in Linux (Ubuntu22.04)

**Author**: *Kunhong Shen*, **Date**: *July 2023*

## Note (WSL)

This tutorial also works for Windows Subsystem Linux (WSL). Just run the below command to install basic tools for compilation.

```bash
sudo apt-get install build-essential
```

I recommend to use Visual Studio Code with WSL extension to access WSL folder.

## For VirtualBox

**If you want to install a virtual machine on Windows operation system, watch the video by this link: [Install Ubuntu by VirtualBox](https://www.youtube.com/watch?v=x5MhydijWmc&ab_channel=ProgrammingKnowledge).**

[Installation of scuff-em](http://homerreid.github.io/scuff-em-documentation/reference/Installing/#2-installing-on-debianubuntu-based-linux-systems).

##### After you install Ubuntu, the next step is to install "git".

```bash
sudo apt-get install git
```

##### Then install external packages. (python3 is optional)

```bash
sudo apt-get install libopenblas-dev
sudo apt-get install libhdf5-openmpi-dev
sudo apt-get install gmsh
```

##### Download and install libGDSII

```bash
cd ~/Downloads
git clone http://github.com/HomerReid/libGDSII
cd libGDSII
sh autogen.sh --prefix=${HOME}/lib/libGDSII
make -j 8 install
```

**Here, 8 is the number of your CPU processor.** 

##### Clone the GitHub repository and build the code.

```bash
cd ~
git clone https://github.com/HomerReid/scuff-em.git
```

##### Configure scuff-em.

```bash
cd scuff-em
export CC=mpicc
export CXX=mpic++
sh autogen.sh --without-python --prefix=${HOME}/scuff-em-installation CPPFLAGS="-I/usr/include/hdf5/openmpi -I${HOME}/lib/libGDSII/include" LDFLAGS="-L/usr/lib/x86_64-linux-gnu/hdf5/openmpi -L${HOME}/lib/libGDSII/lib"
make -j 8 install
```

**You are all done. Follow the instruction online and test scuff-em.**

You can safely delete `./Downloads/libGDSII` and `./scuff-em`. They are downloaded by `git` command. In `./scuff-em-installation/examples`, you can find a lot of useful examples. 

##### Add the location of the scuff-em binary executables to your execution path by running

```bash
export PATH=${PATH}:${HOME}/scuff-em-installation/bin
```

**But this is a temporary method, when you close the terminal, the next time you open it and you need to run this line again.** The better way is to add these executables to `PATH`. Here shows how to do it:

1. Go to `$Home` directory and turn on 'Show Hidden Files' or press `Ctrl`+`H`.

2. Open `.profile` with **Text Editor**.

3. Scroll down to the end. Add
   
   ```bash
   # set PATH so it includes user's private bin if it exists
   if [ -d "$HOME/scuff-em-installation/bin" ] ; then
       PATH="$HOME/scuff-em-installation/bin:$PATH"
   fi
   ```

4. Log out current user and log in again. This allows the system to initialize the **bash** settings. Then everything is done.

## Geometry descriptions

## Notes

##### gmsh output file '*.msh' should follow the format of MSH file format version 2

```bash
gmsh -2 -clscale 0.3 <targetfile>.geo -format msh2 -2
```

**-2: 2D mesh, -clscale: fineness of meshing, the smaller of this value, the finer meshing you get. It is recommended that you customize the fineness of meshing at specific regions. That means you need to learn how to write down a .geo file by 'gmsh'**

## Geometric transformation

$\color{red}{\text{DO NOT COPY .trans file FROM WINDOWS!!!}}$

Especially for the multi-objects manipulation:

Common errors: 

1. unknown keyword ENDTRANSFORMATION
2. junk at end of line (aborting)

ALWAYS use Text Editor from Linux. (You can copy the text and paste, use Text Editor to create a .trans file)

The reason might be the difference of encoding formats between Linux and Windows. The text file should  obey the format:

- Character Encoding: UTF-8
- Line Ending: Unix/Linux

## Unit

For example, length and frequency:

The default units are $L_0=1\;\mu m$ for length and $\omega_0=3\cdot10^{14}$ rad/sec (=$c/L_0$) for angular frequency.

Others refer to [FAQ of General reference](http://homerreid.github.io/scuff-em-documentation/reference/FAQ/).

## Incident fields

Details can be found [here](homerreid.github.io/scuff-em-documentation/reference/IncidentFields/).

### Plane waves

SCUFF-SCATTER command-line syntax:

```bash
--pwDirection nx ny nz
--pwPolarization Ex Ey Ez
```

`--Polarization` may be complex numbers:

```bash
--pwDirection 0 0 1
--pwPolarization 0.7071 0.7071i 0.0
```

specify an incident field consisting of a circularly polarized place wave traveling in the positive z direction. 

Frequency `--omega` is defined elsewhere.

### Gaussian beams

SCUFF-SCATTER command-line syntax:

```bash
--gbCenter Cx Cy Cz
--gbDirection nx ny nz
--gbPolarization Ex Ey Ez
--gbWaist W
```

Selects the incident field to be a focused Gaussian beam, traveling in the direction defined by the unit vector **n**=`(nx,ny,nz)`, with **E**-field polarization vector **E**=`(Ex,Ey,Ez)`, beam center point with cartesian coordinates **C**=`(Cx,Cy,Cz)`, and beam waist `W`.

The values specified for `--gbPolarization` may be complex numbers.

### Point dipole sources

SCUFF-SCATTER command-line syntax:

```bash
--psStrength Px Py Pz
--psLocation xx yy zz
```

Selects the incident field to be the field of a pointlike electric dipole radiator with dipole moment **P**=`(Px,Py,Pz)` and located at cartesian coordinates (`xx,yy,zz`).

### Multi-fields

Here's an example of an incident-field file describing 9 different incident fields. If you specify this file using the `--IFFile` command-line option to scuff-scatter, then every calculation you request (scattered fields, power/force/torque, visualization, etc.) will be done 9 times at each frequency, once for each field.

```
EX    PW    0  0  1       1          0          0
EY    PW    0  0  1       0          1          0
LC    PW    0  0  1       0.7071     0.7071i    0
RC    PW    0  0  1       0.7071    -0.7071i    0

PS1   PS    1.1 2.2 3.3   0.4+0.5i   0.7        -0.8
PS2   MPS   1.1 2.2 3.3   0.4+0.5i   0.7        -0.8

GB    GB    0.0 0.0 0.0   0.0 0.0 1.0    1.0 0.0 0.0    0.5

COMPOUND1
  PW    0   0  1      1        0        0
  PS    1.1 2.2 3.3   0.4+0.5i 0.7      -0.8
END

COMPOUND2
  PW    0   0  1      0        1        0
  PS    1.1 2.2 3.3   0.4+0.5i 0.7      -0.8
END
```

Here's how to understand the 9 incident fields described by this file.

- The first several lines define various types of incident fields in which there is only a single field source. For this type of incident field, the first word on the line is an arbitrary user-specified label (such as `EX` or `PS1`) that will be used to identify data corresponding to this incident field in output files. The second word on the line is one of the four keywords `PW|PS|MPS|GB`. The remainder of the line consists of numerical parameters:
  - For plane waves (keyword `PW`) there are 6 numerical parameters: `nx ny nz Ex Ey Ez`. In the example above, `EX` and `EY` are linearly-polarized plane waves, while `LC` and `RC` are left- and right-circularly polarized waves.
  - For electric-dipole point sources (keyword `PS`) there are 6 numerical parameters: `xx yy zz Px Py Pz`.
  - For magnetic-dipole point sources (keyword `MPS`) there are 6 numerical parameters: `xx yy zz Mx My Mz`.
  - For gaussian beams (keyword `GB`) there are 10 numerical parameters: `Cx Cy Cz nx ny nz Ex Ey Ez W`.
- The final two sections of the file describe *compound* fields---that is, incident fields produced by more than one source acting simultaneously. These types of fields are described by putting their label (here `COMPOUND1` or `COMPOUND2`) on line by itself, then specifying as many `PW|PS|MPS|GB` lines as you like, and finally closing the compound-field definition with the `END` keyword. For example, the field we labeled `COMPOUND1` consists of a planewave acting simultaneously with the field of a point source.

## Command-line syntax (scuff-scatter)

[Link](http://homerreid.github.io/scuff-em-documentation/applications/scuff-scatter/scuff-scatter/).
