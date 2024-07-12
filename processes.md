# Processes list

Natively, CepGen provides a collection of predefined, historical processes that can be extended or modified by the user.
This page lists and describes them.

As derivatives of a base {cepgen}`cepgen::proc::Process` object, a few common members and steering parameters are characterising all processes.


For instance, in [the description of all process parameters](/raw-modules.md#proc), one may notice the following parameters are systematically steered from the user:

- `alphaEM` ({cepgen}`cepgen::ParametersList`): one of the interpolation modules for the \$Q\$-dependent electromagnetic coupling evolution (see [the list](/raw-modules.md#alphaem)) ;
- `alphaS` ({cepgen}`cepgen::ParametersList`): one of the interpolation modules for the \$Q\$-dependent strong coupling evolution (see [the list](/raw-modules.md#alphas)) ;
- `hasEvent` (`bool`): does the process also define an event content that can be generated?
- `kinematics` ({cepgen}`cepgen::ParametersList`): the definition of the phase space to be integrated and for which events will be generated. It is defined in the subsection below ;
- `randomGenerator` ({cepgen}`cepgen::ParametersList`): the modelling for a (potential) random number generator to be used by the process.

````{note}
List of user-steerable parameters for the **definition of event kinematics**
```{toggle}

The `kinematics` parameters block contains the usual parameters to be steered:

- `formFactors` (2-component vector of {cepgen}`cepgen::ParametersList`, for positive-z and negative-z beams respectively): one of the electromagnetic/DIS beam form factors modellings (see [the list](/raw-modules.md#formfac))
- `structureFunctions` ({cepgen}`cepgen::ParametersList`): one of the inelastic beam structure functions modellings (see [the list](/raw-modules.md#strfun))
- `mode` (integer): kinematic regime to generate and the size of the phase space to perform the integration. It can take the following values:
    - {cepgen}`ProcessMode.ElasticElastic <python::Config::ProcessMode_cff::ProcessMode::ElasticElastic>`: elastic emission of photons from the incoming protons (default value if unspecified),
    - {cepgen}`ProcessMode.ElasticInelastic <python::Config::ProcessMode_cff::ProcessMode::ElasticInelastic>` and {cepgen}`ProcessMode.InelasticElastic <python::Config::ProcessMode_cff::ProcessMode::InelasticElastic>`: elastic scattering of one photon and an inelastic/semi-exclusive emission of the other photon, resulting in the excitation/fragmentation of the outgoing proton state,
    - {cepgen}`ProcessMode.InelasticInelastic <python::Config::ProcessMode_cff::ProcessMode::InelasticInelastic>`: both protons fragmented in the final state.

Kinematic cuts are characterised by three parameters collections to be steered:

- `central` ({cepgen}`cepgen::ParametersList`) for central particles cuts
- `initial` ({cepgen}`cepgen::ParametersList`) for initial partons/beam particles cuts
- `remnants` ({cepgen}`cepgen::ParametersList`) for beam particles remnants (after the hard interaction)

Additionally, a few parameters are steered to define the beams and their kinematics:

- `pdgIds` (`vector<int>`) for the specification of positive-z and negative-z beam PDG, or HI identifiers. For historical/backward-compatibility reasons, one may also use the following parameters:
    - `beam1id` and `beam2id` for the PDG identifier of positive-z and negative-z (resp.) incoming beams
    - `beam1A/beam1Z`, and `beam2A/beam2Z` may be used for the atomic/mass numbers of HI beams
- `energies` (`vector<double>`) for the specification of positive-z and negative-z beam energies (in GeV)
- `pz` (`vector<double>`) for the specification of positive-z and negative-z longitudinal beam momenta (in GeV/c)

In the case of symmetric beam particles, the `sqrtS` (`double`) parameters may also be used to directly specify the two-beam centre-of-mass energy (in GeV).
```
````

## LPAIR’s $\ggll$

The $pp \rightarrow p^{(\ast)}(\ggll)p^{(\ast)}$ process was first described and implemented in the early 1990s as a Fortran code: `LPAIR` {cite}`Baranov:1991yq`.
In CepGen, it can be reached through the [`lpair`](/raw-modules.md#proclpair) process.
It allows to compute the cross-section and generate events for the $\ggll$ process for ee, ep, and pp collisions.

A phenomenological review of both this process and its first implementation in `LPAIR` may be found in {cite}`Vermaseren:1982cz`.

The object is implemented as {cepgen}`LPAIR`, and a list of process-specific steering parameters can be found [here](/raw-modules.md#proclpair).

## Factorised processes

This page describes the different techniques used to simplify the computation of a central exclusive process through the factorisation of its total matrix element into sub-blocks:

- a parton-from-beam particle flux, encompassing all kinematic features of a "physical" emission, possibly leaving the beam particle on-shell (i.e. surviving the emission), or in a diffractive state (possibly dissociating into a hadronic jet) ;
- a parton-parton matrix element, e.g. a $\gamma\gamma\to X$ process, much easier to compute than the full beam-beam interaction $S$-matrix.

CepGen gives the user a relative freedom in its implementation of factorised, two-parton level processes.
As of version 1.2.0, two main parton emission types (and central matrix element definitions) are indeed handled:

- a standard, collinear parton emission, with a $x$ (inelasticity) and $Q^2$ squared momentum transfer dependence, implying a full momentum transfer along the beam direction (conventionally, the $z$-axis) ; this technique is used in many other Monte Carlo generators for the simulation of CEPs ;
- a parton $\kt$-dependent emission, also involving a dependence in the transverse parton virtuality, $\kt$, involving a broader definition of the phase space (i.e. more time-consuming, but more precise).

To be considered as a valid factorised process implementation in CepGen, the user class is required to derivate from a {cepgen}`cepgen::proc::FactorisedProcess` base class, with a few required user-overriden methods.
See [this page](processes-devel.md#c-interface) for a detailed description of these methods.

### Collinear parton emission

```{warning} Under construction
```

### $\kt$-factorised processes

The $\kt$ factorisation, described in detail in {cite}`daSilveira:2014jla` through the $\ggff$ process listed above, allows to provide a "physical" modelling of this $\kt$ dependence, commonly introduced as a simple gaussian smearing, e.g. in Pythia 8 (see the [`BeamRemnants:primordialKT`](https://pythia.org/latest-manual/BeamRemnants.html) flag).

For instance, a $pp\to p^{(\ast)}(\ggx)p^{(\ast)}$ matrix element can be factorised through the following formalism:

```{math}
\mathrm d\sigma = \int \frac{\mathrm d^2{\mathbf q_{\mathrm T}^2}_1}{\pi {\mathbf q_{\mathrm T}^2}_1}
                    {\cal F}_{\gamma/p}^{\rm el/inel}(x_1,{\mathbf q_{\mathrm T}^2}_1)
                    \int \frac{\mathrm d^2{\mathbf q_{\mathrm T}^2}_2}{\pi {\mathbf q_{\mathrm T}^2}_2}
                    {\cal F}_{\gamma/p}^{\rm el/inel}(x_2,{\mathbf q_{\mathrm T}^2}_2) ~ \mathrm d\sigma^\ast,
```

where $\mathcal F_{\gamma/p}^{\rm el/inel}(x_i,\vecqt_i)$ are unintegrated parton densities, and $\mathrm d\sigma^\ast$ the hard process factorised out of the total matrix element.

Elastic unintegrated photon densities are expressed as functions of the proton electric and magnetic form factors $G_E$ and $G_M$:

```{math}
\mathcal F_{\gamma/p}^{\rm el}(\xi,\vecqt^2) = \frac{\alpha}{\pi}\left[(1-\xi)\left(\frac{\vecqt^2}{\vecqt^2+\xi^2 m_p^2}\right)^2 F_E(Q^2)+\frac{\xi^2}{4}\left(\frac{\vecqt^2}{\vecqt^2+\xi^2 m_p^2}\right) F_M(Q^2)\right].
```

The inelastic contribution further requires both the diffractive state four-momentum norm $M_X$ and a [proton structure functions parameterisation](/structure-functions.md) as an input:

```{math}
\mathcal F_{\gamma/p}^{\rm inel}(\xi,\vecqt^2) = \frac{\alpha}{\pi}\Bigg[(1-\xi)\left(\frac{\vecqt^2}{\vecqt^2+\xi(M_X^2-m_p^2)+\xi^2 m_p^2}\right)^2\frac{F_2(\xbj,Q^2)}{Q^2+M_X^2-m_p^2}+{}\\
  {}+\frac{\xi^2}{4}\frac{1}{\xbj^2} \left(\frac{\vecqt^2}{\vecqt^2+\xi(M_X^2-m_p^2)+\xi^2 m_p^2}\right) \frac{2\xbj F_1(\xbj,Q^2)}{Q^2+M_X^2-m_p^2}\Bigg],
```

with $\xbj = {Q^2}/({Q^2+M_X^2-m_p^2})$ the Bjorken scaling variable.

### Implementations

#### $\ggff$ process

The photon transverse momentum-dependant description of this process was previously developed in `PPtoLL`, and described in {cite}`daSilveira:2014jla`.
Since it now also supports the quark-antiquark production (thus all charged fermions), it is defined as the {cepgen}`PPtoFF` process in CepGen.

##### Process-specific options

- `method` (integer): switch between the two matrix element definitions:
    - `0`: on-shell amplitude,
    - `1`: off-shell amplitude.
- `pair` (integer/PDG): PDG identifier of the fermion pair to be produced in the final state. It can take the following values:
    - `PDG.electron := 11`: $e^+e^-$ pair production
    - `PDG.muon := 13`: $\mu^+\mu^-$ pair production
    - `PDG.tau := 15`: $\tau^+\tau^-$ pair production
    - `PDG.down`, `PDG.up`, `PDG.strange`, `PDG.charm`, `PDG.bottom`, `PDG.top` (or equivalently `1-6`): quark pair production.

##### Full object reference

```{doxygenclass} PPtoFF
:outline:
```

#### $\ggww$ process

The two-photon production of gauge boson pairs process, i.e. $pp \rightarrow p^{(\ast)}(\ggww)p^{(\ast)}$, is implemented through the {cepgen}`PPtoWW` process object, featuring the on-shell and off-shell matrix elements reviewed in {cite}`Luszczak:2018ntp`.

##### Process-specific options

- `method` (integer): switch between the two matrix element definitions:
    - `0`: on-shell amplitude {cite}`Denner:1995jv`,
    - `1`: off-shell amplitude {cite}`Nachtmann:2005en,Nachtmann:2005ep`.
- `polarisationStates` (int): switch between all combinations of polarisation states to be included in the matrix element. It can take the following values:
    - `0`: all contributions,
    - `1`: longitudinal-longitudinal,
    - `2`: longitudinal-transverse,
    - `3`: transverse-longitudinal,
    - `4`: transverse-transverse.

##### Full object reference

```{doxygenclass} PPtoWW
:outline:
