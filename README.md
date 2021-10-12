# Code-Aster-MPI-in-Singularity-of-SM2021

In this tutorial we will show how to build the MPI Version of Code Aster 15.4 within the Singularity Container of Salome-Meca 2021
(a big thank you goes out to Ing. Nicola, a fellow member of the Code_Aster Forum. All the important work was done by him. Thank you! Of course we owe Salome-Meca and Code_Aster to EDF's R&D-Team, www.code-aster.org)

The following steps of this little tutorial were tried and tested on Ubuntu 20.04 LTS.
________________________________________________________________________________________________________
Install Singularity following the steps on https://singularity-tutorial.github.io/01-installation/:

sudo apt-get update

sudo apt-get install -y build-essential libssl-dev uuid-dev libgpgme11  libgpgme-dev squashfs-tools libseccomp-dev wget pkg-config git cryptsetup debootstrap
    
wget https://dl.google.com/go/go1.13.linux-amd64.tar.gz

sudo tar --directory=/usr/local -xzvf go1.13.linux-amd64.tar.gz

export PATH=/usr/local/go/bin:$PATH

wget https://github.com/singularityware/singularity/releases/download/v3.5.3/singularity-3.5.3.tar.gz

tar -xzvf singularity-3.5.3.tar.gz

cd singularity

./mconfig

cd builddir

make

sudo make install

_________________________________________________________________________________________________________
Download the Singularity Container of Salome-Meca 2021 from the Code_Aster Homepage www.code-aster.org:
https://www.code-aster.org/FICHIERS/singularity/salome_meca-lgpl-2021.0.0-1-20210811-scibian-9.sif

and place it in your ~/

This file is about 5.5G in size.

_________________________________________________________________________________________________________
Download the sources of Code_Aster 15.4 from //sourceforge.net/p/codeaster/src/ci/15.4.0/tree/ via pressing
the 'Download Snapshot' Button

Unzip and place this directory in /home/

Rename the directory to aster-src--> base directory of Code_Aster sources is now /home/aster-src

Copy above pkginfo.py in /home/aster-src/code_aster/ (only for cosmetical reasons, otherwise version number will be incorrect in SM)
_________________________________________________________________________________________________________
Add an overlay to the Singularity container, so data can be written to it:

cd ~

dd if=/dev/zero of=overlay.img bs=1M count=1000 && mkfs.ext3 overlay.img

singularity sif add --datatype 4 --partfs 2 --parttype 4 --partarch 2 --groupid 1 ~/salome_meca-lgpl-2021.0.0-1-20210811-scibian-9.sif overlay.img

The overlay.img is not needed anymore:
rm overlay.img

Your container file salome_meca-lgpl-2021.0.0-1-20210811-scibian-9.sif should now be +1G larger (approx. 6.5G)

_________________________________________________________________________________________________________
Open the container in a shell and bind /home to it:

sudo singularity run --bind  /home:/home -w ~/salome_meca-lgpl-2021.0.0-1-20210811-scibian-9.sif shell

Now execute the following commands to build an install the MPI-Version of Code-Aster 15.4 within the container (denoted by Singularity> prefix):

Singularity> export TOOLS="/opt/salome_meca/Salome-V2021-s9/tools"

Singularity> export ASTER_ROOT="${TOOLS}/Code_aster_15_4_0_mpi"

Singularity> cd /home/aster-src

Singularity> ./waf_mpi configure --prefix=${ASTER_ROOT} --disable-petsc --without-hg --install-tests --jobs=4 #(--without-hg is optional if you get an error without it, however we'll have to do further testing)

Singularity> ./waf_mpi build  --jobs=4

Singularity> ./waf_mpi install

Singularity> echo "vers : stable_mpi:/opt/salome_meca/Salome-V2021-s9/tools/Code_aster_15_4_0_mpi/share/aster" >> ${TOOLS}/Code_aster_frontend-2021001/etc/codeaster/aster

The last command makes the MPI-version accessible within Salome-Meca.
Now you should be able to launch Salome-Meca within the container, first exit the container shell with:

Singularity> exit
___________________________________________________________________________________________________________
Launch the Singularity Container of SM 2021 according to the tutorial given on https://www.code-aster.org/V2/spip.php?article303:

singularity run --app install salome_meca-lgpl-2021.0.0-1-20210811-scibian-9.sif

It is best to run it on a machine with an Nvidia Graphics Card, the more RAM and CPU-cores, the better. Without Nvidia Graphics Cards Software-Rendering will be used (slow 👎).

Now start Salome-Meca 2021 with:
./salome_meca-lgpl-2021.0.0-1-20210811-scibian-9

Salome Meca should launch. If you open 'Asterstudy' and click the 'History View' tab, you should be able to choose 'stable_mpi' in the 'version of code_aster' tab:

![Bildschirmfoto vom 2021-10-12 14-39-48](https://user-images.githubusercontent.com/89903493/136958409-a338627c-867f-4bc8-89d9-b0c707ff6135.png)


To achieve a minimum in calculation time we've found the following parameters to be most suitable with the MPI-version:

Number of MPI CPU = number_of_cores/2

Number of threads = 2

Please be aware, in systems with more than CPU, these numbers may not be suitable. 

In the sequential version (which you'll be likely not using anymore :-) ) the following parameters are most suitable:

Number of threads = number_of_cores/2

In any cases, especially on older CPUs, HyperThreading (Intel) or Simultaneous Multi-Threading (AMD) should be turned off. On a given CPU with an average number of cores (e.g. 8 or 10) a speed improvement of roughly 2.5-3 should be attainable in mechanical simulations with the MPI-version.

Have fun,

emefff.












