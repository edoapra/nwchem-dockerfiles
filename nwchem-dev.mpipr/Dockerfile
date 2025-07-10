#based on earlier work by Vladimir Konjkov <Konjkov.VV@gmail.com>
#
# Create the image with the command
# docker build -t nwchem-700.mpipr .
# 
#after successful build one you can execute the QA tests using the command
#docker run nwchem-700.mpipr dft_he2+ h2o_dk h2o_opt
#

FROM        debian

MAINTAINER  Edoardo Apra <edoardo.apra@pnnl.gov>

RUN         apt-get update \
            && apt-get -y upgrade \
            && apt-get install -y  gfortran mpich libmpich-dev  make ssh patch curl subversion wget unzip perl git file python3 python3-dev bzip2 \
            && apt-get clean
#get OpenBLAS and compile with 64-bit integers
WORKDIR     /opt
ENV         ARMCI_NETWORK=MPI-PR NWCHEM_TOP="/opt/nwchem" \
            NWCHEM_BRANCH=master \
            NWCHEM_TARGET=LINUX64 \
            NWCHEM_MODULES="all python" \
            BUILD_OPENBLAS=1 \
            BUILD_SCALAPACK=1 \
            BLAS_SIZE=8 \
            SCALAPACK_SIZE=8 \
	    USE_HWOPT=n \
	    USE_LIBXC=1 \
	    USE_SIMINT=1 \
	    SIMINT_VECTOR=AVX2 \
	    SIMINT_MAXAM=4 \
            USE_MPI=y \
            USE_MPIF=y \
            USE_MPIF4=y 
#ENV         MRCC_METHODS=y
#ENV         CCSDTQ=y
#ENV         CCSDTLR=y
RUN cd /opt; rm -rf nwchem||true; wget -q https://github.com/${GITHUB_REPOSITORY_OWNER}/nwchem/tarball/"$NWCHEM_BRANCH" -O - | tar -xz  --exclude=web --exclude=examples --exclude=doc --exclude="nwxc*"  --exclude="chem-library-tests" \
	    &&    mv ${GITHUB_REPOSITORY_OWNER}-nwchem-* nwchem \
            && cd nwchem/src \
            && make nwchem_config && make -j3  \
#clean unnecessary source to reduce docker size
            && rm -rf tce tools nwdft NWints geom symmetry util nwxc ddscf lapack blas rism argos peigs rmdft gradients symmetry property smd lucia dplot propery hessian ccsd mp2_grad moints cafe analyz dimqm /opt/nwchem/lib

#ENV         NWCHEM_EXECUTABLE=${NWCHEM_TOP}/bin/LINUX64/nwchem
#ENV         NWCHEM_BASIS_LIBRARY=${NWCHEM_TOP}/src/basis/libraries/
#ENV         NWCHEM_NWPW_LIBRARY=${NWCHEM_TOP}/src/nwpw/libraryps/
ENV         FFIELD=amber \
            AMBER_1=${NWCHEM_TOP}/src/data/amber_s/ \
            AMBER_2=${NWCHEM_TOP}/src/data/amber_q/ \
            AMBER_3=${NWCHEM_TOP}/src/data/amber_x/ \
            AMBER_4=${NWCHEM_TOP}/src/data/amber_u/ \
            SPCE=${NWCHEM_TOP}/src/data/solvents/spce.rst \
            CHARMM_S=${NWCHEM_TOP}/src/data/charmm_s/ \
            CHARMM_X=${NWCHEM_TOP}/src/data/charmm_x/ \
            PATH=$PATH:${NWCHEM_TOP}/bin/LINUX64

#clean unnecessary packages
RUN         apt-get -y remove  make curl subversion  unzip cmake perl tcsh  &&  apt-get -y autoremove && apt-get clean

WORKDIR     /data
ENTRYPOINT  ["mpirun","-np","3","nwchem"]
