# Code-Aster-MPI-in-Singularity-of-SM2021

29.07.2022 Reupload of inadvertently deleted container (NO changes made to container, 2 links below changed slightly)

19.10.2021 update with new original container (version 2021.0.0). Now PETSc is also available.

13.10.2021 created

________________________________________________________________________________________________________________________________________________
Download container built according to recipe below at https://cloud.sylabs.io/library/emefff/collection/code-aster-mpi-in-singularity-of-sm2021

or pull it with

singularity pull --arch amd64 library://emefff/collection/code-aster-mpi-in-singularity-of-sm2021:salome2021 
 
Due to the naming conventions of cloud.sylabs.io please rename the downloaded container to 'salome_meca-lgpl-2021.0.0-0-20210601-scibian-9.sif'. Otherwise there is a conflict between the filename and the notes within the container.

________________________________________________________________________________________________________

In the following tutorial we will show how to build the MPI Version of Code Aster 15.4 within the Singularity Container of Salome-Meca 2021
(a big thank you goes out to Ing. Nicola, a fellow member of the Code_Aster Forum. All the important work was done by him. Thank you! Of course we owe Salome-Meca and Code_Aster to EDF's R&D-Team, www.code-aster.org)

The following steps of this little tutorial were tried and tested on Ubuntu 20.04 LTS (desktop and server version).

________________________________________________________________________________________________________
Install Singularity following the steps on https://singularity-tutorial.github.io/01-installation/:

Note: we assume that newer 3.x versions also work.

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

You may delete both tars now:

cd

rm go1.13.linux-amd64.tar.gz singularity-3.5.3.tar.gz

_________________________________________________________________________________________________________
Download the Singularity Container of Salome-Meca 2021 from the Code_Aster Homepage www.code-aster.org:
https://www.code-aster.org/FICHIERS/singularity/salome_meca-lgpl-2021.0.0-0-20210601-scibian-9.sif

and place it in your ~/

This file is about 5.5G in size.

_________________________________________________________________________________________________________
Download the sources of Code_Aster 15.4 from https://sourceforge.net/p/codeaster/src/ci/15.4.0/tree/ via pressing
the 'Download Snapshot' Button

Unzip and place this directory in /home/

Rename the directory to aster-src--> base directory of Code_Aster sources is now /home/aster-src

Copy above pkginfo.py in /home/aster-src/code_aster/ (only for cosmetical reasons, otherwise version number will be incorrect in SM)
_________________________________________________________________________________________________________
Add an overlay to the Singularity container, so data can be written to it:

cd ~

dd if=/dev/zero of=overlay.img bs=1M count=1000 && mkfs.ext3 overlay.img

singularity sif add --datatype 4 --partfs 2 --parttype 4 --partarch 2 --groupid 1 salome_meca-lgpl-2021.0.0-0-20210601-scibian-9.sif overlay.img

The overlay.img is not needed anymore:
rm overlay.img

Your container file salome_meca-lgpl-2021.0.0-0-20210601-scibian-9.sif should now be +1G larger (approx. 6.5G)

_________________________________________________________________________________________________________
Open the container in a shell and bind /home to it:

sudo singularity run --bind  /home:/home -w ~/salome_meca-lgpl-2021.0.0-0-20210601-scibian-9.sif shell

Now execute the following commands to build an install the MPI-Version of Code-Aster 15.4 within the container (denoted by Singularity> prefix):

Singularity> export TOOLS="/opt/salome_meca/Salome-V2021-s9/tools"

Singularity> export ASTER_ROOT="${TOOLS}/Code_aster_15_4_0_mpi"

Singularity> cd /home/aster-src

Singularity> ./waf_mpi configure --prefix=${ASTER_ROOT} --without-hg --install-tests --jobs=4 

Singularity> ./waf_mpi build --jobs=4

Singularity> ./waf_mpi install

Singularity> echo "vers : stable_mpi:/opt/salome_meca/Salome-V2021-s9/tools/Code_aster_15_4_0_mpi/share/aster" >> ${TOOLS}/Code_aster_frontend-2021001/etc/codeaster/aster

The last command makes the MPI-version accessible within Salome-Meca.
Now you should be able to launch Salome-Meca within the container, first exit the container shell with:

Singularity> exit
___________________________________________________________________________________________________________

Launch the Singularity Container of SM 2021 according to the tutorial given on https://www.code-aster.org/V2/spip.php?article303:

singularity run --app install salome_meca-lgpl-2021.0.0-0-20210601-scibian-9.sif

It is best to run it on a machine with an Nvidia Graphics Card, the more RAM and CPU-cores, the better. Without Nvidia Graphics Cards software-rendering will be used (=slow 👎).

Now start Salome-Meca 2021 with:
./salome_meca-lgpl-2021.0.0-0-20210601-scibian-9

Salome Meca should launch. If you open 'Asterstudy' and click the 'History View' tab, you should be able to choose 'stable_mpi' in the 'version of code_aster' tab:

![Bildschirmfoto vom 2021-10-12 14-39-48](https://user-images.githubusercontent.com/89903493/136958409-a338627c-867f-4bc8-89d9-b0c707ff6135.png)

If everything was successful, the Code Aster sources may be deleted:

sudo rm -r /home/aster-src

To achieve a minimum in calculation time we've found the following parameters to be most suitable with the MPI-version on an otherwise unoccupied machine (no other task with notable CPU usage):

Number of MPI CPU = number_of_cores/2

Number of threads = 2

Please be aware, in systems with more than one CPU, these numbers may not be suitable. 

In the sequential version (which you'll be likely not using anymore :-) ) the following parameters are most suitable:

Number of threads = number_of_cores/2

In any cases, especially on older CPUs, HyperThreading (Intel) or Simultaneous Multi-Threading (AMD) should be turned off. On a given CPU with an average number of cores (e.g. 8 or 10) a speed improvement of roughly 2-3 should be attainable in mechanical simulations with the MPI-version over the sequential version with OpenMP only. With older CPUs and if you are behind a strong firewall, you might also want to consider turning Spectre-Meltdown mitigations off to gain some more speed. 

This container also works on headless machines. It should then choose software rendering automatically. If you log on via ssh, choose -X option on the client side. The GUI will be forwarded to your client.

Have fun,

emefff.












