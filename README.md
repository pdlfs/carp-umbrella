**Download, build, and install carp (and its dependencies) in a single step.**

carp-umbrella
================

## Overview

This package is designed for quickly setting up carp on various computing platforms ranging from commodity clusters to highly-optimized HPC systems used in national labs. The package features an automated process that downloads, builds, and installs carp (including its software dependencies) in a single step.

Written atop cmake, carp-umbrella is expected to work with major computing platforms. We have successfully tested carp-umbrella on CMU PDL Narwhal, LANL Trinity, LANL Grizzly, NERSC Edison, and NERSC Cori.

## Modules

The list of primary modules used by carp-umbrella is in CMakeLists.txt (look for umbrella cmake "include" statements).   These include files reside in the umbrella/umbrella subdirectory.

## Installation

A recent CXX compiler (e.g., gcc 5 or later) with standard building tools including make, cmake, and automake (used by some of our dependencies), as well as a few other common library packages such as libpapi and libnuma (for debugging and performance montoring).

### Ubuntu

On Ubuntu systems, these software requirements can be met by:

```bash
sudo apt-get install gcc g++ make cmake autoconf automake libtool pkg-config libpapi-dev libnuma-dev git
```

To build carp and install it under a specific prefix (e.g., $HOME/carp):

```bash
mkdir -p $HOME/carp
cd $HOME/carp

# After installation, we will have the following:
#
# $HOME/deltafs
#  -- bin
#  -- include
#  -- lib
#  -- src
#      -- carp-umbrella
#      -- carp-umbrella-build
#  -- share
#

mkdir -p src
cd src
git clone https://github.com/pdlfs/carp-umbrella.git
mkdir -p carp-umbrella-build
cd carp-umbrella-build

cmake -DCMAKE_INSTALL_PREFIX=$HOME/carp -DCMAKE_BUILD_TYPE=RelWithDebInfo \
  -DUMBRELLA_BUILD_TESTS=OFF -DUMBRELLA_SKIP_TESTS=ON \
  -DMERCURY_NA_INITIALLY_ON="bmi;sm" -DMERCURY_POST_LIMIT=OFF \
  -DMERCURY_CHECKSUM=OFF \
../carp-umbrella

make

make install

```

#### stdatomic.h

Mercury RPC, one critical deltafs dependency, implements atomic counters using whatever is available on the current system (e.g., OPA library, stdatomic.h, OSX's OSAtomic.h, the windows API). On Linux, when the C compiler (such as gcc 4.8 or eariler) cannot provide stdatomic.h, the OPA (Open Portable Atomics) library must be installed in order to compile mercury. There are at least 3 ways to install this library. The easiest way is to install CCI along with mercury since CCI uses and will install OPA alongside itself. In addition, MPICH also ships with OPA so installing MPICH will install OPA too. Finally, one can always compile and install OPA as a standalone library: https://github.com/pmodels/openpa.

#### _Atomic

Some components in the libfabric codebase make use of the "_Atomic" qualifier defined in c11: https://en.cppreference.com/w/c/atomic. Eariler C compilers (such as gcc 4.8 or eariler, and Intel icc 17.0 or eariler) may not recognize it, and may not even have stdatomic.h. To work around this issue, one simply has to use a compiler that understands _Atomic.

**Enjoy** :-)
