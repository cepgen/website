```{title} LPAIR's two-photon production of fermion pair
```

# LPAIR’s $\ggll$

The $pp \rightarrow p^{(\ast)}(\ggll)p^{(\ast)}$ process as previously implemented in the early 1990s in `LPAIR` {cite}`Baranov:1991yq` can be reached through the `lpair` process in CepGen.
More generally, this implementation was designed to compute the cross-section and to generate events for the $\ggll$ process for ee, ep, and pp collisions.

A phenomenological review of both this process and its first implementation in `LPAIR` may be found in {cite}`Vermaseren:1982cz`.

## Process-specific options

- `mode` (integer):  kinematic regime to generate and the size of the phase space to perform the integration. It can take the following values:
    - `ProcessMode.ElasticElastic := 1`: elastic emission of photons from the incoming protons (default value if unspecified),
    - `ProcessMode.ElasticInelastic := 2 / ProcessMode.InelasticElastic := 3`: elastic scattering of one photon and an inelastic/semi-exclusive emission of the other photon, resulting in the excitation/fragmentation of the outgoing proton state,
    - `ProcessMode.InelasticInelastic := 4`: both protons fragmented in the final state.
- `pair` (integer/PDG): PDG identifier of the lepton to be produced in the final state. It can hence take the following values:
    - `PDG.electron := 11` for the $e^+e^-$ pair production,
    - `PDG.muon := 13` for the $\mu^+\mu^-$ pair production, and
    - `PDG.tau := 15` for the $\tau^+\tau^-$ pair production.

## Full object reference

```{doxygenclass} LPAIR
:private-members:
:undoc-members:
```
