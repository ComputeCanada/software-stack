# Installing with Nix

**Home:** [Software management](INDEX.md)

**Note:** Nix is more of an advanced topic. Most of the times, you will [install
software with EasyBuild](easybuild.md).

The following contain basic commands that you may need for using Nix for the CC
central stack. For detailed documentation, see the [Nix
manual](https://www.google.com/url?q=http://nixos.org/nix/manual/&sa=D&ust=1524255168488000).

### Contents

- [Searching for packages in Nix](#searching-for-packages-in-nix)
- [Installing a package in Nix](#installing-a-package-in-nix)
  - [Existing recipes](#existing-recipes)
  - [Creating a new recipe](#creating-a-new-recipe)

## Searching for packages in Nix

You can search for packages (existing, installed or not) using this command:

```
nix-env -qasPp $NIXUSER_PROFILE [package name]
```

E.g.:

```
nix-env -qasPp $NIXUSER_PROFILE ".*readline.*"
```

## Installing a package in Nix

### Existing recipes

If a package already has an existing recipe, you can install it easily using the
following command:

```
nix-env -iA <package attribute name> [--dry-run]
```

Or:

```
sudo -i -u nixuser nix-env -iA <package attribute name>
```

Without the `sudo -i -u nixuser` the package will be installed in
`$HOME/.nix-profile`, a symbolic link to `~/nix-profile/profile`. This is useful
for testing. The package can then be deleted from `~/.nix-profile` using
`nix-env -e <package name>`.

You can also use the following command. However, note that if there are multiple
recipes for different versions of the same package, this does not control which
one is installed.

```
sudo -i -u nixuser nix-env -i <package name> [--dry-run]
```

If you are not sure what recipes may do, you can see in this list:
https://github.com/NixOS/nixpkgs/blob/master/pkgs/top-level/all-packages.nix.

As well as here for details about a specific recipe:
https://github.com/NixOS/nixpkgs/blob/master/pkgs/.

### Creating a new recipe

If a package does not exist, you can create a new recipe. Before creating a new
recipe, however, you should ask yourself if the package should be installed in
Nix or in EasyBuild. Often, packages not available in Nix will be available in
EasyBuild.

To add a new package to Nix you will first need to create the package. As an
example, we created this one for Lmod:
https://github.com/ComputeCanada/nixpkgs/blob/computecanada-16.09/pkgs/tools/misc/lmod/default.nix.

File `lmod/default.nix` is as follows:

```
  { stdenv, fetchurl, perl, tcl, lua, luafilesystem, luaposix, rsync, procps }:
  stdenv.mkDerivation rec {
    name = “Lmod-${version}”;
    version = “7.0.5”;
    src = fetchurl {
    url = “http://github.com/TACC/Lmod/archive/${version}.tar.gz”;
    sha256 = “780aa83658c2b2787337258b66ff370c27f8388fac9eccd6c1daa6f0b599599e”;
  };

  buildInputs = [ lua tcl perl rsync procps ];
  propagatedBuildInputs = [ luaposix luafilesystem ];
  preConfigure = ‘’ makeFlags=“PREFIX=$out”’’;
  configureFlags = [ “-with-duplicatePaths=yes -with-caseIndependentSorting=yes -with-redirect=yes -with-module-root-path=/cvmfs/soft.computecanada.ca/easybuild/generic/modules” ];

  LUA_PATH=“${luaposix}/share/lua/5.2/?.lua;${luaposix}/share/lua/5.2/?/init.lua;;”;
  LUA_CPATH=“${luafilesystem}/lib/lua/5.2/?.so;${luaposix}/lib/lua/5.2/?.so;;”;

  meta = {
    description = “Tool for configuring environments”;
  };
}
```

To enable this we clone or update the Compute Canada `nixpkgs` branch on GitHub:

```
cd nixpkgs
git pull
git checkout computecanada-16.09
```

You then need to add a line to `pkgs/top-level/all-packages.nix`. Edit this file
as yourself. The `lmod` package can then be built using this syntax

```
nix-env -f ~/nixpkgs -iA lmod
```

(This command builds and adds the symbolic links to `$HOME/.nix-profile`, which
is a symbolic link to `$HOME/nix-profile/profile`), or

```
nix-build ~/nixpkgs -A lmod
```

(This explicitly builds but does not install.) Once this is done, you should add
your new recipe to our repository, using the following:

```
cd ~/nixpkgs
git add pkgs/top-level/all-packages.nix
git add pkgs/tools/misc/lmod/default.nix

# (if not already done so, set name/email)
git config --global user.name“John Doe”
git config --global user.email johndoe@example.com
git pull origin computecanada-16.09
git commit
git push origin computecanada-16.09
```

The final step is to move this into “build-node” production; the first command
syncs the channel from GitHub:

```
sudo -u nixuser -i nix-pull-cc
sudo -u nixuser -i nix-env -iA lmod
```

In general it is best to work by example. There are thousands of `default.nix`
files in the `nixpkgs` GitHub repository.
