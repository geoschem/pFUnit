# pFUnit 4.0

pFUnit is a unit testing framework enabling JUnit-like testing of
serial and MPI-parallel software written in Fortran.  Limited suport
for OpenMP is also provided in the form of managing exceptions in a
thread-safe manner.

pFUnit uses several advanced Fortran features, especially object
oriented capabilities, to offer a convenient, lightweight mechanism
for Fortran developers to create and run software tests that specify
the desired behavior for a given piece of code. 

This framework was originally created by developers from NASA and NGC
TASC.

### Table of contents

1. [Prerequisites](#prerequisites)
2. [Obtaining pFUnit](#obtaining-pfunit)
3. [What's in the directory?](#whats-in-the-directory)
4. [Building and installing pFUnit](#building-and-installing-pfunit)
6. [Using pFUnit in your application](#using-pfunit)
5. [Command line options](#command-line-options)
6. [Acknowledgments](#acknowledgments)
7. [Revisions](#revisions-to-this-document)

## Prerequisites

pFUnit is has been ported to Linux and Apple OS X.  Users have also
contributed support for Windows/CYGWIN, but please note that updates are
not regularly verified in that environment.

Due to the heavy use of F2003 object oriented features and a
smattering of F2008 features, only relatively recent Fortran compilers
are able to correctly build pFUnit.  The list below is not
comprehensive, and earlier compilers may work for some user code.

- Fortran compilers:
  - Intel v18.0.5, v19.0.3
  - NAG v6.2
  - GFortran v8.3, v9.2
- CMake v3.12+
- Python v2.7* or v3.x
- GNU make
- Optional
  - MPI (tested with OpenMPI and iMPI)
  - OpenMP
  
Doxygen was used to genate documention for pFUnit 3, but is not being
maintained.   My intent instead is to leverage the GitHub wiki to better
maintain features.

**Note that support for Python 2.7 my soon disappea.**


## Obtaining pFUnit

The best way to obtain pFUnit is to clone the git repository from
GitHub as follows:

    git clone https://github.com/Goddard-Fortran-Ecosystem/pFUnit.git

This will create the directory pFUnit in the current working
directory.

You can download the latest version as a tarball from the [release
page](https://github.com/Goddard-Fortran-Ecosystem/pFUnit/releases/latest),
and extract it like:

    $ tar zxf ./pFUnit-*.tar.gz

which will place the pFUnit files into a directory called
`pFUnit-<version>`, where `<version>` is the version number.

For other ways to acquire the code visit

    https://github.com/Goddard-Fortran-Ecosystem/pFUnit/

or contact the pFUnit team.

*Note: pFUnit uses 3 other open source packages via git submodules.
These are automatically downloaded during the build process if not
detected.  But this means that a default pFUnit clone must to be built
in an environment connected to the internet.  If the --recursive
option is used for the clone, then an internet connection should not
be necessary.

The other consequence of this is that the GitHub autogenerated release
tar balls do not contain the submodules at all.  I generally try to
provide comprehensive tar balls alongside the GitHub generated ones, but it
is apparentyl impossible to delete the automatic ones.*


## What's in the directory?

In the top level of the pFUnit distribution you will see the following
files.

- `CMakeLists.txt` - Initial support for cmake-based builds.

- `COPYRIGHT` - Contains information pertaining to the use and
  distribution of pFUnit.
  
- `ChangeLog` - history of changes

- `Examples` - Contains examples of how to use pFUnit once it is
  installed.   These are deprecated.  Users should instead go to the
  pFUnit-demos repository.

- `LICENSE` - The NASA Open Source Agreement for GSC-15,137-1 F-UNIT,
  also known as pFUnit.

- `README.md` - This file.

- `VERSION` - Contains a string describing the current version of the framework.

- `bin` - Executables used to construct and perform unit tests.

- `documentation` - Provides information about the pFUnit.  (Very out of date.)

- `extern` - external dependencies installed as git submodules

- `include` - Files to be included into makefiles or source, including use
  code.

- `src` - Source code and scripts of the pFUnit library and framework.

- `tests` - Source code for unit testing pFUnit itself.

- `tools` - Tools used to help develop, build, and install pFUnit.

## Building and installing pFUnit

pFUnit is now exclusively built and installed via CMake.  Very little
needs to be done to configure the build, however there are some
settings that are mandatory

- `FC` - used by CMake to identify your Fortran compiler.  If you
  know CMake, then there are other mechanisms to provide an explicit
  path to your compiler.

- `mpif90` - If using MPI, it must be in a location that CMake's
  FindMPI can find.  Usually having mpif90 in your path should
  suffice.
  
Note that pFUnit 4.0 handles MPI support differently than for 3.0.
Previously one explictly built and installed pFUnit separately for
serial and parallel configurations.  4.0 instead builds and install
both with a separate library for the MPI features.  E.g., a serial
application now links to libfunit.a, while a parallel application
links to `libpfunit.a` *and_ `libfpnut.a`.
  
CMake options:

- `MAX_ASSERT_RANK` - Control maximum rank for assertion overloads. It
  is 5 by default which is sufficient for most applications.  Setting
  higher may signifcantly increase build time.
- `SKIP_MPI` - Set to YES to skip building MPI support
- `SKIP_OPENMP` - Set to YES to skip building OpenMP support
- `SKIP_FHAMCREST` - Set to YES to skip building hamcrest support.
  This is a relatively new layer and is esp. taxing on compilers.
- `SKIP_ESMF` - Set to YES to skip building ESMF support.  Actually
  this is the current default as I have not yet properly integrated the ESMF
  extensions.  It should be easy to complete if there is any demand.



The process for building pFUnit using CMake is as follows. In the
top directory of the distribution make a new directory to support the
build, then change to that directory and run cmake (pointing back to
the source) to generate the required makefiles.

    $ mkdir build
    $ cd build
    $ cmake ..
    $ make tests
		$ make install


By default, pFUnit will put the installation in a subdirectory
called `installed` inside the build directory.  This unusual
default (from a CMake perspective) is due to the fact that
many/most pFUnit users lack elevated priviges for installing in the
usual places.  Most users will want to explicitly override the
default with CMake's `-DCMAKE_INSTALL_PREFIX=...`

## INSTALLATION

Because many pFUnit users lack permissions to install 
in the cmake default installation path, pFUnit instead by default
installs in the "installed" subdirectory in the main build directory.  
This can be easily overridden via  `CMAKE_INSTALL_PREFIX` on the cmake
command line.  Note that cmake allows a few variants on the internal structure
of the install, and future versions of pFUnit are likely to switch to a different 
variant.  (`find_package(pFUnit)` should continue to work, but there may be 
issues with the search order for users with conflicting installs.)

## Using pFUnit

Please see https://github.com/Goddard-Fortran-Ecosystem/pFUnit_demos for
more details.


### Using pFUnit in a CMake project

First, one must tell CMake where to find your installation of pFUnit.
Typically this looks like:
```script
$ cmake .. -DCAMKE_PREFIX_PATH=<path-to-pfunit-install>
```

Next, within your CMakeLists.txt, enable pFUnit with the lines:
```cmake
find_package(PFUNIT REQUIRED)
enable_testing()
```

Then, for each test suite, use the `add_pfunit_ctest` macro.
E.g.,

```cmake
add_pfunit_ctest (my_tests
  TEST_SOURCES ${test_srcs}
  LINK_LIBRARIES sut # your application library
  )
```

The make step should then produce a test executable.  In the example
above it would be `my_tests`.   To run:
```script
$ ./my_tests
```


### Using pFUnit in a GNU make project

Somewher in your Makefile you should add the lines
```make
include $(PFUNIT_DIR)/PFUNIT-4.0/include/PFUNIT.mk
FFLAGS += $(PFUNIT_EXTRA_FFLAGS)
```

Note that the path to `PFUNIT.mk` may change in the future.

Then, for each test suite, specify a set of make variables prefixed by
your test suite name, and invoke the `make_pfunit_test` function.
E.g.:

```make
my_tests_TESTS := test_square.pf
my_tests_REGISTRY :=
my_tests_OTHER_SOURCES :=
my_tests_OTHER_LIBRARIES := -L. -lsut
my_tests_OTHER_INCS :=

$(eval $(call make_pfunit_test,my_tests))
```

Then to build and execute:
```script
make all
./my_tests
```

### Command Line Options

The executable test program provides several command line options,
when "include/driver.F90" is used, as it is automatically when using
the pFUnit preprocessor.

    -h or --help                    Prints this help message
    -d or -debug or --verbose       Provide debugging information.  Useful when a test crashes.
    -f or --filter <pattern>        Only run tests that match the specified substring
    -h or --help                    Print help message.
    -o or --output <outputfile>     Direct pFUnit messages to a file.
    -r or --runner <runner>         Specify a non default test runner. (Advanced)
    -s or -skip  <n>                Used internally.

For example to run all tests whose names start with "test_ABC":

    $ ./tests.x -f  test_ABC

## Acknowledgments

Thanks to the follwing for their review and comments: B. Van Aartsen, T. Clune.

- Windows/CYGWIN contributions from E. Lezar.

- PGI port contributions from M. Leair (PG Group).

- Other acknowledgments: S.P. Santos (NCAR), M. Hambley (UK Met Office),
  J. Krishna (ANL), J. Ebo David.

## REVISIONS TO THIS DOCUMENT
- 2019-1027 Major rewrite. TLC
- 2015-1210 Minor changes to documentation. MLR
- 2015-0608 Added note about PFUNIT_EXTRA_USAGE (from MH). MLR
- 2015-0508 Some PGI workarounds removed for PGI 15.4. MLR
- 2015-0420 Clarified PFUNIT_MAX_ARRAY_RANK note. MLR
- 2015-0320 PGI port workarounds, including examples. 3.1. MLR
- 2014-1211 Minor updates for 3.0.2. MLR
- 2014-1110, 2014-1031 Minor edits. MLR
- 2014-0915 Minor updates for 3.0.1. MLR
- 2014-0404 Updated for release of 3.0. TLC
- 2014-0131, 2014-0205. Updated. MLR
- 2013-1107. Minor edits. MLR
- 2013-1031. Added user contributed code for Windows/CYGWIN & IBM's XLF. MLR
- 2013-0830-1359. Minor corrections and added MPIF90 to 6.2. MLR
- 2013-0806-1345. Corrected git reference. Was using old URL. MLR
- 2013-0805. Initial draft. MLR