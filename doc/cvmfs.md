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

There are four repositories to which we have access:

- **dev:** `/cvmfs/soft-dev.computecanada.ca`
- **prod:** `/cvmfs/soft.computecanada.ca`
- **restricted:** `/cvmfs/restricted.computecanada.ca`
- **data:** `/cvmfs/data.rsnt.computecanada.ca`

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

On `publisher-node.computecanada.ca`, switch to the user `libuser`:

```
sudo su - libuser
```

As `libuser`, start a transaction:

```
sudo /etc/rsnt/start_transaction <dev|prod|restricted|data>
```

Then synchronize the files needed. You can sync manually, but to avoid most
mistakes, a wrapper script is provided to make it easier. Typically, you will
want to run one of the commands below, with the most common at the top:

```
# Note: <software name> must be lowercase module name, regardless of what the recipe is named ...
/etc/rsnt/rsnt-sync --what easybuild --software <software name> --version <software version>
/etc/rsnt/rsnt-sync --what gentoo
/etc/rsnt/rsnt-sync --what custom --path <path>
/etc/rsnt/rsnt-sync --what config
/etc/rsnt/rsnt-sync --what easybuild-recipes
```

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

exit
```

### Deploying to the restricted repository with CVMFS
Software installed in the restricted repository have binaries in the restricted
repository, and modules in the public repository. Because of that,  you must start two
transactions. One to `dev` or `prod` repository for the module, and one to the
`restricted` repository. Here is an example:

```
sudo su - libuser

sudo /etc/rsnt/start_transaction <dev|prod>
sudo /etc/rsnt/start_transaction restricted

/etc/rsnt/rsnt-sync --what easybuild --software <software name> --version <software version>

sudo /etc/rsnt/publish_transaction restricted
sudo /etc/rsnt/publish_transaction <dev|prod>

exit
```

### Deploying datasets with CVMFS
Similarly to restricted software, datasets have data sitting in the `data.rsnt` repository
and modules in the public repository. Because of that,  you must start two
transactions. One to `dev` or `prod` repository for the module, and one to the
`data` repository. Here is an example:

```
sudo su - libuser

sudo /etc/rsnt/start_transaction <dev|prod>
sudo /etc/rsnt/start_transaction data

/etc/rsnt/rsnt-sync --what easybuild --software <software name> --version <software version>

sudo /etc/rsnt/publish_transaction data
sudo /etc/rsnt/publish_transaction <dev|prod>

exit
```

## Advanced options to the `rsnt-sync` script

The `rsnt-sync` script accepts many options. The most current information about
them is obtained by running:

```
/etc/rsnt/rsnt-sync --help
```

As of March 15th, 2021, the help reads as: 

```
Usage: /etc/rsnt/rsnt-sync --what <nix|nix-store|gentoo|custom|config|easybuild|easybuild-recipes> [--repo <dev|prod|restricted|data>] [--path <source path>] [--dry-run] [--full] [--no-size-only] [--no-ignore-times] [--profile <profile>] [--software <software name> [--year <2017|2020>] [--version <software version] [--architecture <software architecture>] [--root_path <software root path>]] [[--modules-only]] [[--exclude <PATTERN>]] [[--no-delete]]

  If synchronizing with "--what nix", "--what nix-store", "--what gentoo", "--what custom", "--what gentoo", or "--what config", no path is required.
  If synchronizing with "--what easybuild", you must provide the path to the folder that is to be synchronized.
  The path provided is relative to /cvmfs/soft.computecanada.ca/easybuild
  If synchronizing with "--what custom", you must provide the path to the folder that is to be synchronized.
  The path provided is relative to /cvmfs/soft.computecanada.ca/custom

  If no repository is provided, the synchronization will detect what repository is currently in a transaction and use this one.
  Make sure that you have first started a transaction to this repository.
  When synchronizing to the dev repository, the source folder will be the local /cvmfs.
  When synchronizing to the prod repository, the source folder is the dev repository.
  The option --full will synchronize all of either easybuild or nix. It should only be used if a full sync is needed to correct a problem after a failed sync
  The option --no-size-only disables the flag --size-only which is enabled by default
  The option --no-ignore-times disables the flag --ignore-times which is enabled by default
  The option --profile allows to specify a specific nix profile to be synchronized. If not specified, all current versions of profiles are synchronized, and outdated software versions are deleted. Not available for synchronizing to production.
  The option --software can be used to specify to synchronize a specific EasyBuild-installed software. Unless --version is specified, all versions of the software will be synchronized.
  The option --version can be used to synchronize only a specific version of a software to synchronize.
  The option --architecture can be used to synchronize a given software only for a specific architecture.
  The option --root_path can be used to synchronize a given software only within a specific root path (for example Cuda, Compilers or MPI).
  The option --modules-only will synchronize only modules and will ignore software.
  The option --software-only will synchronize only software and will ignore modules.
  The option --exclude <PATTERN> will exclude a pattern from the rsync command.
  The option --no-delete removes the --delete flag from rsync
  The option --year <2017|2020> will allow to select only software under the 2017 or 2020 subfolder of EasyBuild
```

Most of these options are useful only for synchronizing something installed via
EasyBuild. The following are designed to be used with `--software <software
name>`.

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

- To synchronize some configuration files (in the subdirectory
  `soft.computecanada.ca/config`), use:

```
/etc/rsnt/rsnt-sync --what config
```

- To synchronize every Python wheels, use:

```
/etc/rsnt/rsnt-sync --what custom --path python/wheelhouse
```

- To synchronize specific Python wheels, use:

```
/etc/rsnt/rsnt-sync --what custom --path python/wheelhouse --software numpy --version 1.18
```

