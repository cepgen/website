# Event content definition

## Event content object

A couple of specific objects containing the full information of a physics event at a given snapshot of its processing are defined.
The {cepgen}`cepgen::Event` object can be seen as an extended collection of {cepgen}`cepgen::Particle` components with the attributes defined hereafter.

### Particle properties

The {cepgen}`cepgen::Particle` object holds the minimal amount of information needed to define a particle: a momentum (as a {cepgen}`cepgen::Momentum` wrapper of the 4-momentum kinematics), some status (as a {cepgen}`cepgen::Particle::Status` enum) and role in event (the {cepgen}`cepgen::Particle::Role` enum) flags, a {cepgen}`cepgen::pdgid_t` PDG identifier, a mothers/daughters parentage, helicity, ...

The detailed description of the object's attributes can be shown below:

````{toggle}
```{doxygenclass} cepgen::Particle
:members:
```
````

### Event properties

In addition to a map between a particles' role in the event and a collection of all particles with this role, the event object {cepgen}`cepgen::Event` also handles a few useful accessors, helpers, and properties.
To quote a few:

- {cepgen}`cepgen::Event::cmEnergy` allows to compute the centre-of-mass energy of the incoming particles states,
- {cepgen}`cepgen::Event::compress` returns a `compressed` event, where all intermediate "vertex-like" states are skipped, only to keep the minimal 2-to-N event content,
- {cepgen}`cepgen::Event::missingMomentum` allows to compute the missing momentum from a "standard" detector's perspective (the sum of all invisible particles, such as neutrinos/neutralinos/...)

The detailed description of the object's methods and attributes can be found in {cepgen}`cepgen::Event`.

## Event accessors

The interaction with event content was simplified from CepGen version `1.1.0` to introduce a base placeholder object with the full knowledge of the {cepgen}`cepgen::Event` object.
This intermediate object, the {cepgen}`cepgen::EventHandler` provides a few base utilities, such as:

- an {cepgen}`cepgen::EventHandler::initialise` initialisation procedure to prepare the external algorithm ;
- an optional {cepgen}`cepgen::EventHandler::engine` algorithm engine retrieval method (if exists), to allow a more specific steering for complex cases.

````{toggle}
```{doxygenclass} cepgen::EventHandler
:members:
```
````

The implementations of event accessors can be classified into three categories: importers, modifiers, and exporters.

### Event importer

This category was introduced in CepGen [version `1.2.0`](/changelog.md#release-1-2-0), and allows to load {cepgen}`cepgen::Event` objects from an external source.
It can be useful when, for instance, CepGen is used as an interface to several modification algorithms (as described below), or to convert a dataset between two event formats.

````{toggle}
```{doxygenclass} cepgen::EventImporter
:members:
```
````

### Event modifier

Event modifiers are basic buiding blocks for e.g. hadronisation algorithms interfaces, or treatment algorithms to match one event content definition to another.
They are fully described in the [Event modification algorithms](/event-modifiers.md) section of this documentation.

### Event exporter

Event exporters, or output modules are providing an interface to major output formats definitions and writers.
Again, a full documentation of this sub-class can be found in the [Output formats](/output-formats.md) section.

### String-based event accessors
```{versionadded} 0.9.7
```

The {cepgen}`cepgen::utils::EventBrowser` object was introduced to allow an easier interaction to all its attributes, either at the event level, or for each of its individual particles (single, or combined) and momenta.

It can be used the following way:

```cpp
#include <CepGen/EventFilter/EventBrowser.h>

int main() {
    auto evt = cepgen::Event();
    //... define event attributes

    auto browser = cepgen::utils::EventBrowser();
    const double variable_value = browser.get(evt, "variable");

    return 0;
}
```

where the `variable` string can follow one of the three following conventions:

- `var` for event-level information (e.g. diffractive outgoing proton state multiplicity)
- `var(role)` for the retrieval of a single particle with a given role.

   These particle roles may be one of the followings:
    - `ib1` and `ib2` (resp. `ob1` and `ob2`) for the incoming (resp. outgoing) beam kinematics,
    - `pa1` and `pa2` for the parton/initiator particle emitted from the first/second incoming beam particle respectively,
    - `cs` for the two-parton/initators system, and
    - `int` for any intermediate $s$-channel particle exchange (depending on the process),

    as documented in
    ```{doxygenvariable} cepgen::utils::EventBrowser::role_str_
    ```

- `var(id)` for the retrieval of a single particle with a given integer identifier
- `var(role1,role2)` or `var(id1,id2)` for the multi-particles kinematics correlation.

The following momentum-based variables `var` are currently handled, either for single particle or combined multi-particles systems:

```{doxygenvariable} cepgen::utils::EventBrowser::m_mom_str_
```

In addition, particles momenta's correlations can be accessed through the following keywords:

```{doxygenvariable} cepgen::utils::EventBrowser::m_two_mom_str_
```
