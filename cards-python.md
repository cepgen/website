# Python configurations

The default steering format for CepGen is Python (2/3) configuration files.

Thanks to the huge modularity Python allows (e.g. through `import` statements, dynamic typing, and large choice of widely-used modules), the configuration overhead is largely reduced, allowing the user to concentrate on the physics process definition.

The steering capability, and deployment of all overhead scripts relies on the `CepGenPython` add-ons library.
This latter obviously relies on the Python C API presence (`python-devel` or `python-dev` packages on RPM/DEB managers respectively) on the machine at compile or installation time.

Two core object types (both inheriting from Python’s `dict` containers) are imported in the scripting scope through the general statement:

```python
import Config.Core as cepgen
```

To ensure your environment is ready for this import, just open a Python shell and type the line above.
In case of failure, it might be useful to ensure the `CEPGEN_PATH` environment variable is well set to your CepGen sources path (or to `/usr/local` if you installed CepGen through pre-compiled packages).
You may also override the Python search path to let it know about the CepGen path too:

```bash
export PYTHONPATH=$CEPGEN_PATH:$PYTHONPATH
```

Once all set, you will have access to the two types described above: `Module`, and `Parameters`.
The earlier is a subset of the latter, with the first string-type attribute defining the module name.

The common usage for such a module definition is, for instance:

```python
import Config.Core as cepgen
module = cepgen.Module('my_first_module', foo = 'bar')
```

Additionally, as of CepGen version 0.9.7, a third base type, i.e. a sequence of modules (inherited from Python's `list` containers) is introduced.
It allows to define an ordered chain of modules to launch, for instance to modify the events content according to an external library or trigger a chain of output modules at each event generation.

In [this page](/python-containers.md), one can see an illustrated example of this `Module`/`Parameters`/`dict` and `Sequence`/`list` relations.

Several blocks are required to be defined in a CepGen steering card.
In the following sections, you may find a nonexhaustive (and evolving) list of such attributes.

## `integrator` module block

This collection of parameters allows the user to steer the cross section estimation part of the run.

As of version 0.8 of CepGen, the three GSL implementations of the following integration algorithms are supported:

- `Vegas` by Lepage {cite}`Lepage:1977sw`,
- `MISER` stratified sampling by Press et al. {cite}`Press:1989vk`,
- `plain` "hit-and-miss" algorithm.

Since then, several add-ons were introduced, to quote a few:

- a "`Naive`" Boost integrator, as documented in [the official Boost documentation](https://www.boost.org/doc/libs/1_81_0/libs/math/doc/html/math_toolkit/naive_monte_carlo.html),
- an interface to `ROOT`'s [ROOT::Math::IntegratorOneDim](https://root.cern.ch/doc/master/classROOT_1_1Math_1_1IntegratorOneDim.html) and [ROOT::Math::IntegratorMultiDim](https://root.cern.ch/doc/master/classROOT_1_1Math_1_1IntegratorMultiDim.html) general purpose MC integrator algorithms, and the more specific interface to the [TFoam](https://root.cern.ch/doc/master/classTFoam.html) implementation of the FOAM algorithm {cite:p}`Jadach:2002kn`,
- the various interfaces to the integrators of the `Cuba` suite: `cuba_vegas`, `cuba_suave`, `cuba_divonne`, and `cuba_cuhre`, as documented in [the official Cuba library documentation from FeynArts](https://feynarts.de/cuba/).

### Integration modules parameters

````{toggle}
```{doxygennamespace} python::Config::Integration
:members:
```
````

### Usage

To import one of these modules in your steering card, we advise you to load it into a private variable, and modify its attribute through cloning (to avoid strange behaviours in complex chains with Python's reference modification).
For instance, to import (and steer) the Vegas algorithm:

```python
from Config.Integration.vegas_cfi import vegas as _integ
# ...
integrator = _integ.clone(
    iterations = 5,
    verbose = 0,
    # ...
)
```

But you can also define the module directly (instead of the Python-steered configuration loaded through the `import` method above, the [default module parameters](/raw-modules.md#integr) will be loaded instead)

## `generator` module block

This {cepgen}`collection of parameters <python::Config::generator_cfi::generator>` allows the user to steer the event generation part of the run.

### Variable definition

```{doxygenvariable} python::Config::generator_cfi::generator
```

### Members

- `numEvents`: number of events to generate in this run
- `numPoints`: number of points to generate in integration time
- `printEvery`: period at which the event content will be dump in the terminal when generating events
- `numThreads`: number of threads to use to generate events (in CepGen multithread mode)

## `eventSequence` sequence block

As of version 0.9.7 of CepGen, an ordered collection of modification algorithms can be triggered on an event-by-event basis
for the modification, hadronisation, correction, ... of the full event kinematics.

The full list and description of algorithms with an interfacing already implemented in CepGen [may be found here](/event-modifiers.md).

Being sequential, this block acts on a _first-come, first-served_ basis, hence if two hadronisers/decay modules are to be triggered
one after the other, the first defined in this sequence will act the first.

````{important}
Prior to version 0.9.7, this sequence was defined as a single `hadroniser` module.
This latter is still properly parsed for legacy configurations, but we encourage you to update your scripts
accordingly.

For instance:

```python
hadroniser = cepgen.Module('pythia8')
```

should become

```python
pythia = cepgen.Module('pythia8')
eventSequence = cepgen.Sequence(
    pythia,
)
```
````

(pdg-block)=

## `PDG` parameters block

The {mod}`PDG` module consists of a `PDG` container object holding the list of particles definitions is parsed by the CepGen core at the initialisation level, and utility functions.

The former allows to specify new PDG members and propagate their properties for its usage in all parts of the framework.

A single `registerParticle` utility function allows to add such a particle in one single line before the `process` block definition.

````{toggle}
```{doxygennamespace} python::Config::PDG_cfi
:members:
```
````

## `process` module block

This block comes as a required `Module` object defined in the general scope.
Its first feature is to specify the process to account for in the user-defined run.
See the list of processes section of the left hand side menu to find your model of interest.

### `process.inKinematics` parameters block

A `pz` Python pair (or list) of floating point numbers allows to specify the two incoming protons’ longitudinal momentum (in GeV).
The `cmEnergy` keyword can also be used to define directly the centre of mass energy $\sqrt{s}$ of the two incoming beams for symmetric, head-on collisions.
In that latter case, $p _ {z,1-2} = \pm \sqrt{s}/2$.

Equivalently, a `pdgIds` pair/list of [integer-type PDG identifiers](http://pdg.lbl.gov/2007/reviews/montecarlorpp.pdf) (complete list handled {ref}`here <pdg-block>`) may be used to control beam particles type.
A default `pdgIds = (2212, 2212)` initial state, or equivalently `(PDG.proton, PDG.proton)`, is used.

````{toggle}
```{doxygenclass} python::Config::StructureFunctions_cff::StructureFunctions
:members:
:undoc-members:
```
````

The `structureFunctions` attribute specifies the $F _ {2/L}(\xbj,Q^2)$ structure function to use in the parameterisation of the incoming photon fluxes.
The name of the structure functions set (see [the complete list here](/structure-functions.md)) has to be prepended by `StructureFunctions`

For instance, the *Suri-Yennie* set may either be selected through the `StructureFunctions.SuriYennie` enum value, or its numeric code `11`.

### `process.outKinematics` parameters block

The kinematics phase space to be used in the integration and events production can be specified using a set of cuts applied on the matrix element level:

- `pt`: single central particle transverse momentum range definition,

- `energy`: single central particle energy range definition,

- `eta`: single central particle pseudo-rapidity range definition,

- `rapidity`: single central particle rapidity range definition,

- `mx`: outgoing excited proton mass range definition,

- `xi`: outgoing proton fractional longitudinal momentum loss $\xi = \Delta p/p$.

  ```{versionadded} 0.9.2
  ```

### `process.processParameters` parameters block

This block is a generic placeholder for all process-dependent parameters.
The usual basic parameter for this block is the process mode, which tells about the type of kinematics to be considered when defining its phase space:

```{doxygenclass} python::Config::ProcessMode_cff::ProcessMode
:members:
:undoc-members:
```

See the description page of each process to get a list of supported parameters to include in this collection.

## `output` module/sequence block

Like the `integrator` block described above, the steering of output module(s) is operated through the import or definition of an `output` module or sequence.


(configuration-card-example-python)=

## Configuration card example

The generation of 100k single-dissociative $\gg{\mu^+\mu^-}$ events at 13 TeV with the [LPAIR matrix element](/processes-lpair.md) implementation with the following phase space cuts:

- $\pt(\mu^\pm)>$ 25 GeV, $\lvert\eta(\mu^\pm)\rvert< $ 2.5
- 1.07 \$\< M_X \<\$ 1000 GeV

can be steered using the following card:

```python
import Config.Core as cepgen
from Config.Integration.vegas_cff import integrator
from Config.generator_cff import generator as gentmpl
from Config.PDG_cfi import PDG

process = cepgen.Module('lpair',
    processParameters = cepgen.Parameters(
        mode = cepgen.ProcessMode.InelasticElastic, # single-dissociation
        pair = PDG.muon, # or, equivalently, 13
    ),
    inKinematics = cepgen.Parameters(
        pz = (6500., 6500.), # or cmEnergy = 13.e3,
        structureFunctions = cepgen.StructureFunctions.SuriYennie,
    ),
    outKinematics = cepgen.Parameters(
        pt = (25., ),
        energy = (0., ),
        eta = (-2.5, 2.5),
        mx = (1.07, 1000.),
    )
)

generator = gentmpl.clone(
    numEvents = 1e5,
)

output = cepgen.Module('lhef',
    filename = 'lpair-example.lhef',
)
```

This configuration is equivalent to the *LPAIR card* shown [here](/cards-lpair.md#configuration-card-example).
