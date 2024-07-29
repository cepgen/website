# MadGraph5_aMC@NLO interface

```{versionadded} 1.2.0
```

A CepGen interface to the [`MadGraph5_aMC@NLO`](http://madgraph.phys.ucl.ac.be/) (here abbreviated as `MG5_aMC`) matrix element framework described in {cite}`Alwall:2014hca` was developed to generalise the two-photon (and central exclusive) processes production to advanced final states and interactions.

It relies on the `standalone_cpp` output mode [introduced in version 2.9.10](https://github.com/mg5amcnlo/mg5amcnlo/commit/b2b2118fc6dacdb8ae0c16903deafcf4baf87dcd) to be used as an extension to Pythia 8.

The CepGen addon module, `CepGenMadGraph` requires a valid installation of `MG5_aMC` in the search path (`MADGRAPH_DIR` environment variable) at configuration/build time.
Once loaded in the runtime environment manager, the `mg5_aMC` process type is handled with its standard parameters (see [this summary page](/raw-modules.md#procmg5_aMC) for a detailed description of these parameters).

For instance, the generation of a $\ggll$ process can be steered as:

```python
process = cepgen.Module('mg5_aMC',
	processParameters = cepgen.Parameters(
		model = 'sm-full',
		process = 'a a > mu+ mu-',
		mode = cepgen.ProcessMode.ElasticElastic
	),
	inKinematics = cepgen.Parameters(
		pdgIds = (PDG.proton, PDG.proton),
		sqrtS = 13.6e3  # in GeV
	),
	outKinematics = cepgen.Parameters(
		qt = (0., 10.),
		pt = (25.,)
	)
)
```

As seen above, the `process.processParameters.process` and `process.processParameters.model` strings are handling the `MG5_aMC` process to be generated, and the physics model considered in the operation.

The addon will directly call the `MG5_aMC` executable to produce a CepGen-compatible runtime library through the specialisation of a {cepgen}`cepgen::MadGraphProcess` interface object.
Therefore, a MadGraph-generated process shared library can be shipped by processes developers and used with any compatible version of CepGen to compute a cross section or generate events according to kinematic constraints.

As for any factorised process (as `pptoff` and `pptoww`), both $\kt$-dependent, or collinear fluxes can be convoluted to the central matrix element to obtain the total beam-level cross section and event definition.
