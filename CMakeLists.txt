#
# Copyright (c) 2021 Carnegie Mellon University,
# Copyright (c) 2021 Triad National Security, LLC, as operator of
#     Los Alamos National Laboratory.
#
# All rights reserved.
#
# Use of this source code is governed by a BSD-style license that can be
# found in the LICENSE file. See the AUTHORS file for names of contributors.
#

#
# CMakeLists.txt  build carp and its environment
# 12-Mar-2021  chuck@ece.cmu.edu
#

#
# general command line config:
#
#   -DCMAKE_INSTALL_PREFIX=/usr/local     # installation prefix
#   -DCMAKE_BUILD_TYPE=RelWithDebInfo     # or Release, Debug, etc.
#      (XXX: currently only applied to cmake-based builds)
#
#   -DUMBRELLA_BUILDTESTS=OFF             # build unit tests?
#   -DUMBRELLA_RUNTESTS=OFF               # skip running unit tests?
#
# finding dependencies:
#
#   -DCMAKE_PREFIX_PATH='/pkg'            # look for additional installs here
#
# the following also applies for configure scripts:
#   -DCMAKE_INCLUDE_PATH='/pkg/include'   # extra include directories
#   -DCMAKE_LIBRARY_PATH='/pkg/lib'       # extra library path
#
# note these are all cmake lists (so more than one directory can
# be specified using a semicolon to create a path).
#
# specifying alternate compilers (overrides CC/CXX environment variables):
#   -DCC=/bin/cc
#   -DCXX=/bin/cxx
#
#   you may also use the longer names
#     -DCMAKE_C_COMPILER
#     -DCMAKE_CXX_COMPILER
#
# specifying which mpi to use by pointing at the wrappers:
#   -DMPI_C_COMPILER=/usr/bin/mpicc
#   -DMPI_CXX_COMPILER=/usr/bin/mpicxx
#   -DMPIEXEC=/usr/bin/mpiexec
#

cmake_minimum_required (VERSION 3.0)

#
# put the umbrella scripts in our search path and include umbrella-init
# (umbrella-init needs to run before "project")
#
list (APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/umbrella")
include (umbrella-init)

#
# we don't really use MPICXX for now. The difference between CXX and MPICXX
# is that CXX refers to the MPI C API being usable from C++, whereas MPICXX
# refers to the MPI-2 C++ API that was removed again in MPI-3.
# To enable MPICXX, run umbrella with -DMPI_CXX_SKIP_MPICXX=Off.
#
set (MPI_CXX_SKIP_MPICXX ON CACHE BOOL "True if MPICXX should be skipped")

#
# need to mark project as using C/CXX so we can probe for MPI
# using a compiler that may have been passed in via the command line
# or by using environment vars.
#
project (carp-umbrella C CXX)

#
# configure umbrella for MPI and pull in the main routines
#
set (UMBRELLA_MPI 1)
include (umbrella-main)

#
# propagate the MPICXX setting to all cmake subprojects
#
list(APPEND UMBRELLA_CMAKECACHE
     -DMPI_CXX_SKIP_MPICXX:BOOL=${MPI_CXX_SKIP_MPICXX})


#
# set git tags to the versions we want to stick to by default...
#
umbrella_opt_default (BMI_TAG "6ea0b78f")             # Jan 2020
umbrella_opt_default (CARP_TAG "v1.01")
umbrella_opt_default (CH_PLACEMENT_TAG "ab096ee0")    # Feb 2022
# for deltafs-common, use the commit before we updated
# the logging to glog v0.7.0
umbrella_opt_default (DELTAFS_COMMON_TAG "v24.0")
umbrella_opt_default (DELTAFS_NEXUS_TAG "v1.19")
umbrella_opt_default (DELTAFS_SHUFFLE_TAG "a20890d9") # Apr 2021
umbrella_opt_default (DELTAFS_VPIC_PRELOAD_TAG "v2.11")
umbrella_opt_default (DELTAFS_TAG "v2024.0")
# HDF5/H5PART only used if CARP_H5PART turned on
umbrella_opt_default (HDF5_TAG "develop")
umbrella_opt_default (H5PART_TAG "master")
umbrella_opt_default (MERCURY_PROGRESSOR_TAG "6892c220") # Oct 2020
umbrella_opt_default (MERCURY_RUNNER_TAG "cbe7a219")  # Apr 2021
umbrella_opt_default (MERCURY_TAG "3f6b5484")         # Feb 2022
umbrella_opt_default (MSSG_TAG "v1.4")
umbrella_opt_default (NEXUS_RUNNER_TAG "04ee7e8a")    # Apr 2021
umbrella_opt_default (OFI_TAG "1a89deaa")             # Feb 2022
umbrella_opt_default (ONETBB_TAG "6666292f")          # Feb 2022
umbrella_opt_default (PDLFS_SCRIPTS_TAG "carpv1.02")
umbrella_opt_default (GLOG_TAG "v0.6.0")

#
# vpic selection.  vpic407 is the old depreciated vpic code base.
#
set(VPIC407 ON CACHE BOOL "Use depreciate VPIC407 code")
if(VPIC407)
    umbrella_opt_default (VPIC407_TAG "0a1c2c50")         # Aug 2019
    set (my-vpic-inc trecon-reader)                       # pulls vpic407
    umbrella_opt_default (PDLFS_SCRIPTS_VPIC407 1)
    message (STATUS "  vpic: vpic 407 selected (depreciated)")
else()
    umbrella_opt_default (VPIC_TAG "c2e9be14")            # Sep 2019
    umbrella_opt_default (VPIC_DECKS_TAG "1c2ce168")      # Aug 2019
    set (my-vpic-inc vpic-decks)                          # pulls vpic
    umbrella_opt_default (PDLFS_SCRIPTS_VPIC407 0)
    message (STATUS "  vpic: vpic main selected")
endif()

#
# any additional cache variables we want to add that are not already
# part of the umbrella?
#
set (DELTAFS_VERBOSE "0" CACHE STRING "deltafs verbose level")
set_property (CACHE DELTAFS_VERBOSE PROPERTY STRINGS "0"
              "1" "2" "3" "4" "5" "6" "7" "8" "9" "10")
message (STATUS "  deltafs_verbose: ${DELTAFS_VERBOSE}")

#
# set other configs before we start pulling in the pieces...
#

# enable CARP_H5PART to build carp's vpicwriter_runner (requires hdf5/h5part)
umbrella_opt_default (CARP_H5PART OFF)

# note: NA_INITIALLY_ON is only applied as the defaults the first time
# you run cmake.  if you want to reconfigure, change MERCURY_BMI, et al.
umbrella_opt_default (MERCURY_NA_INITIALLY_ON "bmi;ofi;sm")

set (UMBRELLA_USE_GLOG ON CACHE BOOL "Use glog for logging")

set (PDLFS_OPTIONS -DPDLFS_MERCURY_RPC=ON -DPDLFS_SNAPPY=OFF
                   -DPDLFS_GFLAGS=OFF -DPDLFS_GLOG=${UMBRELLA_USE_GLOG}
                   -DPDLFS_VERBOSE=${DELTAFS_VERBOSE})

include (umbrella/carp)
include (umbrella/ch-placement)
include (umbrella/deltafs)
include (umbrella/deltafs-common)
include (umbrella/deltafs-nexus)
include (umbrella/deltafs-shuffle)
include (umbrella/deltafs-vpic-preload)
include (umbrella/mercury)
include (umbrella/mercury-progressor)
include (umbrella/mercury-runner)
include (umbrella/mssg)
include (umbrella/nexus-runner)
include (umbrella/onetbb)
include (umbrella/pdlfs-scripts)
include (umbrella/${my-vpic-inc})
