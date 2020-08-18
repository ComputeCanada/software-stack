# Deploying software with CVMFS

**Home:** [Software management](INDEX.md)

### Contents

- [Introduction](#introduction)
- [Development vs production repositories](#development-vs-production-repositories)
- [How to deploy software on CVMFS](#how-to-deploy-software-on-cvmfs)
  - [Example](#example)
- [Advanced options to the `rsnt-sync` script](#advanced-options-to-the-rsnt-sync-script)
- [Examples of `rsnt-sync` commands](#examples-of-rsnt-sync-commands)

## Introduction

This page covers only essential information about using CVMFS with our
infrastructure and software stack. For complete CVMFS documentation, see the
[official website](https://cvmfs.readthedocs.io/en/stable/).

There are three repositories to which we have access:

- **dev:** `/cvmfs/soft-dev.computecanada.ca`
- **prod:** `/cvmfs/soft.computecanada.ca`
- **restricted:** `/cvmfs/restricted.computecanada.ca`

You can use the dev version to push files and see that synchronization happens
correctly. However, pushing installed software to the dev repository will not
work transparently, because the installed software is not relocatable.

## Development vs production repositories

A new piece of software must always go first to the development repository
before going to the production repository. Synchronization always happens:

- From the build node to the development repository
- From the development repository to the production repository

The idea is that you first test your newly deployed software on the
`cvmfs-client-dev.computecanada.ca` client before deploying it to the production
repository. This is to ensure that you do not forget to synchronize some files,
and to ensure that the piece of software actually works on a different
environment than the build node.

## How to deploy software on CVMFS

**See also:** [Deploying restricted
software](easybuild.md#deploying-posix-group-restricted-software-with-cvmfs)

Switch to the user `libuser`:

```
sudo su - libuser
```

As `libuser`, start a transaction:

```
sudo /etc/rsnt/start_transaction <dev|prod|restricted>
```

Then synchronize the files needed. You can sync manually, but to avoid most
mistakes, a wrapper script is provided to make it easier. Typically, you will
want to run one of the commands below, with the most common at the top:

```
# Note: <software name> must be lowercase module name, regardless of what the recipe is named ...
/etc/rsnt/rsnt-sync --what easybuild --software <software name> --version <software version>
/etc/rsnt/rsnt-sync --what nix
/etc/rsnt/rsnt-sync --what custom --path <path>
/etc/rsnt/rsnt-sync --what config
/etc/rsnt/rsnt-sync --what easybuild-recipes
```

Note: the option `custom-python` exists as a backward compatibility to
synchronize all python wheels.

For more advanced options, see the section below.

**If you make a mistake, don’t panic, simply run:**

```
sudo /etc/rsnt/abort_transaction <dev|prod|restricted>
exit
```

This will revert the changes and you can start over.

Check that the changes rsynced match what you expect:

```
ls /stratum0/cvmfs/soft.computecanada.ca/<sub path>
```

or:

```
ls /stratum0/cvmfs/soft-dev.computecanada.ca/<sub path>
```

or:

```
ls /stratum0/cvmfs/restricted.computecanada.ca/<sub path>
```

Once you are confident that the changes in
`/stratum0/cvmfs/{soft-dev,soft,restricted}.computecanada.ca/` represent what
needed to be done, run:

```
sudo /etc/rsnt/publish_transaction <dev|prod|restricted>
exit
```

Note that changes will propagate to the clusters within 30 minutes.

### Example

Here is a concrete example how to deploy deal.II 8.4.2:

```
sudo su - libuser

sudo /etc/rsnt/start_transaction dev
/etc/rsnt/rsnt-sync --what easybuild --software dealii --version 8.4.2
sudo /etc/rsnt/publish_transaction dev

sudo /etc/rsnt/start_transaction prod
/etc/rsnt/rsnt-sync --what easybuild --software dealii --version 8.4.2
sudo /etc/rsnt/publish_transaction prod
```

## Advanced options to the `rsnt-sync` script

The `rsnt-sync` script accepts many options. The most current information about
them is obtained by running:

```
/etc/rsnt/rsnt-sync --help
```

As of Sept. 6th, 2017, the options are:

```
/etc/rsnt/rsnt-sync --what <nix|config|easybuild|custom> \
  [--software <software name>] \
  [--version <software version>] \
  [--architecture <architecture>] \
  [--root_path <software root path>] \
  [--modules-only] [--software-only] [--no-size-only] \
  [--repo <dev|prod>] \
  [--path <source path>]
```

Most of these options are useful only for synchronizing something installed via
EasyBuild. The following are designed to be used with `--software <software
name>`:

- `--version <software version>`: will synchronize only a specific version
- `--architecture <architecture>`: will synchronize only for a specific
  architecture
- `--root_path <software root path>`: will synchronize only software installed
  within a specific root path below the `easybuild/{software,modules}` folders.
- `--module-only`: will skip synchronizing the `software` folder
- `--software-only`: will skip synchronizing the `modules` folder

The option `--repo` allows you to specify a repository to synchronize to.
Typically, this is automatically detected (provided that you have a transaction
opened only on one repository).

The option `--path` allows you to synchronize a specific path only. The path
specified is relative to the repository (i.e. relative to
`/cvmfs/soft.computecanada.ca/easybuild`) and should NOT include a trailing
slash.

## Examples of `rsnt-sync` commands

- To synchronize all versions of GROMACS, for all architectures:

```
/etc/rsnt/rsnt-sync --what easybuild --software gromacs
```

- To synchronize all builds of GROMACS 5.1.4 for all architectures:

```
/etc/rsnt/rsnt-sync --what easybuild --software gromacs --version 5.1.4
```

- To synchronize all builds of GROMACS 5.1.4 for architecture `avx`:

```
/etc/rsnt/rsnt-sync --what easybuild --software gromacs --version 5.1.4 --architecture avx
```

- To synchronize all builds of GROMACS 5.1.4 for architecture `avx2` under the
  `/cvmfs/soft.computecanada.ca/easybuild/{software,modules}/MPI/intel2016.4/openmpi2.0.2`
  path:

```
/etc/rsnt/rsnt-sync --what easybuild --software gromacs --version 5.1.4 --architecture avx2 --root_path MPI/intel2016.4/openmpi2.0.2
```

- To synchronize the path
  `/cvmfs/soft.computecanada.ca/easybuild/software/2017/avx2/MPI/intel2016.4/openmpi2.0/boost`,
  you would run:

```
/etc/rsnt/rsnt-sync --what easybuild --path software/2017/avx2/MPI/intel2016.4/openmpi2.0/boost
```

You would then need to synchronize the corresponding path in the `modules`
subdirectory to synchronize the module as well as the software:

```
/etc/rsnt/rsnt-sync --what easybuild --path modules/2017/avx2/MPI/intel2016.4/openmpi2.0/boost
```

- To synchronize some configuration files (in the subdirectory
  `soft.computecanada.ca/config`), use:

```
/etc/rsnt/rsnt-sync --what config
```

- To synchronize any Python wheels, use:

```
/etc/rsnt/rsnt-sync --what custom --path python/wheelhouse
```

- To synchronize specific Python wheels, use:

```
/etc/rsnt/rsnt-sync --what custom --path python/wheelhouse --software numpy --version 1.18
```
