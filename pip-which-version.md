# pip: Which version is it installing anyway ? 
A blog post about meandering through the way `pip` decides which package to install

Author: Maxime Boissonneault (Compute Canada, Calcul Québec, Université Laval)

## Preamble
My team at Compute Canada has been managing scientific software available for our users since 2015. 
We maintain a [collection of thousands of scientific software packages](https://docs.computecanada.ca/wiki/Available_software) available 
as [modules](https://docs.computecanada.ca/wiki/Utiliser_des_modules/en). This works rather well for the somewhat standard ecosystem of
software packages which use traditional build systems such as Autoconf, CMake, etc. To automate software package installation, 
we use [EasyBuild](https://easybuilders.github.io/). It works well for such software packages because they use well established tools to build
and install, they are usually slow moving and new versions are usually backward compatible. 

For Python packages however, the ecosystem is much more dynamic: packages evolve quickly and they often have a lot of dependencies, and many users have
package requirements which vary widely and are often incompatible with the requirements of other users. In such an ecosystem, it is better if each
user can trivially install what they need. We have therefore adopted the strategy of pointing users to install the packages they need through
[virtual environment](https://docs.computecanada.ca/wiki/Python#Creating_and_using_a_virtual_environment). Thanks to `pip`, this works very well for 
pure python packages. For compiled packages however, things become complicated. 

To help with compiled packages, the [`manylinux`](https://github.com/pypa/manylinux) standard was introduced. Our experience with these have
been hit and miss. Sometimes they work, sometimes they don't. That is because they tend to assume libraries installed in standard locations (`/usr/lib`), 
while those are often outdated and should not be used on ARC clusters. These wheels also usually come with some version of compiled libraries which we often
*already have* installed and optimized in our software stack, causing duplication and sometimes incompatibilities. 
Finally, `manylinux` wheels are compiled for portability, not for performance. They will usually not include optimizations for specific 
CPU instruction sets for example. Because of these issues, we ended up 
[blocking them on our infrastructure by marking them as incompatible](https://stackoverflow.com/questions/37231799/exclude-manylinux-wheels-when-downloading-from-pip).

Because of the above, we started to [build our own Python wheels](https://github.com/ComputeCanada/wheels_builder/) many years ago. 
This allows us to ensure that compiled wheels are linked against optimized libraries that are sure to be compatible with our software environment. 
It also allows us to provide a partial mirror of PyPI which is available on our clusters' compute nodes even if they don't have access to Internet, through
our [distributed filesystem](https://docs.computecanada.ca/wiki/Accessing_CVMFS).

For this to be transparent to our users, we defined a `PIP_CONFIG_FILE` in our environment, and had it configured with 
```
find-links = /path/to/our/directory/of/wheels
``` 
An equivalent configuration would have been to use `extra-index-url`. For reasons explained below, this broke down when the version 21 of `pip` came out, forcing
us to add a `constraint` file which required `pip < 21`. This worked fine at first, but was not maintainable in the long term. 


## Different use cases: private packages, security, and optimized packages
There are many use cases for which one may want to alter the list of packages found by `pip`, usually through [PyPI](https://pypi.org/). One may want to provide
*additional* packages, which are not otherwise available on PyPI. One may want to *block* packages found on PyPI unless they are vetted. Finally, one may want to
provide *optimized* or *alternative* packages to those available on PyPI. Unfortunately, the most widely exposed features to handle such alterations of the
list of packages found by pip, i.e. `--find-links` or `--extra-index-url`, are not well adapted for all of the above use cases. 

If you are concerned about providing *private packages*, you first have to decide whether you trust PyPI or not, knowing that PyPI is a public repository, it is
subject to [supply chain attacks](https://www.bleepingcomputer.com/news/security/researcher-hacks-over-35-tech-firms-in-novel-supply-chain-attack/). You may want 
to read about the [Package Index Name Retention clause about Name squatting](https://www.python.org/dev/peps/pep-0541/#invalid-projects) or [any](https://discuss.python.org/t/pep-541-should-name-squatting-be-actively-discouraged/2407) [other](https://github.com/pypa/warehouse/issues/4004) [related](https://discuss.python.org/t/pypi-as-a-project-repository-vs-name-registry-a-k-a-pypi-namesquatting-e-g-for-fedora-packages/4045/4) discussion. In short, 
if your goal is to provide additional private packages, you should be worried about someone else uploading a package with the same name to PyPI and injecting
their own code into your infrastructure. 

If you are concerned about *really securing* your environment, you should not at all rely on PyPI. You should carefully curate a list of Python packages which
you need in your environment, and disable PyPI altogether. 

Finally, if your goal is not to secure your environment, but rather to provide versions of packages that are compiled with additional optimization or
configuration options, you can rely on PyPI, but you can not use `--find-links` or `--extra-index-url`. 

## Different solutions and when they apply
### `--find-links` or `--extra-index-url` 
These two options of `pip` are used to add *extra* locations for `pip` to find packages. These locations are in addition to PyPI. It is very important to note that 
they are not *prefered* to PyPI. Based on the [number](https://github.com/pypa/pip/issues/8606) of [threads](https://github.com/pypa/pip/issues/5045) [about](https://stackoverflow.com/questions/67253141/python-pip-priority-order-with-index-url-and-extra-index-url) [it](https://www.gitmemory.com/issue/pypa/pip/8606/788258060), this is a common misconception. The Pip developers are adamant that when multiple indexes are provided, they are [treated equally](https://github.com/pypa/pip/issues/8606#issuecomment-665554122). This means that if the same package, with the same version, is found in multiple indexes, `pip` is free to install whichever: the behavior is undefined, and it can change from one version of `pip` to the next. 

These two options are basically useful when you have a *disjoint* set of Python packages from PyPI, not an *overlapping* set. This comes with a security issue: you may well have a disjoint set on day 1, but somebody in the future can simply upload a Python package with the same name and version as your private package, and it may be installed in your infrastructure without you even noticing it. 

For this reason, if you opt to using these two options in order to provide *private packages*, you should *also* use `--index-url` to prevent `pip` from looking directly on PyPI. You can then configure either a [devpi](https://www.devpi.net/) or a [simpleindex](https://github.com/uranusjr/simpleindex) server, and configure them to redirect to PyPI *except* for the packages which you wish to control. 

### `--index-url` or `--no-index` 
These two options are used to either replace the default PyPI index by your own (using the aforementionned `devpi` or `simpleindex` solutions), or to completely disable the index (and then only locate packages using local paths). These options are the only recommendable options if *security* is your absolute concern. You can then populate a carefully curated list of packages which you allow. 

### Build tags and local versions
While configuring indexes can not be relied upon to prioritize between two identical packages, Python offers two mechanisms which *can* be used for such purpose. Those are [build tags](https://packaging.python.org/specifications/binary-distribution-format/#file-name-convention) and [local version identifiers](https://www.python.org/dev/peps/pep-0440/#local-version-identifiers). These options were pointed to us in [this issue](https://github.com/pypa/pip/issues/10156), by [Pradyun Gedam](https://github.com/pradyunsg). 

Build tags can be simply added to the name of a wheel file after the version number: 
```
{distribution}-{version}(-{build tag})?-{python tag}-{abi tag}-{platform tag}.whl
``` 
If such a tag is found, it is used as a tie breaker between two otherwise identical packages. 

Local version identifiers are a bit trickier to change, because the metadata provided within the wheel needs to match the wheel name. A wheel file with a 
local version identifier would be named
```
{distribution}-{version}+{local version identifier}-{python tag}-{abi tag}-{platform tag}.whl
``` 
and the version would be considered as `{version}+{local version identifier}` in the metadata. If such a local version identifier is found, it is also used as a tie breaker between otherwise identical packages. In both cases, issuing `pip install {distribution}=={version}` will accept packages with or without either build tags or local version identifiers, but it will *prefer* install, in this order: 
1. {version}+{local version identifier}
2. {version}-{build tag}
3. {version}

Note that if one does not require an explicit version when installing the package, and if a more recent version of the package is uploaded on PyPI, `pip` will still prefer the more recent version. 

Build tags must start with a digit, while local version identifiers do not have to. In addition, when listing installed packages with `pip list` or `pip freeze`, build tags will not be displayed, while local version identifiers will be displayed. 

## What we ended up doing
In our case, security is not our concern. There is a clear divide between securing our infrastructure (what is run with elevated privileges), and the code that
users our infrastructure will run. Users run code without any privilege, but we do not limit the codes they can run. For better or for worst, most users already
rely on PyPI heavily to install their packages. Our concern is only to provide our users with packages that are optimized for our infrastructure and that are known
to be compatible with the libraries we provide. We therefore decided to go with local version specifiers. That meant we had to update our build scripts to inject
this identifier into the wheels we build. This was however not very complicated to implement with a [simple shell script](https://github.com/ComputeCanada/wheels_builder/pull/29/files).

Since the wheels we provide to our users already rely on PyPI, we are not concerned about blocking or filtering packages offered by PyPI, and we therefore still use `--find-links`, except we are not relying on it for prioritizing our own wheels: that is done by the local version specifiers. 

As of July 2021, we have yet to repackage all of our existing python wheels to add local version specifiers, since this will be a long endeavour, but the newly compiled ones automatically include it. We are therefore still relying on a `pip<21` constraint, but we should be able to lift that constraint in the future. 
