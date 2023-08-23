---
title: Tricky Plots and Integrals (GSoC '23, 2)
date: 2023-07-25 19:30:00 -0600
categories: [Google Summer of Code, GSoC Updates]
tags: [google summer of code, gsoc, google summer of code 2023, gsoc 2023, computer science, cs, software engineering, software development, open source, python, numpy, scipy, computational research, molecular dynamics, mdanalysis, computational chemistry, biophysics, bioinformatics, biomolecular research, materials research, chemical engineering, physics, mathematics, math, mdakit, mdakits, velocity autocorrelation function, vacf, self-diffusivity, green-kubo, viscosity, helfand, einstein, einstein-helfand]     # TAG names should always be lowercase
math: true
---

Even with my 2 week Vancouver housing search getting in the way, the past period has been satisfying. I finished the bulk of the self-diffusivity implementation, added a plotting function for the velocity autocorrelation function (VACF) class, and started working on the Helfand viscosity implementation. Along the way, I flattened the package's organization and wrote documentation for the VACF class. As always, Hugo, Orion, and the rest of the MDAnalysis organization have been essential supports and voices of wisdom in this process. I am happy to have passed the Google Summer of Code Midterm Evaluation, and all of this progress makes me excited to see what [Transport Analysis](https://github.com/MDAnalysis/transport-analysis) will look like upon its first release.

I will also take this chance to note that a VACF expresses how much the velocity of a system changes over time. If a system stayed at constant velocity for the entire course of the simulation, it would have a stable velocity correlation and the VACF plot would be a horizontal line. The sharper the decay of the VACF, the sharper the changes in velocity in the system.

## Package Organization and VACF Docs

Package organization is an important consideration in the development process. [@orionarcher](https://github.com/orionarcher) brought up that having an analysis subfolder is redundant in our package because all the modules will be analysis type modules. It made more sense to move up all the files so that users will call `transport_analysis.module_name` instead of `transport_analysis.analysis.module_name`. I merged this update in [PR#22](https://github.com/MDAnalysis/transport-analysis/pull/22).

Using Orion's [SolvationAnalysis docs](https://solvation-analysis.readthedocs.io/en/latest/) as a guide, I was able to smoothly put together the [docs for Transport Analysis](https://transport-analysis.readthedocs.io/en/latest/). In particular, I wrote [a user-friendly page on the VACF calculation](https://transport-analysis.readthedocs.io/en/latest/api/velocityautocorr.html) and provided a simple example of its usage.

## Plotting VACFs

`Matplotlib`'s [object-oriented API](https://matplotlib.org/stable/api/index.html) makes it easy to plot data. To streamline the VACF calculation workflow, I wrote `plot_vacf()` using `Matplotlib` ([PR#23](https://github.com/MDAnalysis/transport-analysis/pull/23)), saving users from having to write the plotting code themselves. Testing this function was less straightforward. I considered using image comparison tests but ultimately decided they would make the repository more bloated and difficult to maintain. [mwaskom's Stack Overflow comment](https://stackoverflow.com/questions/27948126/how-can-i-write-unit-tests-against-code-that-uses-matplotlib) helped me figure out that validating the plot data, labels, and start, stop, step functionality was an effective solution for our package.

## Self-Diffusivity

The Green-Kubo self-diffusivity is calculated by integrating the VACF and dividing by the dimensionality. I implemented this calculation as the `VelocityAutocorr` class function `sd()` using `scipy.integrate.trapezoid`. I will soon be amending the function name to `sd_gk()` to reflect that it follows the Green-Kubo approach. After consulting [@hmacdope](https://github.com/hmacdope) and [@orionarcher](https://github.com/orionarcher), we decided to test the calculation against `scipy.integrate.simpson` and the [MDAnalysis MSD module](https://docs.mdanalysis.org/stable/documentation_pages/analysis/msd.html) to start. [@hmacdope](https://github.com/hmacdope) gave some really valuable feedback in [PR#24](https://github.com/MDAnalysis/transport-analysis/pull/24) about checking whether the analysis has been run and other considerations that I will be following up on shortly.

## Viscosity

The initial idea to write only utility functions was not as user-friendly as letting users calculate shear viscosity directly via the Einstein-Helfand method, so we adjusted our plans accordingly. I began writing a `ViscosityHelfand` class instead of an Einstein viscosity utility function ([PR#25](https://github.com/MDAnalysis/transport-analysis/pull/25)). Shear viscosity can be calculated using the Einstein-Helfand method by taking the slope of the following:

![Viscosity Formula Einstein-Helfand](/assets/img/2023-07-25/viscosityHelfand.PNG){: width="902.4" height="152.8" }

where $x$ is the position, $p$ is the momentum, $k_B$ is the Boltzmann constant, $T$ is the temperature, and $V$ is the volume.[^1] This information is all obtainable through MDAnalysis, so the Helfand viscosity calculation works well as an analysis subclassing `AnalysisBase`. The user will no longer need to provide any pressure tensors; they will only need to enter an `AtomGroup` and the average temperature of the simulation to run the calculation.

Since there is no suitable alternative for the Green-Kubo viscosity calculation, I will write a utility function for this as planned.

## Lessons Learned

- Start small with the tests - for `sd()`, there was no need to let my difficulties finding an analytical solution hinder my progress
- Thinking about and looking into whether something can be done better is a great way to come up with new ideas and make headway with writing tests
- Documentation is easy to write when you can find a reference to follow

## Next Steps

- Follow up on the feedback in [PR#24](https://github.com/MDAnalysis/transport-analysis/pull/24): use a cache variable to check whether the analysis has been run, organize the tests that use the fixture in a separate class and make the parameterization over the class, and add functionality to plot the running integral
- Continue writing tests for the `ViscosityHelfand` class ([PR#25](https://github.com/MDAnalysis/transport-analysis/pull/25))
- Install OpenMM and test the self-diffusivity implementation with real data

## References

[^1]: Kirova, E. M.; Norman, G. E. Viscosity Calculations at Molecular Dynamics Simulations. *J. Phys. Conf. Ser.* **2015**, *653* (1), 012106. https://doi.org/10.1088/1742-6596/653/1/012106.
