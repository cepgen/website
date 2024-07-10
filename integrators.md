# Monte Carlo numerical integrators

The modularity of CepGen also allows for multiple integration algorithms to be steered and profit for some increased numerical stability of a wise choice of parameters.
Several interfaces to external algorithms are provided in the core and `CepGenAddOns` libraries.

In the [Python](/cards-python.md) cards parsing these can be steered through the `integrator` keyword.
All modules are derived from a common {cepgen}`cepgen::Integrator` object, described below:

A full list of the algorithms and their parameters can be found [here](/raw-modules.md#integr).

```{doxygenclass} cepgen::Integrator
:outline:
```

Detailed description

````{toggle}
```{doxygenclass} cepgen::Integrator
:members:
:no-link:
```
````

______________________________________________________________________

## GSL Monte Carlo integrator algorithms

This category encompasses all "historical" integrator algorithms.
Relying on the GSL implementation of `gsl_monte_xxx` integration routines, they are all characterised by a set of algorithm-specific relevent parameters (see [the GSL manual](https://www.gnu.org/software/gsl/doc/html/montecarlo.html) for a description).

Currently, three modules are provided natively in CepGen:

- {cepgen}`cepgen::VegasIntegrator` for the Vegas algorithm by G. P. Lepage (also provided as a Python module, see below). The various steering parameters are listed [here](/raw-modules.md#integrVegas) ;
- {cepgen}`cepgen::MISERIntegrator` for the MISER algorithm. The various steering parameters are listed [here](/raw-modules.md#integrMISER) ;
- {cepgen}`cepgen::PlainIntegrator` for the "plain", trial/error algorithm. The various steering parameters are listed [here](/raw-modules.md#integrplain).

In CepGen, all interfacing modules are derivatives of the base {cepgen}`cepgen::GSLIntegrator` object:

```{doxygenclass} cepgen::GSLIntegrator
:private-members:
:outline:
```

## ROOT integration algorithms

```{versionadded} 0.9.10
```

This two-in-one {cepgen}`cepgen::root::Integrator` interfacing object allows the numerical integration of an integrand through the two ROOT numerical integration algorithms:

- a one-dimensional integrator (see [ROOT::Math::IntegratorOneDim](https://root.cern.ch/doc/master/classROOT_1_1Math_1_1IntegratorOneDim.html)) ;
- a multidimensional integrator (see [ROOT::Math::IntegratorMultiDim](https://root.cern.ch/doc/master/classROOT_1_1Math_1_1IntegratorMultiDim.html)).

According to the user's request, either of the two objects is populated and configured. The following parameters are to be steered by the end user:

- `type`: integrator algorithm type:
  : - `gauss`, `legendre`, `adaptive`, `adaptiveSingular`, `nonAdaptive`, for one-dimensional integration,
    - `adaptive`, `plain`, `miser`, `vegas` for multidimensional integration. The last three are one-to-one equivalent to the GSL Monte Carlo algorithms described above (except for the interface and parameters definition).
- `absToL`: desired absolute error ;
- `relToL`: desired relative error ;
- `size`: maximum number of sub-intervals.

### Foam integration algorithm

```{versionadded} 0.9.10
```

Whenever found in the ROOT installation path, an interface to the Foam algorithm {cite}`Jadach:2002kn` is also provided (either for numerical integration, or for unweighted event generation).
The interfacing object, {cepgen}`cepgen::FoamIntegrator`, can be steered using the parameters listed [here](/raw-modules.md#integrFoam).

## Python integration algorithms

```{versionadded} 1.2.0
```

The {cepgen}`cepgen::python::Integrator` Python extension module allows the interfacing between CepGen and any Python numerical integrator algorithm.
It relies on the definition of a Python wrapper/interfacing function of the form:

```python
def integrate(f,                # [](vector<double>) -> double, C++ integrand wrapper
              num_dim: int,     # number of dimensions to integrate
              num_iter: int,    # (optional) number of iterations for integration
              num_warmup: int,  # number of function calls for (optional) warmup
              num_calls: int,   # number of function calls at each iteration
              limits: list[tuple[float]]=[]  # list of (min, max) variables limits
              ):
    # definition of integration procedure
    # [...]
    return (average, standard_deviation)
```

Among centrally provided implementations of Python integrators wrapper (found in the `python/IntegrationAlgos` directory), one may quote:

- {cepgen}`MCint.py`, interface to the [MCint](https://pypi.org/project/mcint/) MC numerical integration tool ;
- {cepgen}`Scipy.py`, interface to the [`scipy.integrate`](https://docs.scipy.org/doc/scipy/reference/integrate.html) integration/ODE package of scipy ;
- {cepgen}`Torchquad.py`, interface to the [torchquad](https://torchquad.readthedocs.io) GPU multidimensional numerical integator based on PyTorch ;
- {cepgen}`Vegas.py`, interface to the Python version of the [vegas](https://vegas.readthedocs.io) integrator by G. P. Lepage.
