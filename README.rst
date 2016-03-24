**************************
CESM Unit Testing Tutorial
**************************

Author: Bill Sacks (sacks@ucar.edu)

Credits:

* Sean Santos, for putting together the unit testing infrastructure that we use
  in CESM

* Ben Andre, for putting together the initial local machine build scripts

.. sectnum::

.. contents::

General notes and organization of this tutorial repository
==========================================================

The instructions here assume that you are using a recent development version of
CESM, such as in the cesm1_5 beta series.

The file you are currently reading provides the main instructions for building,
running and writing unit tests.

There are also the following subdirectories:

* ``presentations`` contains presentations that have been given about unit
  testing in CESM

* ``source_code`` contains the source code used for this tutorial - both the
  function to be tested and the test code

* ``cmake_files`` contains the CMake-based build scripts needed to build the
  unit tests in this tutorial; these can also be used as templates for your own
  unit tests

* ``local_machine_build_scripts`` contains files that facilitate building and
  running the unit tests on your local desktop or laptop (rather than on
  yellowstone)

This tutorial provides instructions for setting up unit tests in any of the
components that currently have unit tests:

* clm
* cam
* csm_share
* driver_cpl

For the sake of this tutorial, you should pick one of these components, and
follow instructions specific to that component. The main differences are in
paths where you put files and execute the build & run command, and in the
details of the ``CMakeLists.txt`` file that is used for building each set of
unit tests (as described in the section, `Add a CMakeLists.txt file in your new
unit test directory`_).

If you want to create the first unit tests for some other component, I recommend
going through this tutorial for one of the existing components. For other
science components, CLM or CAM are probably the best examples. You may want to
look through both of them and determine which directory structure and build
setup you like best. You can then set up unit tests similarly in your own
component. The main additional thing you will need is a top-level
``CMakeLists.txt`` file. This is the ``CMakeLists.txt`` file that resides in
``$UNITTEST_ROOT`` (described in the section, `Building and running the existing
unit tests`_). Writing that top-level file is beyond the scope of this tutorial,
and requires some knowledge of CMake, but you should be able to get started by
using the CLM and CAM files as a guide.

Building and running the existing unit tests
============================================

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
      previous tags). It works in `this branch tag
      <https://svn-ccsm-models.cgd.ucar.edu/cam1/branch_tags/fix_unit_tests_with_cime_tags/fix_unit_tests_with_cime_n01_cam5_4_51>`_.

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

  $CIMEROOT/tools/unit_testing/run_tests.py --test-spec-dir=. --compiler=intel \
  --mpilib=mpich2 --mpirun-command=mpirun.lsf \
  --cmake-args=-DPAPI_LIB=/glade/apps/opt/papi/5.3.0/intel/12.1.5/lib64

Note that the build is done in the directory ``$UNITTEST_ROOT/__command_line_test__``.

Now skip ahead to the section, `Determining if the unit tests successfully built
and ran`_.

Building and running on your local machine
------------------------------------------

One-time setup for your machine
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Before you can run on your own machine, you need to install some
pre-requisites. These are basically the same prerequisites needed for building
CESM, plus pFUnit. These include:

* C and Fortran compilers

  * We recommend gfortran 4.9 or later

* MPI
* cmake 2.8 or later
* python 2.7 or later
* netcdf 4.3.2 or later (4.3.3.1 recommended)
* pFUnit 3.0 or later: details in `Installing pFUnit`_

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

You can also rerun that command to clean out an existing unit test build and
start from scratch.

Then, to build and run the unit tests, run::

  make -j 4 -f Makefile.utest CIMEROOT=${CIMEROOT} test

For subsequent builds, you can just run the ``make ... test`` command, without
first running ``make ... config``.

Note that the build is done in the directory ``$UNITTEST_ROOT/build``.

Determining if the unit tests successfully built and ran
--------------------------------------------------------

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
If you want to write unit tests for CAM, you can use `this branch tag
<https://svn-ccsm-models.cgd.ucar.edu/cam1/branch_tags/fix_unit_tests_with_cime_tags/fix_unit_tests_with_cime_n01_cam5_4_51>`_.


Writing the unit tests
======================

Source code for this tutorial
-----------------------------

For the sake of this tutorial, we will test the ``circle_area`` function defined
in the file ``circle.F90`` in the subdirectory ``source_code``.

Copy the file ``source_code/circle.F90`` into the source tree of the component
you are interested in unit testing.

Let's use the following directories (pick one, based on which component you're
interested in unit testing, and copy ``circle.F90`` there):

* clm: components/clm/src/main
* cam: components/cam/src/utils
* csm_share: cime/share/csm_share/shr
* driver_cpl: cime/driver_cpl/driver

Creating a directory for your unit tests
----------------------------------------

Because of the way our unit test build system is set up, it works best to have a
separate directory for each collection of unit tests. This collection often
includes tests of a single module / file in the production code, but it could
also be a group of related modules.

For this tutorial, you will create a directory named ``circle_test``. Where you
should put this differs for each component:

* clm: components/clm/src/main/test/

  * Unit tests live in the ``test`` subdirectory of the directory containing the
    code they are testing

* cam: components/cam/test/unit

  * Unit tests are all together in this directory

* csm_share: cime/share/csm_share/test/unit

  * Unit tests are all together in this directory

* driver_cpl: cime/driver_cpl/unit_test

  * Unit tests are all together in this directory

Create a directory named ``circle_test`` as a subdirectory of one of the above
directories (for whichever component you're interested in unit testing).

The unit tests themselves
-------------------------

For the sake of this tutorial, we will use a set of unit tests that have already
been written for ``circle_area``.

Copy the file ``source_code/test_circle.pf`` into the directory you created
above.

Note that the ``.pf`` extension marks this as a file that should be
processed by the pFUnit pre-processor. This is basically Fortran code, but with
a few pFUnit-specific directives, which start with ``@``.

Read through that file, and try to understand how the tests are set up. If you
haven't done any object-oriented programming using Fortran2003 before, then
don't feel a need to understand the TestCircle class for now. (A ``class`` is
basically like a ``type`` in Fortran, but it can also have procedures -
functions and subroutines - in addition to data.) Pay particular attention to
the two subroutines that are preceded by the ``@Test`` directive: these are the two
tests we will run against the ``circle_area`` function.

For more information on writing your own tests, see the section, `More details
on writing your own unit tests`_.

Building and running your new unit tests
========================================

We build the unit tests using a build system called CMake. There are a few steps
needed to get your new unit tests to build alongside the others:

#. `Add the new production module to the build system`_

#. `Tell CMake about your new unit test directory`_

#. `Add a CMakeLists.txt file in your new unit test directory`_

This might look complicated, but once you have done it a few times, it should
only take a few minutes.

Add the new production module to the build system
-------------------------------------------------

You must first tell CMake about the new source code you have written - i.e., the
production module (not the test code).

If the source code directory already contains CMakeLists.txt
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Look in the directory where you added the source code (if you're doing the
example in this tutorial, this is the directory where you added circle.F90). If
this directory already has a ``CMakeLists.txt`` file (which should be the case
if you used one of the directories suggested above), then simply add your new
file - ``circle.F90`` in this example - to the list of source files in this
``CMakeLists.txt`` file.

In the case of csm_share, there are multiple source lists in
``CMakeLists.txt``. You should add the new file to the ``share_sources`` list.

If the source code directory does not yet contain CMakeLists.txt
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If this directory does *not* already have a ``CMakeLists.txt`` file, you will
need to add one. Follow the example of other ``CMakeLists.txt`` files for the
component you're working with. In addition, you will need to add an
``add_subdirectory`` call in the top-level ``CMakeLists.txt`` file in
``$UNITTEST_ROOT``. For example, if you have added source code in
``components/cam/src/control``, then you will need to create a
``CMakeLists.txt`` file in that directory containing these lines::

  list(APPEND cam_sources your_new_file.F90)
  sourcelist_to_parent(cam_sources)

and you will need to add the following line in
``components/cam/test/unit/CMakeLists.txt``::

  add_subdirectory(${CAMROOT}src/control control_cam)

Adding dependencies
^^^^^^^^^^^^^^^^^^^

If your new module depends on other modules (via use statements), either
directly or indirectly, then those must also be included in the unit test
build, following the same instructions as above. I generally just try building
the unit tests and seeing if the build complains: it will tell you about any
missing dependencies.

This step should not be needed for the example in this tutorial.

Testing the build
^^^^^^^^^^^^^^^^^

If you'd like, you can test the build at this point (following the instructions
under `Building and running the existing unit tests`_) before going on to the
next step.

Tell CMake about your new unit test directory
---------------------------------------------

Add an ``add_subdirectory`` call in the appropriate ``CMakeLists.txt`` file to
add the new unit test directory. For the ``circle_test`` example, this looks
like::

  add_subdirectory(circle_test)

This should be added in the ``CMakeLists.txt`` file in the parent directory of
your new test directory. For this example, this is:

* For CLM: components/clm/src/main/test/CMakeLists.txt

* For CAM: components/cam/test/unit/CMakeLists.txt

* For csm_share: cime/share/csm_share/test/unit/CMakeLists.txt

* For driver_cpl: cime/driver_cpl/unit_test/CMakeLists.txt

You can add the new ``add_subdirectory`` call at the bottom of the file.

Special note for CLM
^^^^^^^^^^^^^^^^^^^^

For CLM: If your unit tests are in a new subdirectory that didn't have any tests
before (e.g., ``cpl``), then you will also need to add an ``add_subdirectory``
call at the bottom of ``components/clm/src/CMakeLists.txt``, as in::

  add_subdirectory(${CLM_ROOT}/src/cpl/test clm_cpl_test)

This should **not** be needed if you used the recommended location in this
tutorial.

Add a CMakeLists.txt file in your new unit test directory
---------------------------------------------------------

You need to put a ``CMakeLists.txt`` file in your new unit test directory, which
tells CMake how to build this unit test. For this ``circle_test`` example, you
can copy one of the files from the ``cmake_files`` subdirectory of this
repository. Pick the file matching the component you are testing (e.g.,
``CMakeLists_cam.txt`` if you're doing this example for CAM). Copy this file
into your new unit test directory (the directory containing
``test_circle.pf``). **Rename the file to just CMakeLists.txt.**

The main difference between the components is whether each unit test explicitly
lists the source files that it depends on (currently done for CAM and
csm_share), or all unit tests link against an already-built library (currently
done for CLM and driver_cpl). There are pros and cons of each approach; for now,
just follow the style of whatever component you're writing unit tests for.

When you write your own unit tests, you can use the appropriate ``CMakeLists``
file as a template. You will need to replace any names that refer to ``circle``;
other than that, these templates should work without modification in most cases.

Run your new unit tests
-----------------------

Finally you're ready to build and run your new unit tests!

Follow the instructions under `Building and running the existing unit
tests`_. If all goes well, you should see the ``circle`` test suite listed
somewhere in the list of tests, and it should be listed as having ``Passed``.

More details on writing your own unit tests
===========================================

General guidelines for writing unit tests
-----------------------------------------

Unit tests typically test a small piece of code (e.g., order 10 - 100 lines,
such as a single function or small-ish class).

Good unit tests are "FIRST"
(https://pragprog.com/magazines/2012-01/unit-tests-are-first):

* Fast (order milliseconds or less)

  * This means that, generally, they should not do any file i/o. Also, if you
    are testing a complex function, test it with a simple set of inputs - not a
    10,000-element array that will require a few seconds of runtime to process.

* Independent

  * This means that test Y shouldn't depend on some global variable that was
    created by test X. Dependencies like this cause problems if the tests run in
    a different order, if one test is dropped, etc.

* Repeatable

  * This means, for example, that you shouldn't generate random numbers in your
    tests.

* Self-verifying

  * This means that you shouldn't write a test that writes out its answers for
    manual comparison. Tests should generate an automatic pass/fail result.

* Timely

  * This means that the tests should be written *before* the production code
    (Test Driven Development), or immediately afterwards - not six months later
    when it's time to finally merge your changes onto the trunk, and have
    forgotten the details of what you have written. Much of the benefit of unit
    tests comes from developing them alongside the production code.

Good unit tests test a single, well-defined condition. This generally means that
you make a single call to the function / subroutine that you're testing, with a
single set of inputs. This means that you usually need multiple tests of the
function / subroutine, in order to test all of its possible behaviors. The main
reasons for testing a single condition in each test are:

* This makes it easier to pinpoint a problem when a test fails
* This makes it easier to read and understand the tests, allowing the tests to
  serve as useful documentation of how the code should operate

A good unit test has four distinct pieces:

#. **Setup**: e.g., create variables that will be needed for the routine you're
   testing. For simple tests, this piece may be empty.

#. **Exercise**: Call the routine you're testing

#. **Verify**: Call assertion methods to ensure that the results matched what
   you expected

#. **Teardown**: e.g., deallocate variables. For simple tests, this piece may be
   empty. **However, if this is needed, you should almost always do this
   teardown in the special tearDown routine, as discussed in the sections,**
   `Defining a test class in order to define setUp and tearDown methods`_ and
   `More on test teardown`_.

pFUnit provides many assertion methods that you can use in the Verify step. Some
of the most useful are the following:

* ``@assertEqual(expected, actual)``

  * Ensures that expected == actual

  * Accepts an optional ``tolerance`` argument giving the tolerance for
    real-valued comparisons

* ``@assertLessThan(expected, actual)``

  * Ensures that expected < actual

* ``@assertGreaterThan(expected, actual)``

  * Ensures that expected > actual

* ``@assertLessThanOrEqual(expected, actual)``

* ``@assertGreaterThanOrEqual(expected, actual)``

* ``@assertTrue(condition)``

  * It's better to use the two-valued assertions above, if possible. For
    example, use ``@assertEqual(foo, bar)`` rather than ``@assertTrue(foo ==
    bar)``: the former gives more information if the test fails.

* ``@assertFalse(condition)``

* ``@assertIsFinite(value)``

  * Ensures that the result is not NaN or infinity

* ``@assertIsNan(value)``

  * Can be useful for failure checking, e.g., if your function returns NaN to
    signal an error

Comparison assertions accept an optional ``tolerance`` argument, which gives the
tolerance for real-valued comparisons.

In addition, all of the assertion methods accept an optional ``message``
argument, which gives a string that will be printed if the assertion fails. If
no message is provided, you will be pointed to the file and line number of the
failed assertion.

If you have many tests of the same subroutine, then you'll often find quite a
lot of duplication between the tests. It's good practice to extract major areas
of duplication to their own subroutines in the .pf file, which can be called by
your tests. This aids the understandability and maintainability of your
tests. pFUnit knows which subroutines are tests and which are "helper" routines
because of the ``@Test`` directives: You only add a ``@Test`` directive for your
tests, not for your helper routines.

Defining a test class in order to define setUp and tearDown methods
-------------------------------------------------------------------

As noted in the comments in ``test_circle.pf``, the definition of a test class
(here, ``TestCircle``) is optional. I generally go ahead and define a minimal
test class when I first write a new .pf file::

  @TestCase
  type, extends(TestCase) :: TestCircle
   contains
     procedure :: setUp
     procedure :: tearDown
  end type TestCircle

Defining this test class allows you to take advantage of some useful pFUnit
features like the setUp and tearDown methods.

If you define this test class, then you also need to:

* Define setUp and tearDown subroutines. These can start out empty::

    subroutine setUp(this)
      class(TestCircle), intent(inout) :: this
    end subroutine setUp

    subroutine tearDown(this)
      class(TestCircle), intent(inout) :: this
    end subroutine tearDown

* Add an argument to each test subroutine, of class ``TestCircle`` (or whatever
  you called your test class). By convention, this argument is named ``this``.

Code in the setUp method will be executed before each test. This is convenient
if you need to do some setup that is the same for every test.

Code in the tearDown method will be executed after each test. This is often used
to deallocate memory. See the section, `More on test teardown`_ for details.

You can add any data or procedures to the test class. Adding data is
particularly useful, as this can be a way for the setUp and tearDown methods to
interact with your tests: The setUp method can fill a class variable with data,
which can then be used by your tests (accessed via
``this%somedata``). Conversely, if you want the tearDown method to deallocate a
variable, that variable cannot be local to your test subroutine. Instead, you
can make the variable a member of the class, so that the tearDown method can
access it.

So, for example, if you have this variable in your test class (as in the
example)::

  real(r8), pointer :: somedata(:)

Then ``somedata`` can be created in the setUp method (if it needs to be the same
for every test). Alternatively, it can be created in each test routine that
needs it (if it differs from test to test, or some tests don't need it at
all). Its creation can look like::

  allocate(this%somedata(5))
  this%somedata(:) = [1,2,3,4,5]

Then your tearDown method can have code like this::

  if (associated(this%somedata)) then
    deallocate(this%somedata)
  end if

More on test teardown
---------------------

All of the tests in a single test executable - which, for CESM, typically means
all of the tests defined in all ``.pf`` files in a single test directory - will
execute one after another in one run of the executable. This means that, if you
don't clean up after yourself, tests can interact with each other. In the best
case, this can mean you get a memory leak. In the worst case, it can mean that
the pass / fail status of tests depends on what other tests have run before
them, making your unit tests unrepeatable and unreliable. **As a general rule,
you should deallocate any pointers that your test allocated, reset any global
variables to some known, initial state, and do other, similar cleanup for
resources that may be shared by multiple tests.**

As described in the section, `Defining a test class in order to define setUp and
tearDown methods`_, code in the tearDown method will be executed after each
test. This is often used to do cleanup operations after each test. **Any
teardown like this should generally happen in this tearDown method. This is
because, if an assertion fails, the test aborts. So any teardown code in the
test method (following the failed assert statement) is skipped, which can lead
other tests to fail or give unexpected results. But this tearDown method is
still called in this case, making it a safe place to put teardown that needs to
be done regardless of whether the test passed or failed (which is the case for
most teardown).** In order for this to work, you sometimes need to move
variables that might otherwise be subroutine-local to the class - because the
tearDown method can access class instance variables, but not subroutine-local
variables.

Note that, in Fortran2003, allocatable variables are automatically deallocated
when they go out of scope, but pointers are not. So you need to explicitly
deallocate any pointers that have been allocated, either in test setup or in the
execution of the routine you're testing.

CESM makes extensive use of global variables: variables declared in some module,
which may be used (directly or indirectly) by the routine you're testing. If
your test has allocated or modified any global variables, it is important to
reset them to their initial state in the teardown portion of the
test. (Incidentally, this is just one of many reasons to prefer explicit
argument-passing over the use of global variables.)

pFUnit documentation and examples
---------------------------------

Some pFUnit documentation is available here: http://pfunit.sourceforge.net/

If you download pFUnit (from
http://sourceforge.net/projects/pfunit/files/latest/download), you can find more
extensive documentation and examples in the following places. Among other
things, this can show you other assertion methods that are available:

* documentation/pFUnit3-ReferenceManual.pdf

* Examples/

* tests/

  * These are tests of the pFUnit code itself, written in pFUnit. You can see
    many uses of pFUnit features in these tests.


Finding more documentation and examples in CESM
===============================================

Documentation of the unit test build system
-------------------------------------------

The CMake build infrastructure is in ``cime/externals/CMake``.

The infrastructure for building and running tests with ``run_tests.py`` is in
``cime/tools/unit_testing``. That directory also contains some general
documentation about how to use the CESM unit test infrastructure (in the
``README`` file), and examples (in the ``Examples`` directory).

Finding more detailed examples
------------------------------

At this point, there are many examples of unit tests in CESM, some simple and
some quite complex. You can find these by looking for files with the '.pf'
extension::

  find . -name '*.pf'

You can also see examples of the unit test build scripts by viewing the
CMakeLists.txt files throughout the source tree.

  
