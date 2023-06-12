---
title: A Fast Start with VACFs (GSoC '23, 1)
date: 2023-06-12 16:30:00 -0600
categories: [Google Summer of Code, GSoC Updates]
tags: [google summer of code, gsoc, google summer of code 2023, gsoc 2023, computer science, cs, python, numpy, scipy, computational research, molecular dynamics, mdanalysis, computational chemistry, biophysics, bioinformatics, biomolecular research, materials research, chemical engineering, physics, mathematics, math, mdakit, mdakits, velocity autocorrelation function, vacf, self-diffusivity, green-kubo]     # TAG names should always be lowercase
---

## Fruitful Beginnings

The past two weeks have been full of progress and unexpected challenges. Communication with my mentors and the MDAnalysis organization as a whole has gone well, and everyone has been responsive and helpful, even with their other commitments. I built [Transport Analysis](https://github.com/MDAnalysis/transport-analysis), a fresh, standalone Python package from the [MDAKit cookiecutter template](https://github.com/MDAnalysis/cookiecutter-mdakit), then finished implementing and testing the basic functionality for computing velocity autocorrelation functions (VACFs) from an `AtomGroup`. I also raised the appropriate issues for the next features and changes in the GitHub repository. Recently, I worked through the Read the Docs tutorial in preparation for writing the documentation.

## Developing an MDAKit from the Cookiecutter

Transport Analysis is a FAIR-compliant [MDAKit](https://mdakits.mdanalysis.org). Creating the package was the first roadblock because I kept running into the following issue during template generation:

```
Choose from 1, 2 [1]: 1
template_analysis_class [Class name to template (e.g. TransportAnalysis ). Press Enter to skip including analysis templates]: GreenKuboVACF
_dependency_source_keys [default]:
Error: Unable to decode to JSON.
```

It turns out this issue came from using an outdated version of cookiecutter. I resolved the issue by installing cookiecutter using `pip` instead of `conda` in my development environment. A while later, I realized I could have also fixed the issue by adding `conda-forge` to my channels before installing with `conda`.

## Test-Driven Development and VACFs

Hugo was right about Hofstadter's Law: it always takes longer than you expect to perform a task of high complexity. The silver lining is that it did not take much longer than expected. Even with a good plan ready going in, I had to slow down to write and test my code step by step. Becoming more patient and immersing myself into this steady progression helped me finish the bulk of the implementation in a timely manner. Each time I wrote new code, I would test it in Jupyter Lab to ensure it worked as intended, then test it again by writing tests for the code. Most of the time, I test far more than I code. I believe this is critical not only for calculations of this nature, but also to ensure reliable and stable code.

It was really nice to have the [MSD module](https://docs.mdanalysis.org/stable/documentation_pages/analysis/msd.html) as a reference for this part of the package. The greatest challenge here was actually trying to figure out the results to expect and test for with the VACF calculation. I downsized the problem to a unit velocity trajectory and wrote some very simple calculations with pen and paper before writing the first set of tests for basic functionality. From there, I wrote more and more tests until the `VelocityAutocorr` class was tested with the same rigour as the MSD module on top of reaching 100% code coverage.

## Continuous Integration and Continuous Delivery (CI/CD)

One of the many benefits of building the package as an MDAKit is the included CI/CD pipeline. This practice was fairly easy to get into with the help of the template, but would have been less straightforward to add on my own. Thanks to Orion's suggestion to add a linter, I was able to add [Black](https://black.readthedocs.io/en/stable/) as a new check in the CI without much trouble. I read through the scripts for [GitHub Actions](https://github.com/features/actions), watched a video tutorial, and read through Black's documentation. One minor issue that came from this is that because Black is a formatter, not a linter, it is extremely strict with its checks. All of the runs through the CI/CD will fail until the VACF pull request (PR) is merged. Hopefully that will be soon!

## Lessons Learned

These past few weeks were rife with challenges and victories. Here are some of my key takeaways:

- Always check if the dependencies used are up-to-date and/or the expected version
- Test-driven development, and programming in general, is a gradual, step-by-step process
- Look online to see if there are any well-maintained, trustworthy packages as reference for the feature to be implemented (e.g. the MSD module)
- Documentation and tutorial videos are great resources if approached with caution
- Ask a lot of questions - many of the mentors who were not assigned to my project helped! Thank you [@lilyminium](https://github.com/lilyminium) [@IAlibay](https://github.com/IAlibay) [@RMeli](https://github.com/RMeli) [@orbeckst](https://github.com/orbeckst) !!

## Next Steps

- Meet the rest of the MDAnalysis team, my fellow GSoC students, and the new Station1 students tomorrow!
- Start using GitHub Projects
- Prepare to write the VACF documentation once the VACF PR is merged
- Finish planning/start writing the viscosity calculation via the Helfand method
