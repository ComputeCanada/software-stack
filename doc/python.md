# Installing Python packages

**Home:** [Software management](INDEX.md)

### Contents

- [Introduction](#introduction)
- [Creating a Python wheel](#creating-a-python-wheel)
- [Architecture dependent wheels](#architecture-dependent-wheels)
- [Caveats](#caveats)
- [Troubleshooting](#troubleshooting)

## Introduction

You will find that Python modules in our stack include few packages. They
include a minimal set of packages, such as `virtualenv`, `setuptools`, `packaging`. `pip` and `wheel`.  

We also provide a few modules for users to get a bundle of frequently used packages. One such module is the module: `scipy-stack`.
This module includes `numpy`, `scipy`, `pandas`, `matplotlib` to name a few.

Note: the `scipy-stack` is mostly useful as a dependency for modules. One should use a virtual environment and requirements file where possible.

Other examples of module that contains Python extensions are:
* `mpi4py`
* `pyarrow` (Arrow)
* `opencv-python` (OpenCV)

In general, we do not install Python packages as modules.
Instead, we provide [Python wheels](https://pythonwheels.com/) for users to
build their own virtual environment with whatever they need. Python wheels are
binary packages that can be installed with `pip`.

Since they are binary packages, they take a fraction of a second to install, and
need not to be compiled by users. A repository of binary packages on the
file system also acts as a local cache which does not require an Internet
connection. This means that users can bundle the creation of their Python
environment within their job script, even when compute nodes do not have access
to the Internet, making their jobs more reproducible.

## Creating a Python wheel
The RSNT has developed a script to build many different wheels automatically.

The script as well as documentation is available on Compute Canada's Github:
https://github.com/ComputeCanada/wheels_builder/.

To build a package, from the `wheels_builder` directory:
```bash
bash build_wheels.sh --package biopython --version 1.85 --verbose 3
```

The scripts will test that the package imports correctly without error. If the package is built with GPU support, it should be tested with a gpu device on a production system.

If there are tests available for this specific package, please run them. 

To get your wheel into CVMFS, use the [cp_wheels.sh](https://github.com/ComputeCanada/wheels_builder/blob/main/cp_wheels.sh) script. Calling it without parameters will handle the wheels in the current directory. It has many advantages compared to copying manually, namely: it will automatically determine the correct destinations in CVMFS, and it will catch some mistakes, such as forgetting to add the local version identifier.

We have a hierarchical wheelhouse layout per OS-abstraction layer (Gentoo)
and per architecture. They are located here:
```
/cvmfs/soft.computecanada.ca/custom/python/wheelhouse
               ├── generic              generic
               ├── gentoo2023           
               │    ├── generic         Gentoo2023 generic and x86-64-v3 optimized
               │    ├── x86-64-v3       Gentoo2023 x86-64-v3 arch-dependent
               │    └── x86-64-v4       Gentoo2023 x86-64-v4 arch-dependent
               ├── nix/  **DEPRECATED**
               │   ├── generic          NIX but arch-independent
               │   ├── sse3             NIX SSE3 and above
               │   ├── avx              NIX AVX  and above
               │   ├── avx2             NIX AVX2 and above
               │   └── avx512           NIX AVX512
               ├── gentoo2020
               │   ├── avx              Gentoo2020 AVX  and above
               │   ├── avx2             Gentoo2020 AVX2  and above
               │   ├── avx512           Gentoo2020 AVX512  and above
               │   ├── generic          Gentoo2020 but arch-independent
               │   └── sse3             Gentoo2020 SSE3 and above
               └── gentoo/
                   ├── generic          Gentoo but arch-independent
                   ├── sse3             Gentoo SSE3 and above
                   ├── avx              Gentoo AVX  and above
                   ├── avx2             Gentoo AVX2 and above
                   └── avx512           Gentoo AVX512
```

Once the wheel is copied, you can sync to CVMFS, using the `--what wheels --software <name>` option of `rsnt-sync`. See the [CVMFS documentation](cvmfs.md).

From then on, when a user runs a `pip install <package>`, if it is found in
these paths, `pip` will use this version if our version is the most recent.

## Architecture dependent wheels

It may sometimes be useful to compile wheels optimized for a specific
architecture, such as `avx512`. The wheelhouse do allow for having architecture dependent wheels.

## Caveats
- Some generic wheels (`py3`) might actually contain libraries. We should then update the tags accordingly.
- Some wheels have libraries that depends on other libraries that will be present in a virtual environment. Common examples are packages depending on `libtorch.so` or `libtensorflow_framework.so`
- Nvidia released their libraries on PyPI and some wheels have them as requirements. Since they are duplicates of our modules, they should be removed from the wheel's requirement. For example, `nvidia-cublas-cu12==12.8.3.14`.

## Troubleshooting

* Try adding `--no-cache-dir` to `pip` commands if caching might be causing issues
(eg. older prerequisites are being used).  The `pip` cache can be deleted by
removing `~/.cache/pip` directory.

* Remove any local python installation under `~/.local/bin` and `~/.local/lib/python*` as they more then often conflicts and create surprising issues.
