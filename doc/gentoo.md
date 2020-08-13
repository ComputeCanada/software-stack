# Installing with Gentoo

**Home:** [Software management](INDEX.md)

**Note:** Gentoo is more of an advanced topic. Most of the times, you will
[install software with EasyBuild](easybuild.md).

**Note:** You must always first `module load StdEnv/2020` before using Gentoo
commands.

The following contain basic commands that you may need for using Gentoo Prefix
for the CC central stack. For a much more detailed Gentoo guide, see [a basic
guide to write Gentoo
Ebuilds](https://wiki.gentoo.org/wiki/Basic_guide_to_write_Gentoo_Ebuilds). See
also the [Gentoo Cheat Sheet](https://wiki.gentoo.org/wiki/Gentoo_Cheat_Sheet).

### Contents

- [Searching for packages in Gentoo](#searching-for-packages-in-gentoo)
- [Installing a package in Gentoo](#installing-a-package-in-gentoo)
  - [Existing recipes](#existing-recipes)
  - [Creating a new recipe](#creating-a-new-recipe)

## Searching for packages in Gentoo

You can search for packages (existing, installed or not) using this command:

```
emerge -s [package name]
```

E.g.:

```
emerge -s ".*readline.*"
```

The list of all installed Gentoo-packages can be viewed with:

```
equery list "*"
```

## Installing a package in Gentoo

### Existing recipes

If a package already has an existing recipe, you can install it easily using the
following command:

```
sudo -i -u gentoouser emerge <package name> [--pretend]
```

This will sometimes ask you to make modifications to configuration files, but
generate temporary files first. You then make the modifications using

```
sudo -u gentoouser -i etc-update
```

You can use `--pretend` without `sudo` as well, as it will only do a dry run.
This will install the stable version: if the stable version is too old for some
reason you need to unmask the "testing" version by adding it to
`$EPREFIX/etc/portage/package.accept_keywords`, e.g. `app-misc/tmux ~amd64`
unmasked `tmux-3.1b`.

The ebuilds themselves can be found under `$EPREFIX/var/db/repos/gentoo/`
(upstream, frozen) and `$EPREFIX/var/db/repos/computecanada` (ours, updated via
`sudo -iu gentoouser emerge --sync` from the [CC
overlay](https://github.com/ComputeCanada/gentoo-overlay).

### Creating a new recipe

If a package does not exist, you can create a new recipe. Before creating a new
recipe, however, you should ask yourself if the package should be installed in
Gentoo or in EasyBuild. Often, packages not available in Gentoo will be
available in EasyBuild.

To add a new package to Gentoo you will first need to create the package. As an
example, we created this one for `opa-psm2` (OmniPath libraries):
https://github.com/ComputeCanada/gentoo-overlay/blob/master/sys-fabric/opa-psm2/opa-psm2-11.2.86.ebuild.
This file is as follows:

```
EAPI=7

inherit udev

DESCRIPTION="OpenIB userspace driver for the PathScale InfiniBand HCAs"
SRC_URI="https://github.com/intel/${PN}/archive/PSM2_${PV}.tar.gz -> {P}.tar.gz"

SLOT="0"
KEYWORDS="amd64 ~x86 ~amd64-linux"
IUSE=""

DEPEND="virtual/pkgconfig"
RDEPEND="${DEPEND}
	sys-apps/util-linux
	sys-process/numactl
	virtual/udev"

S="${WORKDIR}/${PN}-PSM2_${PV}"

src_compile() {
	emake arch=x86_64 USE_PSM_UUID=1 WERROR=
}

src_install() {
	emake arch=x86_64 UDEVDIR="/lib/udev" DESTDIR="${D}/${EPREFIX}" install
	dodoc README
}
```

To enable this we clone or update the Compute Canada `gentoo-overlay` on GitHub:

```
cd gentoo-overlay
git pull
```

The `opa-psm2` package can then be test-built using this syntax:

```
repoman manifest # in the same directory as the ebuild
mkdir -p ~/.local/gentoo
rm -rf ~/.local/gentoo/*
PORTAGE_USERNAME=$USER PORTAGE_GRPNAME=$USER PORTAGE_TMPDIR=$HOME/.local/gentoo ebuild opa-psm2-11.2.86.ebuild install
```

(This command builds and installs under `~/.local/gentoo`.)

Once this is done, you should add your new recipe to our repository, using the
following:

```
cd gentoo-overlay/sys-fabric/opa-psm2
git add Manifest opa-psm2-11.2.86.ebuild

# (if not already done so, set name/email)
git config --global user.name “John Doe”
git config --global user.email johndoe@example.com
git pull
git commit
git push
```

The final step is to move this into “build-node” production; the first command
syncs the channel from github:

```
sudo -u gentoouser -i emerge --sync
sudo -u gentoouser -i emerge sys-fabric/opa-psm2
```

In general it is best to work by example. There are thousands of ebuilds under
`$EPREFIX/var/db/repos`.
