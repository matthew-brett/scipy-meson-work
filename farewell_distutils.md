# Distutils is dead, long live ...

This is a discussion of Python packaging as of fall 2021.

In part it is an introduction to what is changing, and a set of pointers for
those who are interested to plan for these changes.

## Not in this page

This discussion is about the official Python packaging options, designed to
install source or binary packages from the [Python Package
Index](https://pypi.org) or similar. We are not discussing other
packaging systems, where the two major options are:

* [Conda](https://docs.conda.io) or (equivalently)
  [Mamba](https://mamba.readthedocs.io), installing from compatible package
  repositories such as [Anaconda.org](https://anaconda.org) or
  [Conda-forge](https://conda-forge.org) — e.g. `mamba install numpy`.
* Your Linux distribution : e.g. `apt install python-numpy`).

## History

### Distutils

Imagine we start in a directory containing the code for `my_package`.

We will call this directory a *source tree*.  It contains all the source files
necessary for installing the package on any compatible system.  It could be a
checkout from version control, or the result of downloading a source code
archive, and unpacking.

For example, the Python Package Index will nearly always contain a `.tar.gz`
or `.zip` file corresponding to each Python package.  This is the *sdist*
(source distribution) file — or source code archive, as we have used the term
above. When unpacked, it forms a *source tree*.  For example, the [1.21.2
release of the Numpy package on PyPI](https://pypi.org/project/numpy/1.21.2/)
contains a file
[numpy-1.21.2.zip](https://files.pythonhosted.org/packages/3a/be/650f9c091ef71cb01d735775d554e068752d3ff63d7943b26316dc401749/numpy-1.21.2.zip).
When unpacked, this is a source tree for the Numpy 1.21.2 release.

In the beginning, there was:

```bash
python setup.py install
```

In this incarnation, Python has to execute the `setup.py` file in the current
directory, to get the various settings that apply to the package being
installed.

The `setup.py` file would typically have a line like the following:

```python
from distutils.core import setup
```

The `setup.py` file then would call the `setup` function with arguments
defining the files and other features of the package to install.

Distutils' job is relatively simple with a pure-Python package, where the
package contains only Python and data files, but no files in languages other
than Python, which need to be compiled.  In that case Distutils functions as a
simple build system, that works to discover a suitable compiler and assemble
relevant options to the compiler, before calling the compiler and creating
binary files, such as compiled Python extensions.

This quickly became unsatisfactory as a way of installing Python packages,
because it meant you had to download a source file archive or do a version
control checkout, and then run this command from the resulting directory.  If
the package had files for compilation, the user would need the relevant
compilers to be set up correctly on their computer, a particularly difficult
problem on Windows.

Therefore Distutils is, in part, a binary build system, specific to Python.

### Setuptools

Later came the [setuptools package](https://pypi.org/project/setuptools).
This had various useful — but sometimes controversial — extensions to
`distutils`.   Setuptools could work as drop in replacement for Distutils
(although it used parts of Distutils internally).  The `setup.py` function
would now have:

```python
from setuptools import setup
```

This import gives more options for the `setup` function, and various other
utilities for finding packages and so on.

In this use, Setuptools acts as an enhanced version of Distutils — *Setuptools
is Distutils+*.

Setuptools also made a first step at solving the problem of having to download
the code manually, by adding the `easy_install` command - as in `easy_install
my_package`.  This would search the Python Package Index (PyPI) for code
archive files, download, unpack and then install them.

[PEP 632](https://www.python.org/dev/peps/pep-0632) specifies that Python >=
version 3.12 will no longer have the `distutils` module in the standard
library, so we will no longer be able to do `import distutils`.  This will
not have great practical effect on common usage, because Setuptools now
carries the Distutils code, and continues to use its own copy for internal
use. Setuptools will continue to be Distutils+ (with some [minor
exceptions](https://www.python.org/dev/peps/pep-0632/#id23).

### Pip

The next step important step was the release of the [pip
utility](https://pypi.org/project/pip) Package Installer for Python.  This
quickly replaced `easy_install` as the standard tool for installing Python
packages from the command line, especially from PyPI, as in:

```python
pip install my_package
```

As for the earlier `easy_install`, this incantation will search for
`my_package` in the PyPI, download it, unpack and install it correctly in
a suitable directory on the Python search path.

### Wheels

The biggest problem at this point in the story was that Pip always installed
Python packages by downloading a source archive, unpacking it, and then
installing using a mechanism similar to `python setup.py install` above.  This
was completely acceptable for pure Python packages, but impractical for larger
packages that needed compilation.   Very common packages like Scipy or
Matplotlib could take a very long time to compile, and have various external
libraries that they relied on at compile-time. The external libraries had to be
installed before running e.g. `pip install scipy`, in practice by reading and
following the installation instructions.  These pre-installation steps were
fairly complicated and fragile for an inexperienced user.  Again, because Pip
called into Distutils to build compiled code, the user needed a suitable
compiler already installed and configured on their system.

The solution to this problem was the [Wheel format
specification](https://www.python.org/dev/peps/pep-0427).  This defined
a simple zip-file layout to store files that can be directly installed,
including pre-built compiled binaries, such as Python extensions.

After integration with Pip, and the PyPI, this meant that the release manager
for packages like Scipy could build a Wheel for each platform and Python
version they wanted to support.  Each platform and Python version results in
wheel (zip file) with a different specific filename, so that Pip can identify
the wheel corresponding to the system to which it is installing.  For example,
the current (at time of writing) [Scipy 1.71 release on
PyPI](https://pypi.org/project/scipy/1.7.1/) includes [many different Wheel
files](https://pypi.org/project/scipy/1.7.1/#files), corresponding to the
different platforms and Python versions, e.g.:

* `scipy-1.7.1-cp37-cp37m-macosx_10_9_x86_64.whl` for any macOS >= version
  10.9, and Python version 3.7 and
* `scipy-1.7.1-cp39-cp39-win_amd64.whl` for 64-bit Windows and Python 3.9.

The initial and current solution to the problem of external libraries was
a crude one — identify all the linked external libraries for compiled binaries
in the wheel, and copy these into the Wheel, while resetting the binary files
to point to these library copies.  The [Delocate
utility](https://github.com/matthew-brett/delocate) does this job on macOS, and
later, [auditwheel](https://github.com/pypa/auditwheel) implemented the
equivalent functionality for Linux.
[Delvewheel](https://github.com/adang1345/delvewheel) looks like it does
something similar on Windows, but it does not yet appear to be widely used. See
the discussion
[here](https://discuss.python.org/t/delocate-auditwheel-but-for-windows/2589)
for more detail and more links.

For example, Pip has a `wheel` subcommand that accepts a source tree directory (see above) and can build a wheel for the system on which Pip is running:

```bash
cd my_package
pip wheel .
```

## Separation of concerns

The next big step in packaging has been the realization that Pip and
Setuptools / Distutils are doing various different tasks, and these tasks can
and should be separable.

Tom Kluver's [Flit](https://flit.readthedocs.io) tool was an early development
of this thinking.  Flit is a Python command line package that greatly
simplifies builds of Sdists and wheels for pure-Python packages, and does not
use Setuptools or Distutils.

[PEP 517](https://www.python.org/dev/peps/pep-0517/) has a good discussion of
the analysis of various packaging concerns.  The PEP distinguishes the
following tasks within the general umbrella of package management.

* *Integration frontend* - a tool that accepts a set of package requirements,
  and installs these packages on the user's system.  In the command `pip install numpy scipy`, pip is acting as an integration frontend.
* *Build frontend* - a tool that accepts a source tree (such as an unpackaged
  source code archive or version control checkout), and builds a Wheel or a
  source distribution (Sdist). In the command `pip wheel ./`, Pip is acting as
  a build frontend for a wheel. `flit build` has flit acting as a build
  frontend for both wheels and Sdists.
* *Build backend* - the tool that the *build frontend* is using to compile the
  files comprising the wheel.   For most packages this will end up being
  Setuptools (Distutils+), but there are other implementations we will come onto soon.

The PEP explains these terms well:

> An *integration frontend* is a tool that users might run that takes a
set of package requirements (e.g. a requirements.txt file) and attempts to
update a working environment to satisfy those requirements. This may require
locating, building, and installing a combination of wheels and sdists. In a
command like `pip install lxml==2.4.0`, pip is acting as an integration
frontend.

> A *build frontend* is a tool that users might run that takes arbitrary
source trees or source distributions and builds wheels from them. The actual
building is done by each source tree's *build backend*. In a command like `pip
wheel some-directory/`, pip is acting as a build
frontend.

Notice that the traditional use of Setuptools/Distutils+ means that Setuptools
acts as both a build frontend and a build backend.

PEP 517 was designed to define the standards that make it possible to write
and use new build frontends and backends, so that we (the packagers) no longer
have to use Setuptools (Distutils+).  In particular we can switch out to other build frontends and backends, and we can write our own.

Along with [PEP518](https://www.python.org/dev/peps/pep-0518), PEP517
specifies sections in a configuration file called `pyproject.toml` that define
the build frontend and backend.  PEP517 gives the example of using the `flit` tool as build frontend and build backend, with this configuration (adapted to modern Flit usage):

```yaml
    [build-system]
    # Defined by PEP 518; specifies package(s).
    requires = ["flit"]
    # Defined by this PEP; specifies Python object.
    build-backend = "flit.buildapi"
```

This example says that, in order to build wheels or source distributions (Sdists), we

The build backend object must implement at least these Python callables:

* `build_sdist` (builds an sdist).
* `build_wheel` (builds a wheel).

For example, for the example above, the `main` object within `flit.api` must
have calleables `build_wheel` and `build_sdist` (e.g.
`flit.api.main.build_wheel`.

The build backend may also implement these callables:

* `get_requires_for_build_sdist` (returns list of specifications for packages
  that should be installed before running `build_sdist` above).
* `get_requires_for_build_wheel` (returns list of specifications for packages
  that should be installed before running `build_wheel` above).
* `prepare_metadata_for_build_wheel` (prepares `.dist-info` directory
  containing wheel metadata for wheel to be build).

The frontend will always call `build_sdist` to build an Sdist, and
`build_wheel` to build a wheel.  It's up to the frontend whether to try
calling the other optional backend functions like
`get_requires_for_build_sdist`.

[PEP 660](https://www.python.org/dev/peps/pep-0660/) added three optional
build backend callables to work with *editable installs*.  Editable installs
are installs where the Python and compiled files for the package being
imported are still in their source tree.  The optional callables are:

* `build_editable`
* `get_requires_for_build_editable`
* `prepare_metadata_for_build_editable`

with the obvious interpretations.

See also [PEP 621](https://www.python.org/dev/peps/pep-0621/) for more specifications for storing project metadata.

## Some backends implementing PEP517

* *Setuptools*.  Have a look at `setuptools.build_meta.__legacy__` for the
  build backend functions above.
* *Flit*: `flit.buildapi`. (only handles pure-Python packages).
* [Enscons](https://pypi.org/project/enscons): `enscons.api`.  Uses the [Scons
  build dependency system](https://scons.org) to do the work of building.
* [Mesonpep517 package](https://thiblahute.gitlab.io/mesonpep517):
  `mesonpep517.buildapi`.  Uses Meson as build sub-system.
* [Mesonpy package](https://github.com/FFY00/mesonpy): `mesonpy` Uses Meson as
  build sub-system.
* [pep517 package](https://pypi.org/project/pep517): `pep517.envbuild`.
* [PDM PEP517](https://pypi.org/project/pdm-pep517): `pdm.pep517.api`.

There is some [discussions of a PEP517 build backend for
scikit-build](https://github.com/scikit-build/scikit-build/issues/124).

## PEP517 build frontends

* Pip (`pip wheel`)
* Flit (`flit build`)
* [Build package](https://github.com/pypa/build): `python -m build`
* [pep517 package](https://pypi.org/project/pep517): The high-level build
  calls are now [deprecated](https://github.com/pypa/pep517/pull/83).

## Backends not implementing PEP517

These are backends that replace, extend or hijack the `setup.py` Setuptools backend to do the work of building.

* Numpy distutils.
* [Scikit-build](https://scikit-build.readthedocs.io): Use with `from skbuild
  import setup` in `setup.py` file.  Uses CMake internally for building.
