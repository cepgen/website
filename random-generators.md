# Random number generators

Several interfaces to external algorithms are provided in the core and `CepGenAddOns` libraries for the generation of random number according to some distributions of interests, and easily steerable through the `randomGenerator` sequential block (in [Python](/cards-python.md) cards).
All modules are derived from a common {cepgen}`cepgen::utils::RandomGenerator` object, described below:

A full list of the algorithms and their parameters can be found [here](/raw-modules.md#rndgen).

```{doxygenclass} cepgen::utils::RandomGenerator
:outline:
```

Detailed description

````{toggle}
```{doxygenclass} cepgen::utils::RandomGenerator
:members:
:no-link:
```
````

______________________________________________________________________

## GSL random number generator

```{versionadded} 1.2.0
```

```{doxygenclass} GSLRandomGenerator
:outline:
```

## ROOT random number generator

```{versionadded} 1.2.0
```

```{doxygenclass} cepgen::root::RandomGenerator
:outline:
```

## STL random number generator

```{versionadded} 1.2.0
```

```{doxygenclass} STLRandomGenerator
:outline:
```
