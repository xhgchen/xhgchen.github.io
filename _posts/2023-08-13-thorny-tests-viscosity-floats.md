---
title: Thorny Tests with Viscosity and Floats (GSoC '23, 3)
date: 2023-08-14 13:30:00 -0700
categories: [Google Summer of Code, GSoC Updates]
tags: [google summer of code, gsoc, google summer of code 2023, gsoc 2023, computer science, cs, software engineering, software development, open source, python, numpy, scipy, computational research, molecular dynamics, mdanalysis, computational chemistry, biophysics, bioinformatics, biomolecular research, materials research, chemical engineering, physics, mathematics, math, mdakit, mdakits, viscosity, helfand, einstein, einstein-helfand, green-kubo]     # TAG names should always be lowercase
math: true
---

The weeks since my last post have been some of the most frustrating and rewarding. I struggled with testing Einstein-Helfand viscosity, but being persistent in my debugging and proactive in meeting with my mentors paid off. I completed my implementation of Einstein-Helfand viscosity, created a basic set of tests with a more fully defined unit velocity trajectory, and merged 2 pull requests (PRs) in the core MDAnalysis repository. Over the course of this work, I learned to better deal with floating point operations and write code for deprecations. I also spoke with Hugo and Orion and we agreed to modify our plans to focus more on ensuring our current implementations of Green-Kubo self-diffusivity and Einstein-Helfand viscosity are reliable through reproduction of documented values/results in the literature. We will save the Green-Kubo utility function for after GSoC, as Helfand viscosity is far more user-friendly and the remaining weeks would be too tight of a deadline.

## Completing and Testing Einstein-Helfand Viscosity

As outlined in the last blog post, shear viscosity can be calculated using the Einstein-Helfand method by taking the slope of the following:

![Viscosity Formula Einstein-Helfand](/assets/img/2023-07-25/viscosityHelfand.PNG){: width="902.4" height="152.8" }

where $x$ is the position, $p$ is the momentum, $k_B$ is the Boltzmann constant, $T$ is the temperature, and $V$ is the volume.[^1]

One of the biggest concerns I had with the equation above was which units to work with. Luckily, meeting with [@hmacdope](https://github.com/hmacdope) and [@orionarcher](https://github.com/orionarcher) helped me understand that it is easiest to simply work with the MDAnalysis units. MDAnalysis has a great page on how it converts units to a set standard to ensure interoperability in its [User Guide](https://userguide.mdanalysis.org/stable/units.html). Another notably takeaway was [working with box volume in MDAnalysis](https://docs.mdanalysis.org/stable/documentation_pages/transformations/boxdimensions.html). Users can set and fetch the box volume at any given timestep, which is really handy in the viscosity calculation itself and extending the unit velocity trajectory to be usable for viscosity testing.

Like the velocity autocorrelation (VACF) and self-diffusivity implementations, the viscosity implementation needed to be tested against a simple, predictable, and minimalistic system. We started with a unit velocity trajectory and plan try reproducing some values from the literature with an OpenMM water system. For viscosity, I specified masses, positions, box volume, and an average temperature of 300 K in the unit velocity trajectory to make the calculation possible.

## Floating Point Issues

When I initially tried to test my `ViscosityHelfand` class with an equivalent calculation applied to the full unit velocity trajectory with for loops, I obtained values that looked almost the same but had larger differences with increasing steps in the unit velocity trajectory. While first testing with `assert_almost_equal`, the output I received was along the lines of:

```
AssertionError: 
Arrays are not almost equal to 4 decimals

Mismatched elements: 4999 / 5001 (100%)
Max absolute difference: 1.00146809e+15
Max relative difference: 6.56064432e-08
x: array([0.0000e+00, 1.8041e+15, 7.2142e+15, ..., 2.5027e+22, 2.5042e+22,
      2.5057e+22])
y: array([0.0000e+00, 1.8041e+15, 7.2142e+15, ..., 2.5027e+22, 2.5042e+22,
      2.5057e+22])
```

Though `assert_allclose` passed, I was somewhat concerned with the reported differences. I promptly met with [@hmacdope](https://github.com/hmacdope), who informed me that temporary variables can cause precision loss. The fewer temporary variables we use, in the context of large or complex floating point operations, the better. It is best to compress data into arrays if possible, as vectorization can improve accuracy and performance. I soon realized that my attempt to use a different method to calculate viscosity through for loops was a little more prone to errors than the `NumPy` approach in my actual `ViscosityHelfand` class. Once I edited my vectorized my tests using `NumPy` arrays, the values agreed better. It is important to mention that this testing was crucial since it allowed me to catch that I had incorrectly placed the final division step inside a for loop, making it repeat many more times than intended. Without these tests, this error would have been much harder to spot.

## Boltzmann Constant Typo Fix and Deprecation

While implementing Einstein-Helfand viscosity, I realized that the Boltzmann constant key in the `constants` dictionary in MDAnalysis was misspelled as `Boltzman_constant`. I raised [Issue #4213](https://github.com/MDAnalysis/mdanalysis/issues/4213) and submitted [PR #4214](https://github.com/MDAnalysis/mdanalysis/pull/4214) to fix this typo, but [@hmacdope](https://github.com/hmacdope) and I soon realized that this was [an API break](https://github.com/MDAnalysis/mdanalysis/issues/4229) in that anything who called this key with a typo outside of the MDAnalysis library would no longer work. Thanks to some good discussion on that GitHub issue, I had the chance to update my fix to allow both the old and new keys with a `DeprecationWarning`. [@IAlibay](https://github.com/IAlibay)'s suggestion to subclass `dict` and wrap `__getitem__` with the warning came in handy and I learned to work with deprecations for the first time in [PR #4230](https://github.com/MDAnalysis/mdanalysis/pull/4230), which turned out to be a lot of fun.

## Lessons Learned

- Vectorize whenever possible - the accuracy and performance improvements can be noticeable, especially the latter
- Following the GitHub discussions that more experienced developers have can be a fantastic learning opportunity
- Testing is a nice way to carefully review code from a new perspective
- Estimating how long a software project will take is a difficult task and it is better to underestimate and overdeliver than to be too ambitious with the timeline; however, it is okay to your adjust plans if the changes are reasonable

## Next Steps

I would like to note that I do intend to continue working on Transport Analysis and MDAnalysis after GSoC, so this "next steps" section is only reflective of the GSoC timeline and not of my greater long term goals.
- Reproduction of transport property values in the literature: start running OpenMM simulations on a minimalistic water system
- Implement plotting functions for the `ViscosityHelfand` class
- Upload example Jupyter Notebooks
- Improve the docs if time permits

## References

[^1]: Kirova, E. M.; Norman, G. E. Viscosity Calculations at Molecular Dynamics Simulations. *J. Phys. Conf. Ser.* **2015**, *653* (1), 012106. https://doi.org/10.1088/1742-6596/653/1/012106.
