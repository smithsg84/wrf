# PFWRF

This repository contains the PF-WRF integration work.

WRF is based on the on released V4.0.  Version was retrieved with:

```bash
wget http://www2.mmm.ucar.edu/wrf/src/WRFV4.0.TAR.gz
```
WRF-Hydro was obtained from the public Git repo at: <https://github.com/smithsg84/wrf_hydro_nwm_public>

The WRF-Hydro repository version was copied into the WRF repository at wrf/hydro.  The cloned WRF-Hydro repository is at: <https://github.com/smithsg84/wrf_hydro_nwm_public>

The pfwrf branch was used to for the PF integration work.

## Building PF-WRF-Hydro

### NERSC Cori system from PF-WRF-HYDRO repository

```bash
#-----------------------------------------------------------------------------
# This gets environment that will build PF and WRF
#-----------------------------------------------------------------------------
git clone git@github.com:smithsg84/pf-build.git

pushd pf-build
source bin/pfsetenv.sh
popd

# Uses Cray install of NetCDF
export NETCDF=${NETCDF_DIR}
export NCDIR=${NETCDF_DIR}

#-----------------------------------------------------------------------------
# Build ParFlow
#-----------------------------------------------------------------------------

pushd pf-build
bin/pfclone

make cori
make
make install

popd

#-----------------------------------------------------------------------------
# Build PF-WRF-Hydro
#-----------------------------------------------------------------------------

# This uses some build scripts developed for LLNL.
git clone git@github.com:smithsg84/wrf.git

source wrf/hydro/template/pfwrf-setEnvar.sh

cd wrf

./configure

#Select the dmpar option for your compiler (50)
#Select option 1 - basic nesting

# Need to patch name of fortran compiler in Hydro

perl -pi.bak -e 's/mpif90/ftn/g' hydro/macros

./compile em_real >& zz.compile
```

### LLNL TOS 3 systems from PF-WRF-HYDRO repository

Requires needs NetCDF to be built.

```bash
#-----------------------------------------------------------------------------
# This gets LC environment that will build PF and WRF
#-----------------------------------------------------------------------------
git clone git@github.com:smithsg84/pf-build.git

pushd pf-build
source bin/pfsetenv.sh
popd

export CC=$(which icc)
export CXX=$(which icpc)
export FC=$(which ifort)

export NETCDF=/usr/gapps/thcs/apps/toss_3_x86_64_ib/netcdf/4.6.1
export NCDIR=${NETCDF}
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$NETCDF/lib

#-----------------------------------------------------------------------------
# Build ParFlow
#-----------------------------------------------------------------------------

pushd pf-build
bin/pfclone

make llnl
make
make install

popd

#-----------------------------------------------------------------------------
# Build PF-WRF-Hydro
#-----------------------------------------------------------------------------

# This uses some build scripts developed for LLNL.
git clone git@github.com:smithsg84/wrf.git

source wrf/hydro/template/pfwrf-setEnvar.sh

cd wrf

./configure

#Select the dmpar option for your compiler (15)
#Select option 1 - basic nesting

 ./compile em_real >& zz.compile
```

## Building NetCDF

### LLNL TOS 3 systems

```bash
#-----------------------------------------------------------------------------
# This gets LC environment that will build PF and WRF
#-----------------------------------------------------------------------------
git clone git@github.com:smithsg84/pf-build.git

pushd pf-build
source bin/pfsetenv.sh
popd

export CC=$(which icc)
export CXX=$(which icpc)
export FC=$(which ifort)

export NETCDF=/usr/gapps/thcs/apps/toss_3_x86_64_ib/netcdf/4.6.1
export NCDIR=${NETCDF}
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$NETCDF/lib

#-----------------------------------------------------------------------------
# Build NetCDF C interface (only needs to be done once)
#-----------------------------------------------------------------------------
wget https://github.com/Unidata/netcdf-c/archive/v4.6.1.tar.gz
mv v4.6.1.tar.gz netcdf-v4.6.1.tar.gz
tar xf netcdf-v4.6.1.tar.gz 

mkdir netcdf-c-4.6.1/build
pushd netcdf-c-4.6.1/build
cmake -DCMAKE_INSTALL_PREFIX=${NETCDF} .. >& zz.cmake
make -j 12 >& zz.make
make install
popd

pushd ${NETCDF}
ln -s lib64 lib
popd

#-----------------------------------------------------------------------------
# Build NetCDF Fortran interface (only needs to be done once)
#-----------------------------------------------------------------------------
wget https://github.com/Unidata/netcdf-fortran/archive/v4.4.4.tar.gz
mv v4.4.4.tar.gz netcdf-fortran-v4.4.4.tar.gz
tar xf netcdf-fortran-v4.4.4.tar.gz

mkdir netcdf-fortran-4.4.4/build
pushd netcdf-fortran-4.4.4/build
# Tests have build issues
cmake -DCMAKE_INSTALL_PREFIX=${NETCDF} -DENABLE_TESTS=OFF .. >& zz.cmake
make -j 12
make install
popd
```

