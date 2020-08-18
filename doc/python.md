# Installing Python packages

**Home:** [Software management](INDEX.md)

### Contents

- [Introduction](#introduction)
- [Creating a Python wheel](#creating-a-python-wheel)
- [Forcing the recompilation of existing binary packages](#forcing-the-recompilation-of-existing-binary-packages)
- [Architecture dependent wheels](#architecture-dependent-wheels)
- [`build_wheel.sh` script](#build_wheelsh-script)
- [Specific examples of building wheels from source code](#specific-examples-of-building-wheels-from-source-code)
  - [Numpy with Intel compiler and MKL](#numpy-with-intel-compiler-and-mkl)
  - [Cython with GCC](#cython-with-gcc)
  - [h5py, with GCC and our module version](#h5py-with-gcc-and-our-module-version)
- [Troubleshooting](#troubleshooting)

## Introduction

You will find that Python modules in our stack include very few packages. They
include python default packages, `virtualenv`, `pip` and `wheel`. We also
provide a few modules for users to get a bundle of frequently used packages. One
such module is the module:

- `scipy-stack`

This module includes the packages listed in the [SciPy Stack
specification](https://www.scipy.org/stackspec.html). Another example is
`mpi4py`.

Beside these few bundle modules, we do not install Python packages as modules.
Instead, we provide [Python wheels](https://pythonwheels.com/) for users to
build their own virtual environment with whatever they need. Python wheels are
binary packages that can be installed with `pip`.

Since they are binary packages, they take a fraction of a second to install, and
need not to be compiled by users. A repository of binary packages on the
filesystem also acts as a local cache which does not require an Internet
connection. This means that users can bundle the creation of their Python
environment within their job script, even when compute nodes do not have access
to the Internet, making their jobs more reproducible.

## Creating a Python wheel

**See also:** The [`build_wheel.sh` script](#build_wheelsh-script)

Creating a Python wheel is often as simple as typing:

```
pip wheel <package>
```

For example:

```
[mboisson@build-node wheels]$ pip wheel pyaml
Collecting pyaml
  Using cached pyaml-16.12.2-py2.py3-none-any.whl
  Saved ./pyaml-16.12.2-py2.py3-none-any.whl
Collecting PyYAML (from pyaml)
  Using cached PyYAML-3.12.tar.gz
Skipping pyaml, due to already being wheel.
Building wheels for collected packages: PyYAML
  Running setup.py bdist_wheel for PyYAML … done
  Stored in directory: /home/mboisson/wheels
Successfully built PyYAML
```

Once the wheel is created, you want to test that it works correctly by
installing it in an environment:

```
[mboisson@build-node wheels]$ virtualenv test
New python executable in /home/mboisson/wheels/test/bin/python2.7
Also creating executable in /home/mboisson/wheels/test/bin/python
Installing setuptools, pip, wheel…done.
[mboisson@build-node wheels]$ source test/bin/activate
(test) [mboisson@build-node wheels]$ pip install PyYAML-3.12-cp27-cp27mu-linux_x86_64.whl
Processing ./PyYAML-3.12-cp27-cp27mu-linux_x86_64.whl
Installing collected packages: PyYAML
Successfully installed PyYAML-3.12
(test) [mboisson@build-node wheels]$ pip install pyaml-16.12.2-py2.py3-none-any.whl
Processing ./pyaml-16.12.2-py2.py3-none-any.whl
Requirement already satisfied (use –upgrade to upgrade): PyYAML in ./test/lib/python2.7/site-packages (from pyaml==16.12.2)
Installing collected packages: pyaml
Successfully installed pyaml-16.12.2
(test) [mboisson@build-node wheels]$ python -c “import pyaml”
```

You should minimally test that the package imports correctly without error. If
there are tests available for this specific package, please run them. Once you
are satisfied that it works, you need to copy this binary to our wheelhouse.

We have a hierarchical wheelhouse layout per OS-abstraction layer (NIX, Gentoo)
and per architecture. They are located here:

```
/cvmfs/soft.computecanada.ca/custom/python/wheelhouse
               ├── generic              totally generic
               ├── nix/
               │   ├── generic          NIX but arch-independent
               │   ├── sse3             NIX SSE3 and above
               │   ├── avx              NIX AVX  and above
               │   ├── avx2             NIX AVX2 and above
               │   └── avx512           NIX AVX512
               └── gentoo/
                   ├── generic          Gentoo but arch-independent
                   ├── sse3             Gentoo SSE3 and above
                   ├── avx              Gentoo AVX  and above
                   ├── avx2             Gentoo AVX2 and above
                   └── avx512           Gentoo AVX512
```


From then on, when a user runs a `pip install <package>`, if it is found in
these paths, `pip` will use this version if our version is the most recent. Once
the wheel is copied, you can sync to CVMFS, using the `--what custom-python`
option of `rsnt-sync`.

## Forcing the recompilation of existing binary packages

Nowadays, many Python packages ship in binary format when you install them with
`pip`. This includes `numpy`, `scipy`, `h5py`, `matplotlib`, `pandas`, etc.
These binary formats will typically provide their own implementation of
underlying libraries, which may not be adapted to our system. For example,
`numpy` provides OpenBLAS rather than MKL. If we want to force compiling `numpy`
to use MKL, we would proceed as follow :

```
[~] pip download –no-binary :all: numpy
[~] unzip numpy-1.12.0.zip
[~] cd numpy-1.12.0
[~] module load imkl
[~]cat << EOF > site.cfg
[mkl]
library_dirs = $MKLROOT/lib/intel64
include_dirs = $MKLROOT/include
mkl_libs = mkl_rt
lapack_libs =
EOF
[~] python setup.py config –compiler=intelem build_clib –compiler=intelem build_ext –compiler=intelem<sup>[[#cmnt10|[j]]][[#cmnt11|[k]]]</sup> bdist_wheel
[~] ls ./dist/<sup>[[#cmnt12|[l]]]</sup>numpy-1.12.0-cp27-cp27mu-linux_x86_64.whl
```

You should now be able to install this wheel in your virtual environment and see
that it was linked against MKL. When testing, ensure that you first run:

```
module unload imkl
```

Given that our compilation always uses `rpath`, this should work without the
`imkl` module loaded.

Also note that it is not always desirable to force compilation from source. It
may be tricky to get it to link correctly, and is not always worth the effort.

## Architecture dependent wheels

It may sometimes be useful to compile wheels optimised for a specific
architecture, such as `avx2`. In many cases it does not make a difference
however. The wheel houses do allow for having architecture dependent wheels. In
case a wheel is found in both the `avx2` and the `generic` folder, the `avx2`
version will have precedence.

## `build_wheel.sh` script

The RSNT has developed a script to build many different wheels automatically.

The script as well as documentation is available on Compute Canada's Github:
https://github.com/ComputeCanada/wheels_builder/.

## Specific examples of building wheels from source code

### Numpy with Intel compiler and MKL

```
[~] module load imkl
[~]cat << EOF > site.cfg
[mkl]
library_dirs = $MKLROOT/lib/intel64
include_dirs = $MKLROOT/include
mkl_libs = mkl_rt
lapack_libs =
EOF
[~] python setup.py config -compiler=intelem build_clib -compiler=intelem build_ext -compiler=intelem<sup>[[#cmnt13|[m]]][[#cmnt14|[n]]]</sup> bdist_wheel
```

### Cython with GCC

```
python setup.py bdist_wheel
```

### h5py, with GCC and our module version

```
module load hdf5
virtualenv build_venv
source build_venv/bin/activate
pip install Cython
HDF5_DIR=$(dirname $(dirname $(which h5dump))) python setup.py bdist_wheel
```

## Troubleshooting

Try adding `--no-cache-dir` to `pip` commands if caching might be causing issues
(eg. older prerequisites are being used).  The `pip` cache can be deleted by
removing `~/.cache/pip` directory.
