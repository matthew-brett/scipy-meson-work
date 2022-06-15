# Working with Scipy on Windows

To start, you will need:

* Python
* Git
* `unzip`, `wget`
* The [rtools](https://cran.r-project.org/bin/windows/Rtools) compiler chain.

These are instructions for getting those requirements on a bare Windows installation, and then using them to build Scipy, install and test.

If the instructions here don't work, you might consider checking against the
[Scipy
.github/workflow/windows.yml](https://github.com/scipy/scipy/blob/main/.github/workflows/windows.yml)
file, for the current Continuous Integration build recipe.

## Installing requirements

We recommend the [Chocolatey package manager](https://chocolatey.org).

First follow the [instructions to install choco](https://chocolatey.org/install#individual).

Consider installing "Windows Terminal" from the Microsoft App Store.  It's a
pleasant and usable terminal, from which you can use Powershell or the old Cmd
shell.

We recommend Powershell, and all the commands here are using Powershell.

In a terminal such as Powershell, with Administrator privileges:

```
choco install -y git unzip wget python rtools
```

Now open a new Powershell terminal as your usual user.

## Working in a virtual environment

You can work in a Virtualenv or a Conda environment.  Here are the instructions for a Virtualenv:

```powershell
# Install virtualenv
python -m pip install --user virtualenv
# Make directory to contain environments.
mkdir c:\envs
# Make, activate environment.
python -m virtualenv c:\envs\venv
. c:\envs\venv\Scripts\activate.ps1
```

## Get requirements

In the same Powershell terminal:

```powershell
pip install numpy==1.22.2 cython pybind11 pythran meson ninja pytest pytest-xdist
```

Get the compiled OpenBLAS binaries, point environment to these binaries.

```powershell
# wget.exe to use Choco wget binary instead of wget Powershell shortcut.
wget.exe https://github.com/scipy/scipy-ci-artifacts/raw/main/openblas_32_if.zip
unzip -d c:\ openblas_32_if.zip
$env:PKG_CONFIG_PATH="c:\opt\openblas\if_32\64\lib\pkgconfig"
```

Point the terminal to the RTools binaries:

```powershell
$ucrt_tools = "C:\rtools40"
$env:PATH = "$env:PATH;$ucrt_tools\ucrt64\bin;$ucrt_tools\usr\bin"
```

## Get, build and install Scipy

In the same Powershell terminal:

```powershell
mkdir c:\repos
cd c:\repos
git clone https://github.com/scipy/scipy.git
cd scipy
git submodule update --init --recursive
```

Build and install Scipy

```powershell
meson build --prefix=$PWD\build
ninja -C build
meson install -C build
```

Copy OpenBLAS binaries into install directory, and point Scipy to this binary:

```powershell
$installed_path="$PWD\build\Lib\site-packages"
$scipy_path = "${installed_path}\scipy"
$libs_path = "${scipy_path}\.libs"
mkdir ${libs_path}
$ob_path = (pkg-config --variable libdir openblas) -replace "lib", "bin"
cp $ob_path/*.dll $libs_path
# Write _distributor_init.py to scipy dir to load .libs DLLs.
& python tools\openblas_support.py --write-init $scipy_path
```

## Run tests

```powershell
$env:PYTHONPATH="${installed_path}"
$env:SCIPY_USE_PROPACK=1
mkdir tmp
cd tmp
python -c 'import scipy; scipy.test()'
```
