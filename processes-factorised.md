# Factorised processes

This page describes the different techniques used to simplify the computation of a central exclusive process through the factorisation of its total matrix element into sub-blocks:

- a parton-from-beam particle flux, encompassing all kinematic features of a "physical" emission, possibly leaving the beam particle on-shell (i.e. surviving the emission), or in a diffractive state (possibly dissociating into a hadronic jet) ;
- a parton-parton matrix element, e.g. a $\gamma\gamma\to X$ process, much easier to compute than the full beam-beam interaction $S$-matrix.

CepGen attempts to give the user a relative freedom in its implementation of factorised, two-parton level processes.
As of version 1.2.0, two main parton emission types (and central matrix element definitions) are indeed handled:

- a standard, collinear parton emission, with a $x$ (inelasticity) and $Q^2$ squared momentum transfer dependence, implying a full momentum transfer along the beam direction (conventionally, the $z$-axis) ; this technique is used in many other Monte Carlo generators for the simulation of CEPs ;
- a parton $\kt$-dependent emission, also involving a dependence in the transverse parton virtuality, $\kt$, involving a broader definition of the phase space (i.e. more time-consuming, but more precise).

## Collinear parton emission

```{warning} Under construction
```

## $\kt$-factorised processes

The $\kt$ factorisation, described in detail in {cite}`daSilveira:2014jla` through the $\ggff$ process listed above, allows to provide a "physical" modelling of this $\kt$ dependence, commonly introduced as a simple gaussian smearing, e.g. in Pythia 8 (see the [`BeamRemnants:primordialKT`](https://pythia.org/manuals/pythia8311/BeamRemnants.html) flag).

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

## Implementations

#### Common options

- `mode` (integer): kinematic regime to generate and the size of the phase space to perform the integration. It can take the following values:
    - `ProcessMode.ElasticElastic := 1`: elastic emission of photons from the incoming protons (default value if unspecified),
    - `ProcessMode.ElasticInelastic := 2 / ProcessMode.InelasticElastic := 3`: elastic scattering of one photon and an inelastic/semi-exclusive emission of the other photon, resulting in the excitation/fragmentation of the outgoing proton state,
    - `ProcessMode.InelasticInelastic := 4`: both protons fragmented in the final state.

### $\ggff$ process

The photon transverse momentum-dependant description of this process was previously developed in `PPtoLL`, and described in {cite}`daSilveira:2014jla`.
Since it now also supports the quark-antiquark production (thus all charged fermions), it is defined as the {cepgen}`PPtoFF` process in CepGen.

#### Process-specific options

- `method` (integer): switch between the two matrix element definitions:
    - `0`: on-shell amplitude,
    - `1`: off-shell amplitude.
- `pair` (integer/PDG): PDG identifier of the fermion pair to be produced in the final state. It can take the following values:
    - `PDG.electron := 11`: $e^+e^-$ pair production
    - `PDG.muon := 13`: $\mu^+\mu^-$ pair production
    - `PDG.tau := 15`: $\tau^+\tau^-$ pair production
    - `PDG.down`, `PDG.up`, `PDG.strange`, `PDG.charm`, `PDG.bottom`, `PDG.top` (or equivalently `1-6`): quark pair production.

#### Full object reference

```{doxygenclass} PPtoFF
:outline:
```

### $\ggww$ process

The two-photon production of gauge boson pairs process, i.e. $pp \rightarrow p^{(\ast)}(\ggww)p^{(\ast)}$, is implemented through the {cepgen}`PPtoWW` process object, featuring the on-shell and off-shell matrix elements reviewed in {cite}`Luszczak:2018ntp`.

#### Process-specific options

- `method` (integer): switch between the two matrix element definitions:
    - `0`: on-shell amplitude {cite}`Denner:1995jv`,
    - `1`: off-shell amplitude {cite}`Nachtmann:2005en,Nachtmann:2005ep`.
- `polarisationStates` (int): switch between all combinations of polarisation states to be included in the matrix element. It can take the following values:
    - `0`: all contributions,
    - `1`: longitudinal-longitudinal,
    - `2`: longitudinal-transverse,
    - `3`: transverse-longitudinal,
    - `4`: transverse-transverse.

#### Full object reference

```{doxygenclass} PPtoWW
:outline:
```
