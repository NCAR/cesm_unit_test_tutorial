Building and Running CLM Unit Tests
===================================

Before you write your own unit tests, you should make sure that you can build
and run the existing tests.

Here we document two ways to do this:

* On yellowstone - easiest, but slower to build and run the tests

* On your personal desktop / laptop - takes more one-time setup work on your
  part, but once it is set up, this will let you build and run the unit tests
  much faster. This also facilitates doing code development on your personal
  machine, rather than remotely.

General notes
-------------

The initial unit test build for a component might take a few minutes. However,
rebuilds from the same directory should be much faster.

Building and running on yellowstone
-----------------------------------

First ``cd`` to the CLM source code directory::

  cd components/clm/src

You must be in an interactive session on caldera to build and run the unit
tests. The easiest way to do this is to run::

  execca

and wait for an interactive prompt.

Then run the following command::

  ../../../cime/tools/unit_testing/run_tests.py --test-spec-dir=. --compiler=intel --mpilib=mpich2 \
  --mpirun-command=mpirun.lsf --cmake-args=-DPAPI_LIB=/glade/apps/opt/papi/5.3.0/intel/12.1.5/lib64


