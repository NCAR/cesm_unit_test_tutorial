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
these instructions assume that you have first defined the environment variable**
``CIMEROOT``. This should point to the cime directory associated with your
current CESM checkout. e.g., for bash, you would use::

  export CIMEROOT=/path/to/source_code/cime


General notes
-------------

The initial unit test build for a component might take a few minutes. However,
rebuilds from the same directory should be much faster.

Building and running on yellowstone
-----------------------------------

First ``cd`` to the directory containing the top-level ``CMakeLists.txt`` file
for the component you want to unit test. This is:

* clm: components/clm/src
* cam: FIXME
* csm_share: FIXME
* driver_cpl: FIXME

You must be in an interactive session on caldera to build and run the unit
tests. The easiest way to do this is to run::

  execca

and wait for an interactive prompt.

Then run the following command::

  $CIMEROOT/tools/unit_testing/run_tests.py --test-spec-dir=. --compiler=intel --mpilib=mpich2 \
  --mpirun-command=mpirun.lsf --cmake-args=-DPAPI_LIB=/glade/apps/opt/papi/5.3.0/intel/12.1.5/lib64
