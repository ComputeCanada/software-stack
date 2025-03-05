# Installing with EasyBuild

**Home:** [Software management](INDEX.md)

**Note:** Before starting, please make sure that you have followed the generic
steps and the EasyBuild specific steps described in [initial setup](setup.md).

The following contains the essential parts you will need to use EasyBuild on our
infrastructure. For detailed EasyBuild documentation, see:

- [EasyBuild documentation](http://easybuild.readthedocs.io/en/latest)
- [Overview of generic
  EasyBlocks](https://easybuild.readthedocs.io/en/latest/version-specific/generic_easyblocks.html#generic-easyblocks)

## Contents

- [Installing with EasyBuild](#installing-with-easybuild)
  * [Contents](#contents)
  * [Specificities of EasyBuild on Compute Canada](#specificities-of-easybuild-on-compute-canada)
  * [Searching for packages in EasyBuild](#searching-for-packages-in-easybuild)
  * [Toolchains](#toolchains)
    + [Background](#background)
    + [Compute Canada toolchains](#compute-canada-toolchains)
      - [Toolchain hierarchy](#toolchain-hierarchy)
      - [Core toolchains](#core-toolchains)
      - [Compiler-only toolchains](#compiler-only-toolchains)
      - [Family toolchains](#family-toolchains)
      - [Toolchains to use with StdEnv/2020](#toolchains-to-use-with-stdenv-2020)
  * [Installing a package in EasyBuild](#installing-a-package-in-easybuild)
    + [Installing for a different toolchain](#installing-for-a-different-toolchain)
    + [Installing for a different architecture](#installing-for-a-different-architecture)
    + [Installing for multiple architectures and toolchains](#installing-for-multiple-architectures-and-toolchains)
    + [Installing for a different StdEnv](#installing-for-a-different-stdenv)
    + [Creating or changing a recipe](#creating-or-changing-a-recipe)
    + [Checksums in EasyConfig recipes](#checksums-in-easyconfig-recipes)
  * [Tips and tricks, troubleshooting](#tips-and-tricks--troubleshooting)
    + [Rebuilding installed software](#rebuilding-installed-software)
    + [Downloading source or binaries manually](#downloading-source-or-binaries-manually)
    + [Fixing the loader and runpath](#fixing-the-loader-and-runpath)
    + [Dependencies](#dependencies)
    + [Debugging your build](#debugging-your-build)
    + [Providing the source distribution](#providing-the-source-distribution)
    + [Other topics to be covered in the future](#other-topics-to-be-covered-in-the-future)
  * [Contributing back to EasyBuild](#contributing-back-to-easybuild)
  * [Installing restricted software](#installing-restricted-software)
    + [Different cases](#different-cases)
    + [Informing users of restrictions at module load time](#informing-users-of-restrictions-at-module-load-time)
    + [Installing the software](#installing-the-software)
    + [Deploying POSIX group-restricted software with CVMFS](#deploying-posix-group-restricted-software-with-cvmfs)
    + [Hiding POSIX group-restricted software on systems that don’t have them](#hiding-posix-group-restricted-software-on-systems-that-don-t-have-them)
    + [Existing POSIX groups to manage access to pieces of software](#existing-posix-groups-to-manage-access-to-pieces-of-software)
  * [Installing datasets](#installing-datasets)
    + [Deploying datasets with CVMFS](#deploying-datasets-with-cvmfs)
  * [FAQ](#faq)


## Specificities of EasyBuild on Compute Canada

While the vast majority of recipes provided by EasyBuild are reusable or easily
adaptable on Compute Canada, there are a number of specificities that you should
know when using EasyBuild on our software stack.

- **Do not use the `--robot`/`-r` flag when installing a software package.**
  This flag tends to install all of the dependencies, which might not be the
  default versions that we have.
- **Do not use the standard upstream toolchain `intel`.**  This
  toolchain uses Intel MPI which we don't use much.
  Prefer the toolchains listed in the EasyBuild toolchains section.
  The `foss` toolchains are now standard and recommended, unless it can be "lowered".
- **Do not use `versionsuffix`.** Our module naming scheme discards
  `versionsuffix`. Prefer using `modaltsoftname` to change the name of the
  module, rather than adding a version suffix.
- **Do not rewrite an EasyConfig file unless you absolutely need to.**   Prefer
  using the `--try-toolchain` option. This helps keeping installation
  procedures consistent across toolchains.

## Searching for packages in EasyBuild

You can search for packages (existing, installed or not) using this command:

```
eb -S REGEX
```

For example, `eb -S GROMACS` or `eb -S gromacs` gives a list of easyconfig
(`.eb`) files for GROMACS.

## Toolchains

### Background

Toolchains are a core concept within EasyBuild. A toolchain is a set of
recipes/modules that includes compilers (GCC, Intel, NVHPC), MPI implementations
(such as OpenMPI), Cuda,  and core mathematical libraries such as MKL and the
FlexiBLAS wrapper library.
Toolchains are layered, with more complex toolchains being built out of a
combination of sub-toolchains. Toolchains are installed as modules, but most of
them are hidden by default. You can get a list of toolchains using:

```
module --show_hidden avail
```

and looking for the hidden (`H`) modules. Typically, they will have obscure
names like `iompi`, `gomkl`, etc.  Not all toolchains are hidden. For example,
`iccifort` is associated with the `intel` modules (visible).

To obtain the meaning of the toolchain names, run:

```
eb --list-toolchains
```

**Important:** Toolchains in EasyBuild recipes are case-sensitive, while our
modules are all lowercase. Except `GCCCore`, `GCC`, `NVHPC`, all toolchain names
are lowercase.

**Important:** Toolchain versions are somewhat obscure since they are
combinations of sub-toolchain versions. To find what modules a toolchain loads,
we recommend running the following command:

```
module show <toolchain>/.version-number
```

### Compute Canada toolchains

Please stick with one of the standard CC toolchains, unless you have a good
reason not to. To get a complete up-to-date list of available toolchains, see
the above instructions.

Note that the `foss,2023a` toolchain and its subtoolchains `gofb`, `gompi`,
`gfb`, and `GCC` are the main toolchains that we use; its components
are loaded by default when logging into the system. 

#### Toolchain hierarchy

Toolchains are arranged in a hierarchy, as illustrated in the following figure:

![Toolchain hierarchy](images/toolchain-hierarchy.png)

#### Core toolchains

Core toolchains are not dependent on a specific compiler:

- `SYSTEM` in easyconfigs or `system,system` for `--try-toolchain`: For
  binary-only installations, and codes without architecture-dependent
  optimization.
- `GCCcore,12.3.0`: This uses
  GCC 12.3.0 with architecture-dependent optimization. It can
  be combined with FlexiBLAS in the `gcccoreflexiblas,2023a` toolchain. This toolchain is
  useful for codes that don't use MPI and are not typically compiled with the
  Intel compiler as well, such as R, Julia, Python, and various base libraries.

#### Compiler-only toolchains

- intel-compilers: 2023.2.1, 2024.2.0
- GCC: 12.3.0, 13.3.0
- NVHPC: 23.9, 25.1

#### Family toolchains

Family toolchains are a combination of **Compiler [+ MKL/FlexiBLAS [+ CUDA [+ Open
MPI]]]**, for example:

- GCC: `GCC`, `gmkl`, `gccflexiblas`, `gompi`, `gomkl`, `gofb`, `gofbf`,
  `gcccuda`, `gompic`, `gmklc`, `gccflexiblascuda`, `gomklc`, `gofbc`, `foss`
- Intel: `intel-compilers`, `iimkl`, `ifb`, `iompi`, `iomkl`, `iofb`, `iofbf`,
  `intelcompilerscuda`, `iompic`, `iimklc`, `ifbc`, `iomklc`, `iofbc`
- NVHPC: `NVHPC`, `nvompi`, `nvhpccuda`, `nvompic`

To better understand naming patterns for family toolchains, see the tables
below.

#### Toolchains to use with StdEnv/2023

```
|------------------------------------------|--------------------------------------|---------------------------------|
| Core-Level; Comp.Can. ; mostly upstream  |                    Intel             |                GCC              |
|------------------------------------------|--------------------------------------|---------------------------------|
| Compiler (arch-independent Core)         |                     n/a              |           SYSTEM                |
| Compiler (arch-dependent Core)           |                     n/a              |          GCCcore-12.3.0         |
| Compiler (a-d Core) + FlexiBLAS          |                     n/a              | gcccoreflexiblas-2023a          |
| Compiler (a-d Core)          + Cuda 12.2 |                     n/a              |      gcccorecuda-2023a          |
|------------------------------------------|--------------------------------------|---------------------------------|
| Compiler only                            |       intel-compilers-2023.2.1       |              GCC-12.3.0         |
| Compiler             + FlexiBLAS         |                   ifb-2023a          |     gccflexiblas-2023a          |
| Compiler             + FlexiBLAS + FFTW  |                  ifbf-2023a          |             gfbf-2023a          |
| Compiler + Open MPI                      |                 iompi-2023a          |            gompi-2023a          |
| Compiler + Open MPI  + FlexiBLAS (FB)    |                  iofb-2023a          |             gofb-2023a          |
| Compiler + Open MPI  + FB+ScaLAPACK+FFTW |                 iofbf-2023a          |             foss-2023a          |
| Compiler                     + Cuda 12.2 |    intelcompilerscuda-2023a          |          gcccuda-2023a          |
| Compiler + Open MPI          + Cuda 12.2 |                iompic-2023a          |           gompic-2023a          |
| Compiler            + FB     + Cuda 12.2 |                  ifbc-2023a          | gccflexiblascuda-2023a          |
| Compiler            + FB+FFTW+ Cuda 12.2 |                 ifbfc-2023a          |            gfbfc-2023a          |
| Compiler + Open MPI + FB     + Cuda 12.2 |                 iofbc-2023a          |            gofbc-2023a          |
| Compiler + OpenMPI+FB+ScaLAPACK+FFTW+Cuda|                iofbfc-2023a          |         fosscuda-2023a          |
|------------------------------------------|--------------------------------------|---------------------------------|

Modules used by toolchains:
 intel/2023.2.1   flexiblas/3.3.1  openmpi/4.1.5 cuda/12.2
 gcc/12.3         flexiblas/3.3.1  openmpi/4.1.5 cuda/12.2

There are also `2024a` versions, which should only be used if there is a clear benefit. They use the modules
 intel/2024.2.0   flexiblas/3.4.4  openmpi/5.0.3 cuda/12.6
 gcc/13.3         flexiblas/3.4.4  openmpi/5.0.3 cuda/12.6
```

## Installing a package in EasyBuild

See also: [Installing restricted software](#installing-restricted-software)

On the build infrastructure we strongly recommend starting an interactive job first, via `salloc`, e.g.
```
salloc -c 8 --mem=24G --time=8:00:00
```
This way your build will not slow down, nor be slowed down by, other users on the login node.

If a package already has an existing recipe, you can install it for testing (in `$HOME/.local/easybuild`, which after testing can be safely deleted) using the following command:

```
eb <name of easyconfig file>
```

For a global installation (**you must do this step before you can deploy to CVMFS**):

```
## Pull to the ebuser's account first
sudo -iu ebuser eb-pull-cc
sudo -i -u ebuser eb <name of easyconfig file>
```

This will, by default, create an AVX2/x86-64-v3 (Haswell processor and up) optimized
executable for all software that is not installed at the “Core” level.

### Installing for a different toolchain

If the recipe you want to use already exists but uses the a different toolchain,
you can sometimes install it using a single command:

```
sudo -i -u ebuser eb HPL-2.3-foss-2023a.eb --try-toolchain=gofb,2023a
```

**Note:** The actual generated easyconfig will be saved into
`/cvmfs/soft.computecanada.ca/easybuild/ebfiles_repo*`.

### Installing for a different architecture

Once a recipe has been installed for the architecture `avx2`, other
architectures can be compiled.

Skylake/Zen4 processors and up (x86-64-v4):

```
sudo -i -u ebuser RSNT_ARCH=avx512 eb <name of easyconfig file>
```

**Note:** The actual generated easyconfig will be saved into
`/cvmfs/soft.computecanada.ca/easybuild/ebfiles_repo*`.

### Installing for multiple architectures and toolchains
Quite often, we want to compile a recipe for multiple architectures, and possibly multiple toolchains. This can be done easily
using `parallel`. For example, to build `HDF5-1.10.6` with both GCC and Intel-based OpenMPI, for both architectures `avx2` and `avx512`, you can execute : 

```
parallel sudo -iu ebuser RSNT_ARCH={1} eb HDF5-1.10.6-gompi-2020a.eb --try-toolchain={2} --disable-map-toolchains ::: avx2 avx512 ::: gompi,2023a iompi,2023a
``` 


### Installing for a different StdEnv

The `StdEnv/2023` is built on top of Gentoo.
In order for EasyBuild to choose the correct toolchains and underlying Gentoo, a suitable StdEnv needs
to be loaded before invoking `eb`. `StdEnv/2023` is the default environment.


### Creating or changing a recipe

Because often a few things need changing in the easyconfig file we are going to
check out a git repository and work with that.

```
cd easybuild-easyconfigs
git pull
```

Then you make the directory of (if needed) and `cd` to the package, e.g.:

```
mkdir -p easybuild/easyconfigs/i/igraph
cd easybuild/easyconfigs/i/igraph
```

And create a new easyconfig to make modifications, by copying from the upstream repository:
```
cp $EBROOTGENTOO/easybuild/easyconfigs/i/igraph/igraph-0.10.6-foss-2022b.eb igraph-0.10.6-gcccoreflexiblas-2023a.eb
```
if the recipe were in the 2020 easyconfig repository, please copy it, e.g.
```
cp igraph-0.10.6-foss-2022b.eb igraph-0.10.6-gcccoreflexiblas-2023a.eb
```

The change from `foss-2022b` to `gcccoreflexiblas-2023a` is a toolchain change. We do
not expose the toolchains to users but use them internally to denote compiler,
MPI and linear algebra combinations.

File `igraph-0.10.6-gcccoreflexiblas-2023a.eb` is then edited as follows:

```
easyblock = 'CMakeMake'

name = 'igraph'
version = '0.10.6'

homepage = 'https://igraph.org'
description = """igraph is a collection of network analysis tools with the emphasis on
efficiency, portability and ease of use. igraph is open source and free. igraph can be
programmed in R, Python and C/C++."""

toolchain = {'name': 'gcccoreflexiblas', 'version': '2023a'}
toolchainopts = {'pic': True}

source_urls = ['https://github.com/igraph/igraph/releases/download/%(version)s']
sources = [SOURCE_TAR_GZ]
checksums = ['99bf91ee90febeeb9a201f3e0c1d323c09214f0b5f37a4290dc3b63f52839d6d']

builddependencies = [
    ('CMake', '3.24.3'),
]

dependencies = [
    ('GLPK', '5.0'),
    ('libxml2', '2.10.3'),
    ('zlib', '1.2.12'),
    ('arpack-ng', '3.8.0'),
]

# Build static and shared libraries
configopts = ["-DBUILD_SHARED_LIBS=OFF", "-DBUILD_SHARED_LIBS=ON"]

multi_deps = {'Python': ['3.10', '3.11'] }
multi_deps_extensions_only = True

exts_defaultclass = 'PythonPackage'
exts_list = [
    ('python-igraph', version, {
        'modulename': 'igraph',
        'source_tmpl': '%(name)s-%(version)s.tar.gz',
        'source_urls': ['https://pypi.python.org/packages/source/p/python-igraph/'],
        'checksums': ['4601638d7d22eae7608cdf793efac75e6c039770ec4bd2cecf76378c84ce7d72'],
    }),
]
modextrapaths = {
# EBPYTHONPREFIXES directories for current python version X.Y to PYTHONPATH.
    'EBPYTHONPREFIXES': [''],
}

sanity_check_paths = {
    'files': ['lib/libigraph.%s' % x for x in [SHLIB_EXT, 'la', 'a']] + ['lib/pkgconfig/igraph.pc'] +
             ['include/igraph/igraph%s.h' % x for x in ['', '_blas', '_constants', '_lapack', '_types', '_version']],
    'dirs': [],
}

moduleclass = 'lib'
```

Two changes were made from the original which can be found
[here](https://github.com/EasyBuilders/easybuild-easyconfigs/blob/main/easybuild/easyconfigs/i/igraph/igraph-0.10.6-foss-2023a.eb):

- Changing the toolchain to `gcccoreflexiblas,2023a`. It is quite frequent that the upstream versions
of EasyBuild recipes use an over-complete toolchain, i.e. a toolchain which includes dependencies that
are not needed. In this example, the upstream recipe uses "foss" which includes OpenMPI, but igraph does
not use OpenMPI. We "downgrade" the toolchain to gcccoreflexiblas, which uses GCC and FlexiBLAS.
- Adding the section for Python extensions: 
```
multi_deps = {'Python': ['3.10', '3.11'] }
multi_deps_extensions_only = True

exts_defaultclass = 'PythonPackage'
exts_list = [
    ('python-igraph', version, {
        'modulename': 'igraph',
        'source_tmpl': '%(name)s-%(version)s.tar.gz',
        'source_urls': ['https://pypi.python.org/packages/source/p/python-igraph/'],
        'checksums': ['4601638d7d22eae7608cdf793efac75e6c039770ec4bd2cecf76378c84ce7d72'],
    }),
]
modextrapaths = {
# EBPYTHONPREFIXES directories for current python version X.Y to PYTHONPATH.
    'EBPYTHONPREFIXES': [''],
}
``` 
When it makes sense, we try to install the python bindings alongside the main
code. In this case, we install the `python-igraph` package. Whenever possible, we
also try to install it for multiple versions of python. This is done with the 
`multi_deps` option. Finally, the `EBPYTHONPREFIXES` environment variable is used by
our python configuration to find packages compatible with the version of python that
is being used.


```
eb igraph-0.10.6-gcccoreflexiblas-2023a.eb
module load igraph/0.10.6
```

Please refer to the [Checksums in EasyConfig
recipes](#checksums-in-easyconfig-recipes) section to learn how to add a
checksum to your new recipe.

This uses the file that you just changed in the current directory. Once you are
satisfied with the local build, you can then add the file to the git repository:

```
git add igraph-0.10.6-gcccoreflexiblas-2023a.eb
git pull origin 2023
git commit -m "commit message goes here"
git push origin 2023
```

The final step to install it on the build node, is to pull it and install it as
the user `ebuser`; the first command syncs the channel from GitHub:

```
sudo -iu ebuser eb-pull-cc
sudo -iu ebuser eb igraph-0.10.6-gcccoreflexiblas-2023a.eb
```

Note that if you installed the package in your own account, that version will
have priority over the globally installed version. Remove the folder
`$HOME/.local/easybuild` if you want to get rid of the version installed in your
home.

Once this is done, you must [deploy on CVMFS](cvmfs.md). Unless the software is
deployed, it will not be visible on the clusters.

In general it is best to work by example. There are thousands of easyconfig
files in the `easybuild-easyconfigs` GitHub repository.


### Checksums in EasyConfig recipes

Since April 3 2020, it is now required to have checksums in the recipes that
are installed. Checksums can be automatically calculated and injected in a
recipe using:

```
eb <recipe.eb> --inject-checksums --force
```

This modifies the file `<recipe.eb>` which must be writeable. Afterwards, you
will be able to proceed with installing the software as usual using that
modified file:

```
eb <recipe.eb>
```

If, for some reason, it is absolutely needed to disable this requirement, it can
be done with:

```
eb <recipe.eb> --disable-enforce-checksums
```

Note that if you are downloading your sources by [cloning from a git repository][eb-download-from-git],
the checksums will change every time, because files will have different timestamps.

In this case:

* First run `eb <recipe.eb> --inject-checksums --force` from within your own 
  account, which will create an archive under e.g.:
  `~/.local/easybuild/m/MyPackage/MyPackage-1.0.1.tar.gz`.
* Then manually copy that archive to the central source location: 
  `/cvmfs/soft.computecanada.ca/easybuild/sources/m/MyPackage/`.
* When building as ebuser, EasyBuild will find the sources in that location and
  skip downloading them again.

[eb-download-from-git]: https://easybuild.readthedocs.io/en/latest/Writing_easyconfig_files.html#downloading-from-a-git-repository

## Tips and tricks, troubleshooting

### Rebuilding installed software

If you want to rebuild the software that has already been installed, you can
append `--rebuild` to the `eb` command to force EasyBuild to do it again. The
same option can be used by a regular user to install the software into their
home directory on a cluster even when it has already been installed centrally.

If you want to regenerate the module file (`.lua`) only, and skip building the
software itself, you can also specify the `--module-only` option for `eb`.

### Downloading source or binaries manually

For some packages, you need to download the source or binary package manually.
One such example is Java. In order to do so, please download the file as your
user. Then copy the file to the source directory. For example, for Java, this
would be:

```
cp /tmp/jdk-8u121-linux-x64.tar.gz /cvmfs/soft.computecanada.ca/easybuild/sources/j/Java/
```

Once the source is there, you will be able to install the package.

### Fixing the loader and runpath

As a design decision, we are typically not setting `LD_LIBRARY_PATH` in software
modules. Instead, our compilers ensure that the correct Gentoo loader ("ELF
interpreter") is used by the binaries and therefore locates appropriate OS
libraries. This works well for things that we compile ourselves. However, for
software that is installed in a binary form, and for some compiled software, it
may be needed to set the correct loader using `patchelf`.

We provide a special script that can set the loader for the binaries. If you
just want to patch the loader, then run it as shown below, where `path` can be
a directory or an individual file.

```
setrpaths.sh --path <path>
```

**Note:** As the primary purpose of the script has changed from patching
*runpath* to setting the loader, its historical name `setrpath.sh` remained
the same for backward compatibility reasons.

If the interpreter used by the binary is already our local Gentoo interpreter, the
script will not patch anything as it is supposed to be working correctly from
the compilation step. Sometimes, however, the binary still needs to be patched,
and a special option `--any_interpreter` can force it:

```
setrpaths.sh --path <path> --any_interpreter
```

By default, the script will only patch the loader, but it can also add paths to
binary's *runpath* using the `--add_path` option, like so:

```
setrpaths.sh --path <path> --add_path <runpath>
```

The last two options can be combined, if needed, like so:

```
setrpaths.sh --path <path> --add_path <runpath> --any_interpreter
```

The `--add_origin` option will also add `$ORIGIN` to the binary's *runpath*.
This should not be necessary as our loader takes care of locating the libraries
in the same directory.

To run the command as a privileged `ebuser`, use `sudo`:

```
sudo -i -u ebuser setrpaths.sh
```

In an EasyBuild config, this script is usually executed as part of
`postinstallcmds`, for example:

```
postinstallcmds = [
    "/cvmfs/soft.computecanada.ca/easybuild/bin/setrpaths.sh --any_interpreter --path %(installdir)s/bin --add_path %(installdir)s/lib",
]
```

You can always search existing easyconfigs for more examples of using
`setrpaths.sh`.

### Dependencies

**`builddependencies`**: If a dependency is only needed at build time (and not
at runtime).

**`dependencies`**: If it is needed both during build and at runtime, so that
it is present in the module.

This is particularly relevant for Python which is a frequent dependency.  The
only times when Python should be a runtime dependency is when the
application/library cannot be used without Python. Otherwise, it should be a
build dependency.

Note that the names of the dependencies are the names of the corresponding
easyconfig files that are case-sensitive. This might be confusing as the
modulefile names are always lowercase. There is a way to find the correct name
of the easyconfig using the search command like so:

```
eb -S <name>
```

### Debugging your build

If your `eb` command fails with an error, you might want to identify the precise
step where the failure occurred, and then attempt to repeat that particular step
(to try different compiler flags, for example).  

To get a clear understanding of where the error occurs, it is often useful to
run the non-parallel build, so that errors are given in the output log file in
the correct order. To do this, add the option `--sequential` to your `eb` command. 

Once you identify the line in the build process that causes the error, you can
have `eb` dump the environment used during the building, source that
environment, and then navigate to the build directory (included in each `eb`
build output). For example:

```
eb igraph-0.8.2-gcccoreflexiblas-2020a.eb
(... some error happens ...)
eb igraph-0.8.2-gcccoreflexiblas-2020a.eb --dump-env-script
module --force purge
module load StdEnv/2020
source igraph-0.8.2-gcccoreflexiblas-2020a.env
cd /tmp/$USER/avx2/igraph/0.8.2/gcccoreflexiblas-2020a/igraph-0.8.2/
```

At this point you run the precise build step command which failed.

To figure out which commands were run by EasyBuild, you can use `grep` to find
instances of `run.py` in the log file: 
```
grep run.py /tmp/eb-gqqs_1w0/easybuild-wszpn04e.log
== 2021-03-15 18:45:09,532 run.py:222 INFO running cmd: type module
== 2021-03-15 18:45:26,032 run.py:222 INFO running cmd: tar xzf /home/mboisson/.local/easybuild/sources/i/igraph/igraph-0.8.2.tar.gz
== 2021-03-15 18:45:29,915 run.py:222 INFO running cmd: /home/mboisson/.local/easybuild/sources/generic/eb_v4.3.3/ConfigureMake/config.guess
== 2021-03-15 18:45:29,960 run.py:538 INFO cmd "/home/mboisson/.local/easybuild/sources/generic/eb_v4.3.3/ConfigureMake/config.guess" exited with exit code 0 and output:
== 2021-03-15 18:45:29,961 run.py:222 INFO running cmd: autoreconf -i && sed -i 's/-lblas/$LIBBLAS/' configure && sed -i 's/-llapack/$LIBLAPACK/' configure &&  ./configure --prefix=/home/mboisson/.local/easybuild/software/2020/avx2/Core/igraph/0.8.2  --build=x86_64-pc-linux-gnu  --host=x86_64-pc-linux-gnu --with-external-blas --with-external-lapack --with-external-glpk
``` 

### Providing the source distribution

It may be necessary for some software to provide the source distribution along
with the compiled binary distribution. Usually, this is to allow users to build
additional custom components on their own. To accomplish this, there is an
option `buildininstalldir` (see
[documentation](https://easybuild.readthedocs.io/en/latest/version-specific/easyconfig_parameters.html))
to build in the installation directory. You may also need to specify additional
options for unpacking the tarball:

```
unpack_options = '--strip-components=1'
buildininstalldir = True
```

For more examples, take a look at the existing recipes:

```
git grep buildininstalldir
```

### Other topics to be covered in the future

- Some kind of an hello world example w/interdependent libraries in the example.
- What toolchain should you use ?
- Toolchain `opts`, `openmp: true`, `usempi: true`; What do they do?
- Dependency vs toolchain (for example OpenBLAS)
- Dependency not found due to capitalization
- `installopts`, `buildopts`, `skipsteps`

## Contributing back to EasyBuild

**Note:** This is considered an advanced topic. Feel free to skip it.

In order to contribute back to EasyBuild, what must be done is called a pull
request. Pull requests are to be done against the `develop` branch of the
easybuilders repositories. In order to be able to do so, you first need to retrieve
that branch. This is done by running the following from your local repository.
We are assuming here that you want to create a pull request on the
`easybuild-easyconfigs` repository. The same would apply for
`easybuild-easyblocks` or `easybuild-framework`.

```
git remote add upstream git@github.com:easybuilders/easybuild-easyconfigs.git
git fetch upstream develop
```

The first command adds a second remote repository to your local repository. This
will allow you to pull changes from the easybuilders version as well as the Compute
Canada version of the repository. The second command retrieves the commits from
the upstream repository, without merging them with your local
version.

The second step is to create a branch that will merge well both into the
`upstream/develop` branch, and into the `origin/computecanada-main` branch. To
do so, you run:

```
  git checkout -b mybranch $(git merge-base upstream/develop origin/computecanada-main)
```

This will create a local branch called `mybranch` that spawns from the latest
common intersection of the `upstream/develop` and the
`origin/computecanada-main` branch. The reason why we do this is that this is
the simplest way to ensure that `mybranch` will be mergeable easily into the two
branches.

Then, start doing your changes. If you had already done local changes before
doing the commands above, you can run `git stash` before doing the checkout, and
`git stash pop` after. This will save your changes in a temporary stash, and
restore them in the new branch.

You can then develop, make as many commits as needed, and push them to our
repository using:

```
git push origin mybranch
```

This will create (or update) a branch named `mybranch` in our GitHub.

Whenever you want to perform an installation on our build node, you then first
want to run:

```
git checkout computecanada-main
git merge mybranch
```

And optionally:

```
git push origin computecanada-main
```

If you want to perform an installation as `ebuser`.

Once you are ready to create the pull request, you will go on GitHub to this
page: `https://github.com/ComputeCanada/easybuild-easyconfigs/tree/mybranch`.

And click on "New pull request". On the left side (`easybuilders` side), make sure
you choose `base:develop` for the branch. On the right side (`computecanada`
side), make sure you select your custom branch.

It should show you the list of commits and changes that are included in the pull
request. Examine the list to confirm that it includes only what you want. If so,
create the pull request.

From this point on, every time you push the `mybranch` branch, the new commit
will be added to the same pull request. This will be the case until the pull
request is merged on `easybuilders` side. Note that the EasyBuild developers are
likely to request various changes, including coding style changes. They also
will not merge the request unless their automated test suite can run on it. This
means that the easyconfig must use one of their main toolchains.

Once the pull request is merge, you can make a final merge of `mybranch` into
the `computecanada-main` branch, and then delete your branch with:

```
git push origin -delete mybranch
git branch -d mybranch
```

## Installing restricted software

**Note:** This is considered an advanced topic. Feel free to skip it.

### Different cases

There are various kinds of restrictions that can exist on a software:

1. In some cases, there is no restriction other than being able to access a
   license server that is provided by the user (bring your own license model).
2. In some cases, such as the Intel compilers, the license is valid for
   everyone, but for non-commercial usage only.
3. In some cases, such as NAMD, we need to ask users to get a license, but it is
   free, and the authors are not very strict about it.
4. Finally, in the more restrictive cases, such as VASP, we have to check with
   the company before granting access to users.

We deal with the four cases slightly differently. In all cases however, we aim
at having the modules themselves publicly visible, but inform users of what they
need to do when they load the module. For all but the POSIX group case,
installation of the software is performed as usual. For the POSIX group case,
see below.

### Informing users of restrictions at module load time

The file `/cvmfs/soft.computecanada.ca/config/lmod/SitePackage.lua` contains
code that can be used to configure various messages that are shown to users when
they load a module. After this file is modified, the `config` folder should be
synchronized to CVMFS.

We specify the type of restriction in the `validate_license` function, in the
`licenseT`. This table has a key which consists of a list of software, with a
value which corresponds to the restriction imposed on that software.

In the above case 1 (access controlled by a license server with no other
restriction), the only thing to be careful about is to install the software in
the restricted repository. This is achieved by using the group `rsnt_soft` to
install, with:

```
  sudo -i -g rsnt_soft -u ebuser eb <recipe>
```

In addition, one can add code similar to what is shown below to the module. This
will search for a license file in the user’s home directory, and give
instructions to the user if no license file is found. Such code can be added to
the module by using the `modluafooter` parameter in the EasyConfig file.

```
require(“SitePackage”)

local found = find_and_define_license_file(“MLM_LICENSE_FILE”,“matlab”)

if (not found) then
        local error_message = [[
        We did not find a suitable license for Matlab. If you have access to one, you can create the file $HOME/.licenses/matlab.lic with the license information. If you think you should have access to one as    part of your institution, please write to support@tech.alliancecan.ca that we can configure it.

        Nous n’avons pas trouve de licence utilisable pour Matlab. Si vous avez acces a une licence de Matlab, vous pouvez creer le fichier $HOME/.licenses/matlab.lic avec l’information de la licence. Si vous    pensez que vous devriez automatiquement avoir acces a une licence via votre institution, veuillez ecrire a support@tech.alliancecan.ca pour que nous puissions la configurer.
        ]]
        LmodError(error_message)
end
```

In the above case 2 (non-commercial use only), add the name of the software in
the list for the `noncommercial_autoaccept` type of restriction in the
`licenseT` table.  This will display a message to the user the first time they
load the module. It will record that it was already displayed in a file in
`$HOME/.licenses/`.

In the above case 3 (license required, but authors not strict about POSIX
restrictions), add the name of the software in the list for the
`academic_license` type of restriction in the `licenseT` table. Also add the URL
to the license page on the software website to the `licenseURLT` table.  This
will display a message to the user the first time they load the module. It will
ask users whether they have a license for this software. It will record the
answer in a file in `$HOME/.licenses/`.

For the above case 4 (license required, and authors strict about POSIX
restrictions), add the name of the software in the list for the `posix_group`
type of restriction in the `licenseT` table. Also add the POSIX group needed in
the `groupT` table. This will test whether the user is part of this group or
not, and if not, refuse to load the module and instruct the user to write to
support@tech.alliancecan.ca to get added to the POSIX group.

### Installing the software

Software with restrictions should not be installed under the default path
`/cvmfs/soft.computecanada.ca/easybuild`.

Instead, it should be installed under
`/cvmfs/restricted.computecanada.ca/easybuild`. However, we want the module of
said software to end up in the central module tree. To do so, use this command:

```
sudo -i -g <group for the software> -u ebuser eb <recipe>
```

If it is restricted, but not by POSIX group, you can use `rsnt_soft` as the
group.

If it is restricted by POSIX group, you should also use the following option in
the EasyConfig file (the `os.environ.get` method requires `import os` in the
config file):

```
if os.environ.get('USER') == 'ebuser':
    group = <group for the software>
```

Software is then installed under
`/cvmfs/restricted.computecanada.ca/easybuild/software`, and sources are
downloaded to `/cvmfs/restricted.computecanada.ca/easybuild/sources` but the
modules are installed in the usual place under
`/cvmfs/soft.computecanada.ca/easybuild/modules`.

This will ensure that EasyBuild changes the group owner of the installed
files. `ebuser` must be part of the POSIX group to be able to do so. This
membership can be changed through the CCDB.

### Deploying POSIX group-restricted software with CVMFS

**See:** [Deploying to the restricted repository with CVMFS](cvmfs.md#deploying-to-the-restricted-repository-with-cvmfs)

### Hiding POSIX group-restricted software on systems that don’t have them

Some systems may have access to the public software stack, but not to the
restricted repository. In this case, we want to hide the modules for the
restricted pieces of software. This is done in the file
[/cvmfs/soft.computecanada.ca/config/lmod/SitePackage_visible.lua](https://github.com/ComputeCanada/software-stack-config/blob/main/lmod/SitePackage_visible.lua)

### Existing POSIX groups to manage access to pieces of software

Existing POSIX groups to manage access to some software (CC staff links):

- [soft_cpmd](https://ccdb.alliancecan.ca/services/soft_cpmd)
- [soft_vasp5](https://ccdb.alliancecan.ca/services/soft_vasp5)
- [soft_vasp4](https://ccdb.alliancecan.ca/services/soft_vasp4)
- [soft_dl_poly4](https://ccdb.alliancecan.ca/services/soft_dl_poly4)
- [soft_orca](https://ccdb.alliancecan.ca/services/soft_orca)
- [soft_gaussian](https://ccdb.alliancecan.ca/services/soft_gaussian)

## Installing datasets
In rare occasions, a piece of software requires large datasets to be installed. If these are larger than 50GB or
if they contain many files larger than 1GB, they should be installed in the `public.data` repository. Interproscan is
such an example. Part of the software is a dataset named `panther`. It can be installed as such: 
```
eb panther-14.1-system.eb --installpath-software=/cvmfs/public.data.computecanada.ca/content/easybuild/data/2020
``` 
This will install the data inside of `/cvmfs/public.data.computecanada.ca/content/easybuild/data/2020`, while the module will
be installed under `/cvmfs/soft.computecanada.ca/easybuild/modules`. Synchronizing to CVMFS is then performed in the same way
as for restricted software, by starting transactions on two different repositories. 

### Deploying datasets with CVMFS

**See:** [Deploying datasets with CVMFS](cvmfs.md#deploying-datasets-with-cvmfs)



## FAQ

**Q:** Is there any way we can find who built a module?

**A:** From `build-nodes.alliancecan.ca`, you can type `who_installed.sh
<module name>`.
