---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: Unstructured LES_DNS of a Turbulent Boundary Layer over a Gaussian Bump
subtitle: ''
summary: 'Introduce the LES<sub>DNS</sub> simulation framework for developing turbulence models. The proposed framework is then applied onto the Gaussian speed-bump problem and compared to other simulation results, including DNS and WMLES.'
authors:
- James R. Wright
- Riccardo Balin
- Kenneth E. Jansen
- John A. Evans
tags: ["PHASTA", "Gaussian Speed-Bump", "LES_DNS"]
date: 2021-01-20
lastmod: 2021-01-22T13:49:54-07:00
featured: false
draft: false

projects: []
publishDate: '2021-01-22T20:56:56.706386Z'
publication_types:
- '1'
abstract: Wall-resolved large eddy simulation (WRLES) is often thought to have mesh
  resolution requirements that grow at the same asymptotic rate as direct numerical
  simulation (DNS), but WRLES actually does grow slower than DNS, even if the near-wall
  region is resolved to DNS resolution. In this paper, a new simulation concept dubbed
  LES<sub>DNS</sub> is undertaken, where the near-wall and inflow resolution of a WRLES is taken
  down to DNS levels. The primary advantage of this approach is that it removes the
  need for sub-grid models in the highly-anisotropic near-wall region, where they
  are the least mature. When combined with the use of unstructured grids that coarsen
  in all three directions outside of this near-wall region, significantly higher Reynolds
  numbers may be simulated for wall-bounded flows relative to traditional DNS. In
  this work, LES<sub>DNS</sub> is applied to the zero pressure gradient flat plate boundary
  layer and Gaussian bump problems. The new concept was found to be effective for
  both cases tested. In particular, LES<sub>DNS</sub> of the Gaussian bump was able to closely
  match DNS results at a fraction of the cost despite the complex physics involved
  with the problem. It is found that the new concept is effectively able to provide
  the sub-grid stress model with a \"perfect wall model\". Because of this \"perfect
  wall model\", deficiencies in the standard dynamic sub-grid stress model were identified
  in the outer region of the boundary layer in the Gaussian bump case.
publication: '*AIAA Scitech 2021 Forum*'
doi: 10/ghszgc
---

