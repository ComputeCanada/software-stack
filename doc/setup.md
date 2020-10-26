# Initial setup

**Home:** [Software management](INDEX.md)

### Contents

- [Git configuration](#git-configuration)
- [EasyBuild repositories](#easybuild-repositories)
- [Nix repository](#nix-repository)

## Before you begin
As a Compute Canada staff, ensure that you have requested the proper permissions, and configured your account, as described on
https://wiki.computecanada.ca/staff/RSNT_Software_-_Setting_up

## Git configuration

If you plan to contribute to the software stack, make sure to configure your
name and email. To do this globally:

```
git config --global user.name "John Doe"
git config --global user.email johndoe@example.com
```

## EasyBuild repositories

EasyBuild is split into three repositories that reside in the [CC
GitHub](https://github.com/ComputeCanada/):

- `easybuild-easyconfigs`
- `easybuild-easyblocks`
- `easybuild-framework`

In 90% of cases, you will only need the first one, and in 99% of cases, you will
only need the first two. We recommend getting only the first one to start.

If you are installing a software package with EasyBuild, you will need to have a
clone of the `easybuild-easyconfigs` repository in your account on
`build-node.computecanada.ca`. To do that, run the commands:

```
cd
git clone https://github.com/ComputeCanada/easybuild-easyconfigs.git
```

In some rare cases, you may also need to edit EasyBlocks. For that, you would need the `easybuild-easyblocks` repository:

```
cd
git clone https://github.com/ComputeCanada/easybuild-easyblocks.git
```

In extremely rare cases, you might need to have the `easybuild-framework`
repository. This would only happen if you need to edit the core of EasyBuild
itself. You would get it by running:

```
cd
git clone https://github.com/ComputeCanada/easybuild-framework.git
```

If you are planning on making changes (such as [creating and changing EasyBuild
recipes](easybuild.md#creating-or-changing-a-recipe)), make sure you install
your SSH public key into your GitHub account and also verify that `.git/config`
contains the proper `url` setting like shown below:

```
url = git@github.com:ComputeCanada/easybuild-easyconfigs.git
```

## Nix repository

Software installers rarely need to use Nix. However, should this be your case,
you can clone our fork of the `nixpkgs` repository using:

```
cd
git clone https://github.com/ComputeCanada/nixpkgs.git
```
