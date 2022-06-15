# Windows configuration

In an elevated terminal:

```
choco install git hub unzip python vim rtools
```

## Testing scipy builds

```powershell
# Init scipy build.
$ucrt_tools = "C:\rtools40"
# Or:
# $ucrt_tools = "C:\msys64-ucrt"
# Compilers, and make tools.
$env:PATH = "$ucrt_tools\ucrt64\bin;$ucrt_tools\usr\bin;$env:PATH"
# In order to find OpenBLAS
$env:PKG_CONFIG_PATH = "c:\opt\openblas\64\lib\pkgconfig;"
```

## Python setup

I install everything as user.  Make sure user script dir on path with:

```powershell
try {
$site_dir = python -c 'import os, site; print(os.path.dirname(site.USER_SITE))'
$env:PATH = "$env:PATH;$site_dir\Scripts"
}
catch [System.Management.Automation.CommandNotFoundException]
{echo "No Python on path" }
```

## Numpy builds

Do init as above.  Check you have the right `gcc`.

```
get-command gcc
```

Install a completely up to date Numpy:

```
pip install --user --extra-index-url https://pypi.anaconda.org/scipy-wheels-nightly/simple --pre --upgrade numpy
```

Try a compile.

```
python setup.py bdist_wheel config_fc --fcompiler=gnu95 config_cc --compiler=mingw32
```

Patch everything from Mingw-w64 patches (in bash)

```bash
# apply_patches
for _patch in ls ../MINGW-packages/mingw-w64-python-numpy/*.patch
do
echo "Applying ${_patch}"
patch -Nbp1 -i ${_patch}
done
```

## Windows configuration

Install package via `choco`.

Get all versions of particular package with:

```
choco list --all-versions --exact python
```

Example install

```
choco install -y --side-by-side python --version==3.9.6

```

## Some Python / MSVC versions

```
PS C:\> .\Python27\python.exe .\check_versions.py
('sys.version', '2.7.18 (v2.7.18:8d21aa21f2, Apr 20 2020, 13:25:05) [MSC v.1500 64 bit (AMD64)]')
('distutils build version', 9.0)
('msvcrt.CRT_ASSEMBLY_VERSION', '9.0.21022.8')
PS C:\> C:\ProgramData\chocolatey\bin\python.exe .\check_versions.py
sys.version 3.5.0 (v3.5.0:374f501f4567, Sep 13 2015, 02:27:37) [MSC v.1900 64 bit (AMD64)]
distutils build version 14.0
msvcrt.CRT_ASSEMBLY_VERSION 14.0.23026.0
PS C:\> .\Python36\python.exe .\check_versions.py
sys.version 3.6.0 (v3.6.0:41df79263a11, Dec 23 2016, 08:06:12) [MSC v.1900 64 bit (AMD64)]
distutils build version 14.0
msvcrt.CRT_ASSEMBLY_VERSION 14.0.24210.0
PS C:\> .\Python36\python.exe .\check_versions.py
sys.version 3.6.8 (tags/v3.6.8:3c6b436a57, Dec 24 2018, 00:16:47) [MSC v.1916 64 bit (AMD64)]
distutils build version 14.1
msvcrt.CRT_ASSEMBLY_VERSION 14.16.27023.1
PS C:\> .\Python37\python.exe .\check_versions.py
sys.version 3.7.0 (v3.7.0:1bf9cc5093, Jun 27 2018, 04:59:51) [MSC v.1914 64 bit (AMD64)]
distutils build version 14.1
msvcrt.CRT_ASSEMBLY_VERSION 14.14.26428.1
PS C:\> .\Python37\python.exe .\io\check_versions.py
sys.version 3.7.1 (v3.7.1:260ec2c36a, Oct 20 2018, 14:57:15) [MSC v.1915 64 bit (AMD64)]
distutils build version 14.1
msvcrt.CRT_ASSEMBLY_VERSION 14.15.26726.0
PS C:\> .\Python37\python.exe .\check_versions.py
sys.version 3.7.9 (tags/v3.7.9:13c94747c7, Aug 17 2020, 18:58:18) [MSC v.1900 64 bit (AMD64)]
distutils build version 14.0
msvcrt.CRT_ASSEMBLY_VERSION 14.0.24234.1
PS C:\> .\Python38\python.exe .\check_versions.py
sys.version 3.8.0 (tags/v3.8.0:fa919fd, Oct 14 2019, 19:37:50) [MSC v.1916 64 bit (AMD64)]
distutils build version 14.1
msvcrt.CRT_ASSEMBLY_VERSION 14.16.27023.1
PS C:\> .\Python38\python.exe .\check_versions.py
sys.version 3.8.10 (tags/v3.8.10:3d8993a, May  3 2021, 11:48:03) [MSC v.1928 64 bit (AMD64)]
distutils build version 14.2
msvcrt.CRT_ASSEMBLY_VERSION 14.28.29912.0
PS C:\> .\Python39\python.exe .\check_versions.py
sys.version 3.9.0 (tags/v3.9.0:9cf6752, Oct  5 2020, 15:34:40) [MSC v.1927 64 bit (AMD64)]
distutils build version 14.2
msvcrt.CRT_ASSEMBLY_VERSION 14.27.29110.0
PS C:\> .\Python39\python.exe .\check_versions.py
sys.version 3.9.7 (tags/v3.9.7:1016ef3, Aug 30 2021, 20:19:38) [MSC v.1929 64 bit (AMD64)]
distutils build version 14.2
msvcrt.CRT_ASSEMBLY_VERSION 14.29.30133.0
PS C:\> .\Python310\python.exe .\check_versions.py
C:\check_versions.py:7: DeprecationWarning: The distutils package is deprecated and slated for removal in Python 3.12. Use setuptoo
ls or check PEP 632 for potential alternatives
  from distutils.msvccompiler import get_build_version
sys.version 3.10.0 (tags/v3.10.0:b494f59, Oct  4 2021, 19:00:18) [MSC v.1929 64 bit (AMD64)]
distutils build version 14.2
msvcrt.CRT_ASSEMBLY_VERSION 14.29.30133.0
```

## Compiling Scipy with MSVC

```
# Need clang-cl.exe
choco install llvm
```

Need gfortran, of some vintage.

My `site.cfg` file, after compiling OpenBLAS:

```
[openblas]
libraries = libopenblas
library_dirs = c:\opt\openblas\if_32\64\lib
include_dirs = c:\opt\openblas\if_32\64\include
```

## Built OpenBLAS wheels

From <https://anaconda.org/multibuild-wheels-staging/openblas-libs/v0.3.18/download/openblas-v0.3.18-win_amd64-gcc_8_1_0.zip>.

e.g.

```
wget `
    https://anaconda.org/multibuild-wheels-staging/openblas-libs/v0.3.18/download/openblas-v0.3.18-win_amd64-gcc_8_1_0.zip `
    -UseBasicParsing -OutFile openblas.zip
```

## Flang

<https://github.com/msys2/MINGW-packages/pull/9782>

## msys configuration

Follow instructions at <https://www.msys2.org>:

* Download the installer: msys2-x86_64-20210725.exe
* Run, install to `c:\msys64`, Run MSYS2.
* `pacman -Syu`
* Exit, run `MSYS MinGW UCRT 64-bit` from start menu.
* `pacman -Su` again.

Then:
* `pacman -S --needed mingw-w64-ucrt-x86_64-toolchain`

On threads: <https://github.com/meganz/mingw-std-threads>.
