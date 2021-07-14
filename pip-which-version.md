# pip: Which version is it anyway ? 
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


## Two distinct use cases: optimization and compatibility vs security

## Different solutions and when they apply

## What we ended up doing


