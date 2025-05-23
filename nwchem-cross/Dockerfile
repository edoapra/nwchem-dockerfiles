ARG UBUNTU_TAG=ubuntu:plucky

FROM --platform=$BUILDPLATFORM tonistiigi/xx AS xx

FROM --platform=$BUILDPLATFORM $UBUNTU_TAG AS build
LABEL maintainer="Edoardo Apra <edoardo.apra@pnnl.gov>"

LABEL org.opencontainers.image.description="NWChem image built from the master branch"
LABEL org.opencontainers.image.source="https://github.com/nwchemgit/nwchem"
LABEL org.opencontainers.image.documentation="https://nwchemgit.github.io/Containers.html"


#caching from
COPY Dockerfile cache* /tmp/

ARG TARGETPLATFORM
ARG DEBIAN_FRONTEND=noninteractive
ARG         FC
ARG         NWCHEM_BRANCH
ARG         GITHUB_REPOSITORY_OWNER
ARG CACHE_HIT
ENV         NWCHEM_TOP="/opt/nwchem"
ENV         FC=${FC}
ENV         NWCHEM_BRANCH=${NWCHEM_BRANCH}
ENV         GITHUB_REPOSITORY_OWNER=${GITHUB_REPOSITORY_OWNER}
ARG         NWCHEM_MODULES="all  "  \
            ARMCI_NETWORK=MPI-PT  \
            USE_OPENMP=1  \
            USE_NOIO=y  \
            USE_MPI=y  \
            USE_MPIF=y  \
            USE_MPIF4=y  \
	    BUILD_MPICH=1 \
	    BUILD_OPENBLAS=1 \
	    BUILD_SCALAPACK=1 \
	    BLAS_SIZE=8 \
	    SCALAPACK_SIZE=8 \
	    USE_HWOPT=n 
#get NWCHEM_TARGET
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

# copy xx scripts to your build stage
RUN uname -a
COPY --from=xx / /
# export TARGETPLATFORM (or other TARGET*)
ARG TARGETPLATFORM
COPY cross.txt /tmp/.
COPY cross32.txt /tmp/.
# you can now call xx-* commands
RUN set -e && xx-info env && \
    env|grep PLATF && \
    GCC_VERSION=15 &&\
    XX_MARCH=$(xx-info march) &&\
    apt-get update && \
    echo "XX_MARCH is ${XX_MARCH}" && \
    apt-get install -y software-properties-common ; \
    xx-apt  install -y libc6-dev  binutils gcc-${GCC_VERSION} gfortran-${GCC_VERSION}  &&\
    apt-get -q=2 install -y make file gcc-${GCC_VERSION} && \
    set -x && \
    echo XX_MARCH $(xx-info march) &&\
	     update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-"$GCC_VERSION" 100 && \
	     ls -lrt /usr/bin/gcc* && \
	     ls -lrt /usr/bin/*gcc* && \
    $(xx-info)-gcc-${GCC_VERSION} -v && \
    ls -lrta /tmp && \
    if [[ -f /tmp/libext.tar.bz2 ]]; then echo "analyzing libext" ; apt-get install -y bzip2 tar;  tar tjvf /tmp/libext.tar.bz2;  fi &&\
arch=${XX_MARCH} ; echo " xx arch is " $arch ;if [[ "$arch" == "x86_64" ]] || [[ "$arch" == "aarch64" ]] || [[ "$arch" == "riscv64" ]] || [[ "$arch" == "ppc64le" ]]|| [[ "$arch" == "s390x" ]]; then echo "export NWCHEM_TARGET=LINUX64" > /tmpfile ; fi && \
arch_123=$(echo $arch |cut -c 1-3); if [[ "$arch" == "i686" ]] || [[ "$arch" == "i386" ]] || [[ "$arch_123" == "arm" ]] ; then echo "export NWCHEM_TARGET=LINUX" > /tmpfile ; fi  &&\
echo "export GCC_VERSION="${GCC_VERSION} >> /tmpfile && \
. /tmpfile; echo "NWCHEM_TARGET is " ${NWCHEM_TARGET}  && \
     echo ' defining modules according to CACHE_HIT=' "$CACHE_HIT" && \
     if [[ "$arch" != "x86_64" ]] && [[ "$CACHE_HIT" == "N" ]]; then export NWCHEM_MODULES="nwdft solvation driver stepper"; echo 'smaller build without cache. CACHE_HIT='$CACHE_HIT ; else echo 'large build with cache. CACHE_HIT='$CACHE_HIT ; export NWCHEM_MODULES="all"  ; fi &&\
#one single ugly command to reduce docker size
            tries=0 ; until [ "$tries" -ge 10 ] ; do \
             xx-apt  install -y  g++ gfortran libssl-dev \
             && apt-get -q=2 install -y --no-install-recommends rsync  make curl unzip cmake git file wget tar bzip2 bc ca-certificates patch pkg-config python3 python3-dev openssh-client python-is-python3 \
	    && break; \
	    tries=$((tries+1)) ; echo attempt no.  $tries    ; sleep 30 ;  done \
	    && echo FCFCFC is ${FC} \
	    && echo NWCHEM_BRANCH is ${NWCHEM_BRANCH} \
	    && echo GITHUB_REPOSITORY_OWNER is ${GITHUB_REPOSITORY_OWNER} \
#	    && arch=$(uname -m) \
	    && arch=$XX_MARCH \
	    && echo arch is "$arch" \
	    && arch_dpkg=$(dpkg --print-architecture)  \
	    && if [[ "$arch" == "aarch64" ]]; then arch_dpkg="arm64"; fi \
	    && if [[ "$arch" == "x86_64" ]]; then arch_dpkg="amd64"; fi \
	    && echo "URL for cloning"  https://github.com/${GITHUB_REPOSITORY_OWNER}/nwchem.git \
	    && echo ca_certificate=/etc/ssl/certs/ca-certificates.crt >  ~/.wgetrc \
            && cd /opt; rm -rf nwchem||true; wget -q https://github.com/${GITHUB_REPOSITORY_OWNER}/nwchem/tarball/"$NWCHEM_BRANCH" -O - | tar -xz  --exclude=web --exclude=examples --exclude=doc --exclude="nwxc*"  --exclude="chem-library-tests" \
	    &&    mv ${GITHUB_REPOSITORY_OWNER}-nwchem-* nwchem \
            && cd nwchem/src \
#set NWCHEM_TARGET 
            &&  . /tmpfile; echo "NWCHEM_TARGET is " ${NWCHEM_TARGET} \
#fix openblas cross-compilation
            && arch=$XX_MARCH \
	    && echo 'openblas arch is ' $arch \
	    && if [[ "$arch" == "aarch64" ]]; then export FORCETARGET=" TARGET=ARMV8 "; fi \
	    && if [[ "$arch" == "ppc64le" ]]; then export FORCETARGET=" TARGET=POWER8 "; fi \
	    && if [[ "$arch" == "riscv64" ]]; then export FORCETARGET=" TARGET=RISCV64_GENERIC "; export ARMCI_NETWORK=MPI-TS; unset USE_OPENMP; fi \
	    && if [[ "$arch" == "ppc64le" ]]; then unset USE_OPENMP; fi \
	    && if [[ "$arch" == "armv7l" ]]; then export FORCETARGET=" TARGET=ARMV7 "; export ARMCI_NETWORK=MPI-TS; fi \
	    && if [[ "$arch" == "s390x" ]]; then export FORCETARGET=" TARGET=ZARCH_GENERIC "; export ARMCI_NETWORK=MPI-TS; fi \
# _SIZE=4 and openblas pkg for 32bit archs
            && echo 'FORCETARGET is ' $FORCETARGET \
            && if [[ ${NWCHEM_TARGET} == "LINUX" ]]; then export USE_PEIGS=1; export BLAS_SIZE=4; unset SCALAPACK_SIZE; unset BUILD_SCALAPACK;  echo 'unset BUILD_SCALAPACK' >> /tmpfile; echo 'export BLAS_SIZE=4' >> /tmpfile;echo 'export USE_PEIGS=1' >> /tmpfile; fi  \
# smaller build on qemu emulated archs
# compile libxc for amd64 
	    && if [[ "$arch" == "x86_64" ]]; then export USE_LIBXC=1;  \
	    export USE_SIMINT=1; export SIMINT_MAXAM=5; export SIMINT_VECTOR=avx2 ; fi \
            && if [[ -f /tmp/libext.tar.bz2 ]]; then echo "fetching libext" ; cd libext ; tar xjvf /tmp/libext.tar.bz2; rm -rf /tmp/libext.tar.bz2 ; ./bin/mpif90 -show; mkdir -p mpich/mpich ; ls -l /opt/nwchem/src/libext/mpich/mpich/../../include ;cd ..  ; fi \
            && if [[ -f /tmp/simint.tar.bz2 ]]; then cd NWints/simint/libsimint_source ; tar xjf /tmp/simint.tar.bz2; rm -rf /tmp/simint.tar.bz2 ; cd ../../..  ; fi\
	    && df -h /dev/shm \
            && mkdir -p ../bin/${NWCHEM_TARGET} \
            && gcc-${GCC_VERSION} -o ../bin/"$NWCHEM_TARGET/depend.x" config/depend.c \
	    && MY_FFLAG_INT="-fdefault-integer-8" \
	    && I4I8=" --enable-i8 " \
	    && MYCPU=${XX_MARCH} \
	    && if [  $MYCPU == "armv7l" ]; then MYCPU="arm"; unset MY_FFLAG_INT ; I4I8=" --enable-i4 "; fi \
	    && echo "MYCPU is " $MYCPU \
	    && echo "MY_FFLAG_INT is " $MY_FFLAG_INT \
	    && echo "I4I8 is " $I4I8 \
            && make DEPEND_CC=gcc-${GCC_VERSION} CC=$(xx-info)-gcc-${GCC_VERSION}  FC=$(xx-info)-gfortran-${GCC_VERSION}   _CPU=${MYCPU}  nwchem_config  \
	    && if [[ "$arch" == "x86_64" ]]; then make CC=gcc-${GCC_VERSION}  FC=gfortran-${GCC_VERSION} V=-1 -j3 ;\
	    else make DEPEND_CC=gcc-${GCC_VERSION} CC=$(xx-info)-gcc-${GCC_VERSION}  FC=$(xx-info)-gfortran-${GCC_VERSION}  FFLAG_INT=${MY_FFLAG_INT} _CPU=${MYCPU} MAYBE_HOST="--host=$(xx-info triple) --build=x86_64-linux-gnu "  MAYBE_OFFLOAD="$I4I8" V=-1 -j3 || { echo 'Compilation Failure'; cat $(find tools -name config.log) ;exit 1; } ; fi \
	    &&  if [[ -f ../bin/${NWCHEM_TARGET}/nwchem ]]; then echo 'NWChem binary created' ; else  cat $(find . -name "*.log") ;  echo 'compilation error1'; exit 1; fi \
	    &&  if [[ -f ../bin/${NWCHEM_TARGET}/nwchem ]]; then echo 'NWChem binary created' ; else cat $(find . -name "*log") ;  echo 'compilation error2'; exit 1; fi \
#clean unnecessary source to reduce docker size
	    && rm -rf ../QA/chem_library_tests ../QA/amber-tests ../QA/Nightly_Jobscripts  \
	    && cd libext ; tar cjvf /opt/nwchem/src/libext/libext.tar.bz2 bin/* lib/* include/*  libxc/install/lib/* libxc/install/include/* ||true \
	    && cd /opt/nwchem/src/NWints/simint/libsimint_source/ ; tar cjvf /opt/nwchem/src/libext/simint.tar.bz2 simint_install/* || true \
	    && cd /opt/nwchem/src \
	    && cd $NWCHEM_TOP/src || { echo "Failure"; exit 1; } 

FROM $UBUNTU_TAG AS relink

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

ARG CACHE_HIT
ARG         NWCHEM_MODULES="all  "  \
            ARMCI_NETWORK=MPI-PT  \
            USE_OPENMP=1  \
            USE_NOIO=y  \
            USE_MPI=y  \
            USE_MPIF=y  \
            USE_MPIF4=y  \
	    BUILD_MPICH=1 \
	    BUILD_OPENBLAS=1 \
	    BUILD_SCALAPACK=1 \
	    BLAS_SIZE=8 \
	    SCALAPACK_SIZE=8 \
	    USE_HWOPT=n 

RUN mkdir -p /opt

COPY --from=build /opt/nwchem /opt/nwchem
COPY --from=build /tmpfile /tmpfile
COPY run_qas.sh /tmp/.
	      
RUN           groupadd -r nwchem -g 1994 \
              && useradd  -u 1994 -r -g nwchem -c 'NWChem user' -M -d /opt/nwchem -s /bin/bash nwchem \
            &&  . /tmpfile; echo "NWCHEM_TARGET is " ${NWCHEM_TARGET} ; echo "GCC_VERSION is " ${GCC_VERSION}\
             && apt-get update \
             && apt-get -q=2   install -y make  gfortran-${GCC_VERSION} python3 python3-dev libssl-dev bzip2 tar\
             && apt-get -q=2 install -y --no-install-recommends file  ca-certificates  pkg-config python3-dev openssh-client perl xz-utils  libgfortran5  libgomp1 libgcc-s1 \
	     && echo " CACHE_HIT is "  "$CACHE_HIT"  \
	     && if [[ "$CACHE_HIT" == "Y" ]]; then \
             export NWCHEM_MODULES="all python " ;\
	     arch=$(uname -m) ; \
	     if [[ "$arch" == "x86_64" ]]; then export USE_LIBXC=1; export USE_SIMINT=1;fi ; \
	     echo "uname -m is " $(uname -m) ;\
	     MYCPU=$(uname -m) ; \
    	     echo "MYCPU is " ${MYCPU} ;\ 
	    cd /opt/nwchem/src ;\
	    rm ../bin/${NWCHEM_TARGET}/depend.x ;\
	    gcc-${GCC_VERSION} -o ../bin/"$NWCHEM_TARGET/depend.x" config/depend\
.c ;\
            make DEPEND_CC=gcc-${GCC_VERSION} CC=gcc-${GCC_VERSION}  FC=gfortran-${GCC_VERSION} _FC=gfortran _CPU=${MYCPU}   nwchem_config ;\
	    cd python ;\
            make DEPEND_CC=gcc-${GCC_VERSION} CC=gcc-${GCC_VERSION}  FC=gfortran-${GCC_VERSION} _FC=gfortran _CPU=${MYCPU}    V=1 -j3;\
            cd .. ;\
	    if [[ ${NWCHEM_TARGET} == "LINUX64" ]]; then cd input ; touch memory_def.F ;\
	    make LIB_DEFINES+=" -DDFLT_TOT_MEM=366772864 -DMORE_MA_MEM=1 "  CC=gcc-${GCC_VERSION} _CC=gcc FC=gfortran-${GCC_VERSION} _FC=gfortran  ; cd .. ;fi ; \
	    make CC=gcc-${GCC_VERSION} _CC=gcc FC=gfortran-${GCC_VERSION} _FC=gfortran    link ; fi \
	     && cp /opt/nwchem/src/libext/bin/mpiexec /opt/nwchem/bin/. \
	     && cp /opt/nwchem/src/libext/bin/hydra* /opt/nwchem/bin/. \
	     &&  echo 'ls -l /opt/nwchem/bin/' $(ls -l /opt/nwchem/bin) \
	     &&  echo 'ls -l /opt/nwchem/src/libext/bin/' $(ls -l /opt/nwchem/src/libext/bin/) \
	     && ln -sf /opt/nwchem/bin/mpiexec /opt/nwchem/bin/mpirun \
	     && file /opt/nwchem/bin/${NWCHEM_TARGET}/nwchem \
	     && uname -a \
	     && ldd /opt/nwchem/bin/${NWCHEM_TARGET}/nwchem \
	     && cd /opt/nwchem/ \
	     && export SKIP_CACHE=1  \
	     && export NWCHEM_EXECUTABLE=/opt/nwchem/bin/${NWCHEM_TARGET}/nwchem \
	     && cp /tmp/run_qas.sh ./travis/run_qas.sh  \
	     && OMP_NUM_THREADS=1 ./travis/run_qas.sh \
             && cp -rp QA/tests/dft_he2+ /tmp/. \
             && rm -rf QA/tests/*   \
	     && mv /tmp/dft_he2+ QA/tests/. \
	     && echo done \
&& echo creating tar file \
&& if [[ "$CACHE_HIT" == "Y" ]]; then \
tar cJf /tmp/nwchem.tar.xz /opt/nwchem/bin/* /opt/nwchem/src/tools/build/config.log  /opt/nwchem/QA/* /opt/nwchem/src/basis/* /opt/nwchem/src/nwpw/libraryps/*  ; \
else  \
tar cJf /tmp/nwchem.tar.xz /opt/nwchem/bin/* /opt/nwchem/src/tools/build/config.log  /opt/nwchem/QA/* /opt/nwchem/src/basis/* /opt/nwchem/src/nwpw/libraryps/* /opt/nwchem/src/libext/openblas/OpenBLAS/openblas.log   /opt/nwchem/src/libext/*tar*bz2  || true; fi \
&& echo ls tar file \
&& ls -lrt /tmp/nwchem.tar.xz

	     
FROM $UBUNTU_TAG AS push
ARG CACHE_HIT
COPY --from=relink /tmp/nwchem.tar.xz /tmp/nwchem.tar.xz
COPY --from=relink /tmpfile /tmpfile
SHELL ["/bin/bash", "-o", "pipefail", "-c"]

RUN apt-get update -q=2 && apt  install -y -q=2 libatomic1  binutils bzip2 xz-utils  \
	     libgfortran5 libpython3.*  libgomp1 libgcc-s1 \
&&           groupadd -r nwchem -g 1994 \
              && useradd  -u 1994 -r -g nwchem -c 'NWChem user' -M -d /opt/nwchem -s /bin/bash nwchem \
&&  . /tmpfile; echo "NWCHEM_TARGET is " ${NWCHEM_TARGET} \
&& cd / ; tar xJf /tmp/nwchem.tar.xz  ; pwd ;ls -lart \
&& ln -sf /opt/nwchem/bin/${NWCHEM_TARGET}/nwchem /opt/nwchem/bin/nwchem \
&&  ls -lart /tmp \
&&  if [[ -f /opt/nwchem/bin/nwchem ]]; then echo 'NWChem binary created from tar file' ; else   echo 'error in tar file extraction' $(pwd); exit 1; fi \
&& ldd /opt/nwchem/bin/nwchem|| true  \
&& rm -rf /tmp/nwchem.tar.xz  /var/lib/dpkg/ /var/lib/apt

ENV         FFIELD=amber  \
            NWCHEM_TOP=/opt/nwchem \
            AMBER_1=/opt/nwchem/src/data/amber_s/  \
            AMBER_2=/opt/nwchem/src/data/amber_q/  \
            AMBER_3=/opt/nwchem/src/data/amber_x/  \
            AMBER_4=/opt/nwchem/src/data/amber_u/  \
            SPCE=/opt/nwchem/src/data/solvents/spce.rst  \
            CHARMM_S=/opt/nwchem/src/data/charmm_s/  \
            CHARMM_X=/opt/nwchem/src/data/charmm_x/  \
	    OMPI_MCA_btl_vader_single_copy_mechanism=none \
	    OMP_NUM_THREADS=1 \
	    COMEX_MAX_NB_OUTSTANDING=16 \
	    NWCHEM_EXECUTABLE=/opt/nwchem/bin/nwchem \
            PATH=${PATH}:/opt/nwchem/bin/


#USER        nwchem

WORKDIR     /data
ENTRYPOINT  ["nwchem"]

