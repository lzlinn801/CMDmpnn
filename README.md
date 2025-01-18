#####How to install GROMACS 2020.3 in Linux#####
###Preparation###
sudo apt install cmake
sudo apt-get install gcc
sudo apt-get install g++
sudo apt-get install python3-numpy
sudo apt install python3-pip
sudo pip install networkx==2.3
sudo apt-get install fftw3 fftw3-dev pkg-config

wget http://www.fftw.org/fftw-3.3.8.tar.gz
tar xzvf fftw-3.3.8.tar.gz
cd fftw-3.3.8
./configure --prefix=/Your Linux Name/fftw338 --enable-sse2 --enable-avx --enable-float --enable-shared
sudo make install

###Install GROMACS 2020.3###
wget https://ftp.gromacs.org/gromacs/gromacs-2020.3.tar.gz
tar xzvf gromacs-2020.3.tar.gz
cd gromacs-2020.3
mkdir build
cd build
sudo cmake .. -DCMAKE_INSTALL_PREFIX=/home/Your Linux Name/gmx2020.3 -DGMX_SIMD=AVX2_256 -DGMX_BUILD_OWN_FFTW=ON         ;If you use GPU, please enable it in this step
sudo make install
source /home/Your Linux Name/gmx2020.3/bin/GMXRC         ;Finally edit the environment and refresh

###Result analysis command statement###
gmx trjconv -s xxx.tpr -f xxx.xtc -o xxx_noPBC.xtc -pbc mol -center         ;Perform periodic correction
gmx trjconv -f xxx_noPBC.xtc -o trj_0-20ns -b 0 -e 20000 -dt 100         ;For the simulation results of the first 20 ns, a frame is extracted every 100 ps
gmx pairdist -f xxx_noPBC.xtc -s xxx.tpr -ref "atomnr A" -sel "atomnr B"         ;Calculate the change in distance between atoms A and B during the entire simulation
gmx cluster -f trj_0-20ns.xtc -s xxx.tpr -method gromos -o rmsd-clust.xpm -g cluster.log -dist rmsd-dist.xvg -cutoff 0.12 -clid clust-id.xvg -cl clusters.pdb -tu ns         ;The gromos method was used for cluster analysis
