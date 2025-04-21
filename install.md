# Installation procedure

The recipe detailed below is intended for “regular users”, who downloaded a version of the code from a current stable release.
You may check in parallel the latest upstream version from [the CepGen repository on GitHub](https://github.com/cepgen/cepgen).

## General and usage-specific dependencies

CepGen was designed to require only a limited set of external dependencies for its core features.

Among the mandatory dependencies,

- [CMake](https://cmake.org/) for the management of the build process,
- [GSL](https://www.gnu.org/software/gsl/), for the integrators definition and general utilitaries.
  A version greater than or equal to `2.1` is recommended for the bilinear spline interpolation capability.
  This latter is used in e.g. the MSTW structure functions and KMR $\kt$-factorised gluon flux definitions.

Depending on your system architecture and distribution, these dependencies may be installed the following way:

````{note}
For Debian/Ubuntu, use the following

```sh
sudo apt-get install cmake
sudo apt-get install g++ gfortran
sudo apt-get install libgsl2 libgsl-dev
```

For RHEL/Fedora/CentOS, use the following
```sh
sudo dnf install cmake
sudo dnf install gcc-c++ gcc-gfortran
sudo dnf install gsl gsl-devel
```
````

It can further be extended through a set of add-ons loaded through shared libraries into the runtime environment.

For instance, it is highly recommended to install the following dependencies to ease your first steps:

- `CepGenPython`: complex Python version 2 or 3 steering cards parsing ; it requires the `python-devel` or `libpython-dev` development headers to be built,
- `CepGenLHAPDF` with [LHAPDF](https://lhapdf.hepforge.org/) version 5 or above, for definition of partonic-level nucleon structure functions, and its $\alpha_S(Q)$ strong coupling constant evolution algorithm.

A set of add-ons specifically designed to ease your workflow when generating and storing events is also provided, given the following libraries are available on your system:

- `CepGenPythia8`: [Pythia](https://pythia.org) version 8.1 or above, for the various particles decays and excited proton fragmentation, but also the LHEF event format generation,
- `CepGenPythia6`: [Pythia](https://pythia.org/pythia6) version 6, for the legacy steering of particles decays and excited proton fragmentation,
- `CepGenHepMC2` and `CepGenHepMC3` for the [HepMC](https://hepmc.web.cern.ch/hepmc/) version ≥ 2, to handle its various ASCII output formats,

  ```{note}
  From version 3 on, a LHEF output is also supported.
  If not present, this latter format can be handled through the Pythia 8 interface instead.
  ```

- `CepGenROOT`: for the [ROOT](https://root.cern.ch/) ntuples building/histogramming/plotting capabilities widely used in HEP.

## CepGen installation

Start by downloading the latest release in the bucket list of [releases](https://github.com/cepgen/cepgen/releases).
Unpack the sources in a location referred here as: `$CEPGEN_SOURCES`.

For instance, for a CepGen version `x.y.z` package:

```sh
tar xvfz cepgen-x.y.z.tar.gz
```

Once you have set up the sources and downloaded/installed the required dependencies, create your building environment (following the [CMake convention](https://cmake.org/cmake/help/latest/guide/tutorial/A%20Basic%20Starting%20Point.html#exercise-1-building-a-basic-project), this latter is usually the `build/` directory).
The compilation, and (optional, although very recommended) system installation is hence done with:

```sh
mkdir build && cd build
cmake [-DCMAKE_INSTALL_PREFIX=/path/to/install] /path/to/cepgen/sources/
make # optionally, add -jN (N=number of parallel threads for compilation)
make install
```

Optionally, if [Ninja](https://ninja-build.org/) is installed on your system, you may add the extra `-GNinja` flag to the aforementioned `cmake` command, and launch the build using the following command:

```sh
cmake [-DCMAKE_INSTALL_PREFIX=/path/to/install] -GNinja /path/to/cepgen/sources/
ninja
ninja install
```

```{note}
As documented [elsewhere](https://cmake.org/cmake/help/latest/variable/CMAKE_INSTALL_PREFIX.html), the `CMAKE_INSTALL_PREFIX` CMake directive allows to specify manually the installation path for the full (libraries + headers) products.

Typically, all CepGen headers (resp. libraries) will be installed in the `/path/to/install/include` (resp. `/path/to/install/lib` or `/path/to/install/lib64`) directories, with a few external dependencies (e.g. PDG catalogue, interpolation grids) in the `/path/to/install/share/CepGen` directory.
Additionally, a `FindCepGen.cmake` directive is automatically generated in the `/path/to/install/cmake` folder to ease its linking to external libraries/user code.
```

This compilation will build a collection of required sub-libraries to be linked against any executable built on top of CepGen.:

- `libCepGen` contains all physics constants, calculators, helpers, along with standard objects implementation, nucleon structure function calculators objects, the definition of events and subleading particles objects (useful for analyses of CepGen outputs), and input cards definition and handling parts ;
- `libCepGenProcesses` contains all processes definitions and implementations.

In addition, upon their availability on your system at compilation time, the collection of `libCepGenXXX` add-ons described above will be generated for your convenience.

As described [here](/usage.md), several test executables can be linked against the CepGen libraries.

```{note}
You may build these executables using the `CMAKE_BUILD_TESTS=1` configuration flag when running your CMake generation command.
All compiled executable will then be located either in the `test/` directory of your build environment, for the general-purpose tests, or directly in the `CepGenXXX/` folder in your build environment for the add-ons-specific tests.

These tests can all be triggered using e.g. the [CTest](https://cmake.org/cmake/help/latest/manual/ctest.1.html) suite of CMake, running `ctest`.
```
