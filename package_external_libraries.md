# Python packages and external libraries

As you will have seen in [the packaging page](./farewell_distutils.md), the
default build of compiled code will often result in binaries that depend on
external libraries, not present in the wheel.

Call our package — `my_package`.

If `my_package` compiled code is to work correctly on the users' system, the
binaries will need access to the external libraries on their system.

There are various solutions to the this problem:

* Make another Python package that ships these external libraries. Call the
  library package `my_library_package`.  Make `my_package` depend on
  `my_library_package`, and make sure `my_package` code finds the libraries
  from `my_library_package` at run-time.  This what your Linux package manager
  does, and it is what Conda / Mamba does.  Nathaniel Smith wrote a proposal
  for doing this for PyPI / wheels at the [PyNativelib
  page](https://github.com/njsmith/wheel-builders/blob/pynativelib-proposal/pynativelib-proposal.rst)
  — but we don't know of anyone using this method yet.
* Tell the user that they must install the libraries somehow before using an
  installed version of `my_package`.
* Ship the external libraries inside `my_package`.  That is the standard
  approach for wheels.  Read on for more information.

Conda has a more general page on this topic called [making packages
relocatable](https://docs.conda.io/projects/conda-build/en/latest/resources/make-relocatable.html)

## Shipping external libraries in wheels

We can run post-processing on wheels to identify all the linked external
libraries for compiled binaries in the wheel, and copy these into the Wheel,
while resetting the binary files to point to these library copies.  The
[Delocate utility](https://github.com/matthew-brett/delocate) does this job on
macOS, and later, [auditwheel](https://github.com/pypa/auditwheel) implemented
the equivalent functionality for Linux.
[Delvewheel](https://github.com/adang1345/delvewheel) looks like it does
something similar on Windows, but it does not yet appear to be widely used.  It appears that [MachoMangler](https://github.com/njsmith/machomachomangler) is the usual tool for relinking / rewriting compiled binaries on Windows.
See the discussion on [packaging DLLs on
Windows](https://discuss.python.org/t/packaging-dlls-on-windows/1401) and
[Delocate/Auditwheel for
Windows](https://discuss.python.org/t/delocate-auditwheel-but-for-windows/2589)
for more detail and more links.
