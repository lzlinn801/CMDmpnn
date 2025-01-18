Please Use The Correct Reading Mode
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

###Protein-ligand MD simulation###
##Put all the .mdp files, protein.pdb, ligand.gro and ligand.itp files together and run terminal##
gmx pdb2gmx -f PROTEIN.pdb -o processed.gro -p topol.top -ignh         ;select AMBER14SB and SPC
##Constructing protein-ligand complex topology files##
gmx editconf -f processed.gro -o newbox.gro -c -d 1.0 -bt cubic
gmx solvate -cp newbox.gro -cs spc216.gro -o solv.gro -p topol.top
gmx grompp -f ions.mdp -c solv.gro -p topol.top -o ions.tpr -maxwarn 1
gmx genion -s ions.tpr -o ions.gro -p topol.top -pname NA -nname CL -neutral
gmx grompp -f em.mdp -c ions.gro -p topol.top -o em.tpr -maxwarn 1
gmx mdrun -v -deffnm em         ;Use the steep method to perform the first energy minimization
gmx grompp -f minim.mdp -c em.gro -p topol.top -o em2.tpr -maxwarn 1
gmx mdrun -v -deffnm em2         ;Use the cg method to perform the first energy minimization
gmx genrestr -f LIGAND.gro -o posre.itp -fc 1000 1000 1000
gmx make_ndx -f em2.gro -o index.ndx         ;Select protein and ligand
gmx grompp -f nvt.mdp -c em2.gro -r em2.gro -p topol.top -n index.ndx -o nvt.tpr -maxwarn 1
gmx mdrun -v -deffnm nvt
gmx grompp -f npt.mdp -c nvt.gro -r nvt.gro -t nvt.cpt -p topol.top -n index.ndx -o npt.tpr -maxwarn 1
gmx mdrun -v -deffnm npt
gmx grompp -f md.mdp -c npt.gro -r npt.gro -t npt.cpt -p topol.top -n index.ndx -o md_0_1.tpr -maxwarn 1 
gmx mdrun -v -deffnm md_0_1         ;Run MD simulation

###Result analysis command statement###
gmx trjconv -s xxx.tpr -f xxx.xtc -o xxx_noPBC.xtc -pbc mol -center         ;Perform periodic correction
gmx trjconv -f xxx_noPBC.xtc -o trj_0-20ns -b 0 -e 20000 -dt 100         ;For the simulation results of the first 20 ns, a frame is extracted every 100 ps
gmx pairdist -f xxx_noPBC.xtc -s xxx.tpr -ref "atomnr A" -sel "atomnr B"         ;Calculate the change in distance between atoms A and B during the entire simulation
gmx cluster -f trj_0-20ns.xtc -s xxx.tpr -method gromos -o rmsd-clust.xpm -g cluster.log -dist rmsd-dist.xvg -cutoff 0.12 -clid clust-id.xvg -cl clusters.pdb -tu ns         ;The gromos method was used for cluster analysis
