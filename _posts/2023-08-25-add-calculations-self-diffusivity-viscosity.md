---
title: Add Calculations for Self-Diffusivity and Viscosity (GSoC 2023 Final Submission)
date: 2023-08-25 00:30:00 -0700
categories: [Google Summer of Code, GSoC Updates]
tags: [google summer of code, gsoc, google summer of code 2023, gsoc 2023, computer science, cs, software engineering, software development, open source, python, numpy, scipy, computational research, molecular dynamics, mdanalysis, computational chemistry, biophysics, bioinformatics, biomolecular research, materials research, chemical engineering, physics, mathematics, math, mdakit, mdakits, velocity autocorrelation function, vacf, self-diffusivity, viscosity, helfand, einstein, einstein-helfand, green-kubo]     # TAG names should always be lowercase
math: true
---

**Student**: Xu Hong Chen

**Mentors**: Hugo MacDermott-Opeskin ([@hmacdope](https://github.com/hmacdope)), Orion Cohen ([@orionarcher](https://github.com/orionarcher))

**Organization**: [MDAnalysis](https://www.mdanalysis.org/)

Transport property analyses are among [the most requested](https://github.com/MDAnalysis/mdanalysis/issues/2187) features in the [MDAnalysis project](https://www.mdanalysis.org/). Biomolecular researchers and chemical engineers routinely use these measurable values describing the transfer of mass, momentum, heat, and charge in their regular work.[^1]<sup>,</sup> [^2] [My Google Summer of Code (GSoC) project](https://summerofcode.withgoogle.com/programs/2023/projects/4vt9npUg) aimed to add methods for calculating self-diffusivity and viscosity, the two most common transport properties in the literature,[^1] to MDAnalysis with a focus on user-friendliness and test-driven development. Initially, I proposed to:

- Implement a class to calculate self-diffusivity via the Green-Kubo method
- Write a utility function to calculate shear viscosity via the Einstein method
- Create a utility function to calculate shear viscosity via the Green-Kubo method
- Test and document the new features to ensure they are reliable and easy to use

After some discussion that began with Discord user jenclark's suggestion to try the Helfand method for viscosity to better integrate the feature wtih MDAnalysis, we shifted our priorities away from the utility functions to focus on a new Einstein-Helfand viscosity class. That extra research and trial and error were well worth the result - better software with a more simple and robust API.

## My Summer in Python and NumPy Arrays

My summer culminated in the creation of [Transport Analysis](https://github.com/MDAnalysis/transport-analysis), a standalone Python package to compute and analyze transport properties powered by MDAnalysis.

Transport Analysis GitHub: https://github.com/MDAnalysis/transport-analysis

### Building a FAIR-compliant Python Package from the MDAKit Cookiecutter Template

[Generating my package](https://github.com/MDAnalysis/transport-analysis/commit/1814818e6922cd7351ff1aa265e2f3f140b4ba0a) from the MDAKit cookiecutter template streamlined package creation, ensuring that Transport Analysis was set up through a reliable, stringently reviewed process. The included CI/CD pipeline needed some adjusting on GitHub, but it facilitated writing maintainable software from the very beginning. [Pull request (PR) #3](https://github.com/MDAnalysis/transport-analysis/pull/3) added the new code formatter *Black* to the CI, while [PR #22](https://github.com/MDAnalysis/transport-analysis/pull/22) flattened the package's structure to simplify its organization of modules.

### Velocity Autocorrelation Functions (VACFs) and Green-Kubo Self-Diffusivity

[PR #1](https://github.com/MDAnalysis/transport-analysis/pull/1) built two implementations of the VACF calculation: a simple "windowed" algorithm and a fast FFT algorithm leveraging `tidynamics`. All the computations involved are vectorized, but the FFT algorithm speeds the calculations up even further. I designed a simple test system from a unit velocity trajectory and some very simple calculations on pen and paper, then gradually wrote more and more tests until my code was tested with the same rigour as the [MDAnalysis MSD module](https://docs.mdanalysis.org/stable/documentation_pages/analysis/msd.html) on top of reaching 100% code coverage.

[PR #23](https://github.com/MDAnalysis/transport-analysis/pull/23) introduced a plotting method for VACFs via `Matplotlib`'s [object-oriented API](https://matplotlib.org/stable/api/index.html), saving users from having to their own plotting code. [PR #24](https://github.com/MDAnalysis/transport-analysis/pull/24) then makes it easy to calculate the Green-Kubo self-diffusivity with the methods `self_diffusivity_gk` and `self_diffusivity_gk_odd`. It also added a new plotting method for the running integral.

### Einstein-Helfand Viscosity

[PR #25](https://github.com/MDAnalysis/transport-analysis/pull/25) was challenging because it was not part of the initial plan and the methodology and practices were less prominent in the literature. The calculation itself was also more difficult to work with because it a lot more terms than self-diffusivity, but I enjoyed the process of learning and developing this class. Like the VACF implementation, it takes full advantage of NumPy arrays and the [MDAnalysis AnalysisBase API](https://docs.mdanalysis.org/stable/documentation_pages/analysis/base.html).

[PR #29](https://github.com/MDAnalysis/transport-analysis/pull/29), which adds plotting functionalities for the Einstein-Helfand viscosity class, is essentially complete but has yet to be merged because we are still considering whether it would be better to leave the plot in standard MDAnalysis units or convert them to SI units. While standard MDAnalysis units are friendly for VACFs and self-diffusivity, that is not the case for viscosity due to the number of terms involved in the calculation. It is the only unmerged PR in the project at the time of writing.

Because we prioritized the new, more user-friendly Einstein-Helfand viscosity class, which turned out to be more time consuming thanks to the research and complex implementation, I did not get to write the Green-Kubo viscosity utility function within the GSoC timeline. Nonetheless, I am happy to have adapted my original plan to our community discussions and delivered a more useful project. I plan to implement the utility function after GSoC, though I suspect most users will much prefer the Einstein-Helfand viscosity class.

### Reproduction of the Literature

To go beyond testing against simple "toy" systems and test data, I began a reproduction of established transport property calculations in the literature to validate my implementations. I successfully calculated a self-diffusivity of $2.47 \times 10^-9$ $m^2 / s$ using my Green-Kubo self-diffusivity implementation from a $20$ $ps$ simulation of the SPC/E water model at $298$ $K$, which agrees well with the results in the literature.[^3]<sup>,</sup> [^4] A full writeup of this reproduction can be found in my last blog post, [*Reproduction and Beyond GSoC (GSoC '23, 4)*](https://xhgchen.github.io/posts/reproduction-and-beyond-gsoc/). Some of the key plots are provided below.

![VACF Plot](/assets/img/2023-08-22/vacf_plot.PNG){: width="1280" height="960" }

### Running Integral Plot

![Running Integral Plot](/assets/img/2023-08-22/running_integral_plot.PNG){: width="1280" height="960" }

## Using Transport Analysis

### Installation

Transport Analysis is nearing release, and it can be cloned and installed from GitHub in a virtual environment. Here is a simple example using `mamba`.

First, we clone Transport Analysis:

```console
git clone https://github.com/MDAnalysis/transport-analysis.git
```

Then we can go ahead and install the package with `mamba` and `pip`:

```console
cd transport-analysis

mamba create -n ta-package

mamba env update --name ta-package --file devtools/conda-envs/test_env.yaml

mamba activate ta-package

pip install -e .
```

Once Transport Analysis is officially released, this process will be even smoother.

### A Quick VACF Example

This example calculates a VACF from a very small set of AMBER test data with velocities. In Python, execute:

```python
# imports
import MDAnalysis as mda
from transport_analysis.velocityautocorr import VelocityAutocorr

# test data for this example
from MDAnalysis.tests.datafiles import PRM_NCBOX, TRJ_NCBOX
```

We will calculate the VACF of an atom group of all water atoms in residues 1-5. To select these atoms:

```python
u = mda.Universe(PRM_NCBOX, TRJ_NCBOX)
ag = u.select_atoms("resname WAT and resid 1-5")
```

We can run the calculation using any variable of choice such as `wat_vacf` and access our results using `wat_vacf.results.timeseries`:

```python
wat_vacf = VelocityAutocorr(ag).run()
wat_vacf.results.timeseries
array([275.62075467, -18.42008255, -23.94383428,  41.41415381,
     -2.3164344 , -35.66393559, -22.66874897,  -3.97575003,
      6.57888933,  -5.29065096])
```

Notice that this example data is insufficient to provide a well-defined VACF. When working with real data, ensure that the velocities are written frequently enough to obtain a VACF suitable for your needs.

A growing collection of Jupyter notebook tutorials/examples can be found in the [tutorials directory](https://github.com/MDAnalysis/transport-analysis/tree/main/docs/tutorials) in the repository.

## Lessons Learned

Here are a few highlights from my series of blog posts:

- Always check if the dependencies used are up-to-date and/or the expected version
- Look online to see if there are any well-maintained, trustworthy packages as reference for the feature to be implemented (e.g. the MSD module)
- Ask a lot of questions - many of the mentors who were not assigned to my project helped
- Following the GitHub discussions that more experienced developers have can be a fantastic learning opportunity
- Having a call with someone more experienced, like Hugo, when starting something completely new can really streamline the learning process
- Approaching your software with a real use case from the perspective of a user is a wonderful way to find new areas for improvement

## The Future

The beauty of open source is the potential for continued improvement and engagement with the community. I will continue working on Transport Analysis after GSoC, and both my mentors intend to make their own contributions to the package. We are always open to new ideas, GitHub issues, bug reports, and contributions. The following is simply a list of a few of the many possibilities for making improving this community resource:
- Implement a utility function to calculate viscosity via the Green-Kubo method (planned)
- Reproduce literature results using the implemented Einstein-Helfand viscosity class (will continue looking into in consultation with MDAnalysis community)
- Implement additional transport property analyses (ionic and thermal conductivity, etc) and tools (utility functions, plotting functions, etc)
- Continue adding example Jupyter Notebooks
- Continue improving the documentation

## References

[^1]: Maginn, E. J.; Messerly, R. A.; Carlson, D. J.; Roe, D. R.; Elliot, J. R. Best Practices for Computing Transport Properties 1. Self-Diffusivity and Viscosity from Equilibrium Molecular Dynamics [Article v1.0]. *Living J. Comput. Mol. Sci.* **2018**, *1* (1), 6324–6324. <https://doi.org/10.33011/livecoms.1.1.6324>.

[^2]: Wang, J.; Hou, T. Application of Molecular Dynamics Simulations in Molecular Property Prediction II: Diffusion Coefficient. *J. Comput. Chem.* **2011**, *32* (16), 3505–3519. <https://doi.org/10.1002/jcc.21939>.

[^3]: Kumar, S.; Sarkar, S.; Bagchi, B. Anomalous Viscoelastic Response of Water-Dimethyl Sulfoxide Solution and a Molecular Explanation of Non-Monotonic Composition Dependence of Viscosity. *J. Chem. Phys.* **2019**, *151* (19), 194505. <https://doi.org/10.1063/1.5126381>.

[^4]: Mark, P.; Nilsson, L. Structure and Dynamics of the TIP3P, SPC, and SPC/E Water Models at 298 K. *J. Phys. Chem. A* **2001**, *105* (43), 9954–9960. <https://doi.org/10.1021/jp003020w>.