# Distutils is dead, long live ...

This is a discussion of Python packaging as of fall 2021.

In part it is an introduction to what is changing, and a set of pointers for those who are interested to plan for these changes.

To be clear, this discussion is about the official Python packaging options,
designed to install source or binary packages from the [Python Package
Index](https://pypi.org) or similar. We are not discussing other packaging
systems, where the two major options are:

* [Conda](https://docs.conda.io) or (equivalently)
  [Mamba](https://mamba.readthedocs.io), installing from compatible package
  repositories such as [Anaconda.org](https://anaconda.org) or
  [Conda-forge](https://conda-forge.org) — e.g. `mamba install numpy`.
* Your Linux distribution : e.g. `apt install python-numpy`).

## What is changing

There have been various important developments in the official Python packaging
system over the last 10 years. 

The basic theme of these development is three-fold:

* A move away from the classic `setup.py` executable file to define tasks and
  metadata for packaging.
* A move towards providing [binary wheels](

### Where Python packaging is going

`distutils` & co (`setuptools`, `numpy.distutils`) are still used by the vast majority of packages. However projects with very complex builds have moved away or never used it in the first place (e.g., cuDF/RAPIDS uses `scikit-build`, PyTorch uses CMake with a custom `setup.py` that invokes it, TensorFlow uses Bazel).

What has been changing in the Python packaging ecosystem itself is a move away from "assume a package uses setuptools" to a standards-based approach with hooks, so any package installer (like `pip`) can work with any build backend. Some of the most important PEPs and projects:
- [PEP 517: A build-system independent format for source trees](https://www.python.org/dev/peps/pep-0517/)
- [PEP 518: Specifying Minimum Build System Requirements for Python Projects](https://www.python.org/dev/peps/pep-0518/) 
- [PEP 621: Storing project metadata in pyproject.toml](https://www.python.org/dev/peps/pep-0621/)

The tl;dr of all those things is: you can specify all your project metadata and build-time and runtime dependencies in `pyproject.toml` and you can add a hook there that calls a Python function in a package you specify. When you do `pip install .`, pip finds the hook and invokes it - no `setup.py` needed.

There are also a number of small projects that have as purpose to be a standalone tool fulfilling one aspect of the build/package/install lifecycle, adhering to the PEPs linked above and not requiring `setuptools`: [build](https://github.com/pypa/build), [installer](https://github.com/pradyunsg/installer), [packaging](https://github.com/pypa/packaging).


### Status of building SciPy with Meson

I have a [branch named `meson` in my fork](https://github.com/rgommers/scipy/tree/meson) which makes a start, it builds a couple of modules, and uses C, C++, Cython and Fortran code. Look at the `meson.build` files there to get a sense of what things look like.

To actually use that branch:
- create a new conda env or virtualenv with all build dependencies (e.g., `conda env create -f environment.yml`)
- Install Meson into it from this PR: https://github.com/mesonbuild/meson/pull/8706. That PR is a (working) WIP PR to add 1st class Cython support to Meson (see https://github.com/mesonbuild/meson/issues/8693 for context).
- Run `meson setup build` followed by `ninja -C build`. This should successfully build the project. OR: run [mesondev.sh](https://github.com/rgommers/scipy/blob/meson/mesondev.sh) in the root of the repo to do both those steps plus install and run tests (e.g., it replaces our `runtests.py`)

The next steps / main blockers are:

1. We need to implement good BLAS/LAPACK support for Meson, so one can use that as a [dependency](https://mesonbuild.com/Dependencies.html#dependency-method). I've tried with conda + OpenBLAS, that seems to be straightforward via `pkg-config`. Complete support, e.g. Windows, MKL, is a fair bit of work. See https://github.com/mesonbuild/meson/issues/2835
2. That Cython support for Meson needs to be completed.
3. For codegen targets in combination with Cython, it looks like we need https://github.com/mesonbuild/meson/pull/8775/.

We can work around both (2) and (3) I believe, by extensive use of `custom_target()`, but it'll be super verbose and clunky, so don't really want to do that.

(2) and (3) are both being implemented by @dcbaker, so a big thank you to him.

We should probably use our own fork of Meson for a bit that is like an integration branch of upstream for 1-3. And then once we're completely happy, contribute the BLAS/LAPACK detection upstream.


## Can we start a long-term branch in this repo?

Right now the strategy I used in my fork is:
- add 17 WIP commits that remove each of the SciPy submodules in one go, one commit per submodule
- when working on adding Meson support to a new submodule, drop the relevant commit via an interactive rebase
- add the required `meson.build`, ensure things build cleanly and tests pass
- the build log is very clean, so the occasional build warnings can be more easily fixed in SciPy itself (try for: everything except warnings from vendored Fortran code and the deprecated NumPy API warnings)

This rebase strategy obviously doesn't work well when multiple people are collaborating on the same branch, if it's in the main repo (with a few people it does work on a fork). Also, CI runs on the main repo are not all that useful. So my preference would be to push it a little further in my own fork; once we have BLAS/LAPACK working we can add support for more submodules quickly. And then also make CI work on the fork first, so we can deal with Windows, building wheels, etc. Using PRs to my repo should work for that - I'd love some help though.

Once we have most submodules done and some working CI, then I think it's time to move to this repo.

The one thing we should probably do now to make life easier is to create a `scipy/meson` repo with an integration branch. Making people install code that lives in a WIP PR to Meson isn't ideal.
