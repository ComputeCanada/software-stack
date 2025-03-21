# Software management

**Note:** Staff members should first read our internal [Software
Installation](https://wiki.alliancecan.ca/wiki/Software_installation)
documentation to learn how to request privileged access, set up their account,
process software installation tickets, etc.

### Contents

- [Introduction](#introduction)
- [Initial setup](#initial-setup)
- [Choosing an installation method](#choosing-an-installation-method)
  - [Gentoo or EasyBuild?](#gentoo-or-easybuild)
  - [Python packages](#python-packages)
- [Installing software](#installing-software)
  - [With EasyBuild](#with-easybuild)
  - [With Gentoo](#with-gentoo)
  - [Python packages](#python-packages)
- [Deploying software](#deploying-software)
- [Testing software](#testing-software)
- [Examples](#examples)

## Introduction

The software available to Alliance users, including scientific and
research software, is maintained in a single central software stack accessible
from all Alliance clusters. This presents users with a consistent
experience across all current and future Alliance sites.

A variety of tools are used to achieve this. [Gentoo](https://www.gentoo.org/) 
provide an underlying layer containing the
basic tools and commands usually provided by Linux operating systems.
[EasyBuild](https://github.com/hpcugent/easybuild) is used to build scientific
and research software, parallel and numerical libraries, and other
performance-critical software.
[Lmod](https://www.tacc.utexas.edu/research-development/tacc-projects/lmod)
makes software packages available as modules that users can selectively add to
their environment. Custom [Python Wheels](http://pythonwheels.com/) provide
Python packages optimised for the Compute Canada software stack. Finally,
[CVMFS](https://cvmfs.readthedocs.io/en/stable/) distributes the software stack,
making usable on all Compute Canada clusters and also publicly accessible.

## Initial setup

Before you can install software, you must [set up](setup.md) the Compute Canada
software stack repositories.

## Choosing an installation method

### Gentoo or EasyBuild?

Software in Gentoo is architecture- and platform-independent and software which
would usually be provided by the OS.

The following guidelines should be followed, although exceptions may arise.

1. Is the software performance critical ?
   - Yes => [EasyBuild](easybuild.md)
2. Do you expect that multiple versions will be needed, swappable through
   modules?
   - Yes => [EasyBuild](easybuild.md)
   - No => [Gentoo](gentoo.md) => please contact the RSNT team first

The current list of installed (via `emerge`) Gentoo packages is tracked in the
`world` file (`/cvmfs/soft.computecanada.ca/gentoo/2020/var/lib/portage/world`,
which is also copied
[here](https://github.com/ComputeCanada/gentoo-overlay/blob/main/etc/portage/world);
the complete list including dependencies can be obtained using `equery list
"*"`.

### Python packages

With very few exceptions, [installing Python packages](python.md) is done
through wheels rather than modules.

## Installing software

### With EasyBuild

See: [Installing with EasyBuild](easybuild.md)

### With Gentoo

See: [Installing with Gentoo](gentoo.md)

### Python packages

See: [Installing Python packages](python.md)

## Deploying software

Centrally-installed software is made available by [deploying it with
CVMFS](cvmfs.md).

## Testing software

Several methods are available to [test software](testing.md) at various steps of
the installation process before it is made available to users.

## Examples

- [ANSYS software installation procedure](examples/ansys.md)
