.. sectnum::

.. contents::

General notes
=============

The instructions here assume that you are using a recent development version of
CESM, such as in the cesm1_5 beta series.

Organization
============

This file describes aspects of building, running and writing unit tests that are
common to all components.

Subdirectories *cam*, *clm*, *csm_share* and *driver_cpl* contain instructions
and sample files specific for setting up unit tests with each of those
components. Each of those directories contains a README file that describes
steps unique to that component.

Subdirectory *common* contains sample code that is used in all of the examples.

Subdirectory *presentations* contains presentations that have been given about
unit testing in CESM.

Building and Running Unit Tests
===============================

Before you write your own unit tests, you should make sure that you can build
and run the existing tests.

Here we document two ways to do this:

* On yellowstone - easiest, but slower to build and run the tests

* On your personal desktop / laptop - takes more one-time setup work on your
  part, but once it is set up, this will let you build and run the unit tests
  much faster. This also facilitates doing code development on your personal
  machine, rather than remotely.

**Regardless of whether you are building on yellowstone or your own machine,
these instructions assume that you have defined environment variables giving two
important paths:**

* ``CIMEROOT``: Path to the cime directory associated with your current CESM
  checkout

* ``UNITTEST_ROOT``: Path to the directory containing the top-level
  ``CMakeLists.txt`` file for the component you want to test. This is:

  * clm: components/clm/src
  * cam: components/cam/test/unit

    * The CAM unit testing is broken at least as of cam5_4_51 (and for many
      previous tags). It works in
      https://svn-ccsm-models.cgd.ucar.edu/cam1/branch_tags/fix_unit_tests_with_cime_tags/fix_unit_tests_with_cime_n01_cam5_4_51

  * cime (includes tests of csm_share and driver_cpl): cime

So, for example, to test CAM, assuming I have a CESM checkout at /path/to/cesm
(and assuming bash shell syntax)::

  export CIMEROOT=/path/to/cesm/cime
  export UNITTEST_ROOT=/path/to/cesm/components/cam/test/unit

(The ``CIMEROOT`` and ``UNITTEST_ROOT`` variables are not actually required by
any of the unit test build scripts, but are used as a convenience in the
instructions below. You may also explicitly enter these paths wherever you
see ``$CIMEROOT`` or ``$UNITTEST_ROOT`` in these instructions.)

The initial unit test build for a component might take a few minutes. However,
rebuilds from the same directory should be much faster.

Building and running on yellowstone
-----------------------------------

First ``cd`` to ``$UNITTEST_ROOT``.

You must be in an interactive session on caldera to build and run the unit
tests. The easiest way to do this is to run::

  execca

and wait for an interactive prompt.

Then run the following command::

  $CIMEROOT/tools/unit_testing/run_tests.py --test-spec-dir=. --compiler=intel --mpilib=mpich2 \
  --mpirun-command=mpirun.lsf --cmake-args=-DPAPI_LIB=/glade/apps/opt/papi/5.3.0/intel/12.1.5/lib64

Note that the build is done in the directory ``$UNITTEST_ROOT/__command_line_test__``.

Building and running on your local machine
------------------------------------------

One-time setup for your machine
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Before you can run on your own machine, you need to install some
pre-requisites. These are basically the same prerequisites needed for building
CESM, plus pFUnit. These include:

* C and Fortran compilers

  * We recommend gfotran 4.9 or later

* MPI
* cmake 2.8 or later
* python 2.7 or later
* netcdf 4.3.2 or later (4.3.3.1 recommended)
* pFUnit: details below

Installing pFUnit
"""""""""""""""""

We use the pFUnit unit testing framework. This is an xUnit framework for writing
Fortran unit tests. The CESM unit tests require an mpi-enabled build of pFUnit.

#. Download pFUnit from
   http://sourceforge.net/projects/pfunit/files/latest/download

#. Set the PFUNIT environment variable. **This is also needed when running unit
   tests, so you should define it in your dot-file (e.g., .bashrc).** For
   example::

     export PFUNIT=/usr/local/pfunit/pfunit-mpi

#. Build pFUnit::

     mkdir build
     cd build
     cmake -DMPI=YES -DOPENMP=YES ..
     make -j 4

#. Run pFUnit's own unit tests::

     make tests

#. Install pFUnit on your system::

     make install INSTALL_DIR=$PFUNIT

One-time setup for this ``$UNITTEST_ROOT``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The first time you test a given code checkout, you need to copy files into
``$UNITTEST_ROOT`` that provide the build configurations specific to your
machine. **Note that this is a temporary measure which we hope to soon replace
with more robust out-of-the-box support for user-defined machines.**

These files can be found in the ``local_machine_build_scripts`` subdirectory of
this repository. Copy these files into ``$UNITTEST_ROOT`` and configure the file
``CESM_Macros.cmake`` for your machine. (In principle, you should not need to
modify ``Makefile.utest``.)

Building and running the unit tests
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

First ``cd`` to ``$UNITTEST_ROOT``.

The first time you build the unit tests in this location, run::

  make -j 4 -f Makefile.utest CIMEROOT=${CIMEROOT} config

You can also rerun that command to clean out the unit test build and start from
scratch.

Then, to build and run the unit tests, run::

  make -j 4 -f Makefile.utest CIMEROOT=${CIMEROOT} test

For subsequent builds, you can just run the ``make ... test`` command, without
first running ``make ... config``.

Note that the build is done in the directory ``$UNITTEST_ROOT/build``.

Determining if unit tests successfully built and ran
----------------------------------------------------

If the build was successful, you should get a message that looks like this::

  ==================================================
  Running CTest tests for __command_line_test__/__command_line_test__.
  ==================================================

Followed by a list of tests. Most (if not all) should pass. You should then see
a final message like this::

  100% tests passed, 0 tests failed out of 16

If just one or two tests fail, this could mean that these tests are currently
broken in the version of the code you're using. **Note that all CAM unit tests
are broken on the trunk at least as of cam5_4_51 (and for many previous tags).**

