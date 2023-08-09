# Building and deploying containers

**Home:** [Software management](INDEX.md)

The following contains the essential parts you will need to build and deploy Apptainer containers on our
infrastructure. 

## Before building a container image
Container images in our environment are intended to be a last resort option. They are to be used only for software packages that are too complicated to install with our other methods such as [EasyBuild](easybuild.md) or [Python wheels](python.md). Such examples could be software packages that heavily depend on Anaconda, or complex workflows with a lot of intertwined dependencies. If you are not certain that the container you want to build meets these criteria, we encourage you to ask in our #rsnt-software Slack channel. 

Please be aware that we intend to deploy an [automated container publishing system](https://cvmfs.readthedocs.io/en/latest/cpt-containers.html#distributing-container-images-on-cernvm-fs) when it is available. It will regularly pull, convert, and publish a list of approved container images, from an OCI-format ("Docker") container registry to a CVMFS repository. Keep in mind that in the future it may be advantageous and desirable to use the automated system to publish container images instead, but the manual procedure described here will still be possible.

## Where to build a container image 
Our build nodes have the required configuration to build container images. Every staff member of the federation can get access to our build nodes. Please refer to our [initial setup](https://github.com/ComputeCanada/software-stack/blob/containers-doc/doc/setup.md#before-you-begin) page for instructions on requesting access to those nodes.

# Building a container image
One you have determined that the container you want to build meets the above criteria, building a container on our infrastructure is done through a [script](https://github.com/ComputeCanada/containers-recipes/blob/main/build_container_image.sh) which supports building a container image based on three different types of sources:
1. It can build based on an [Apptainer definition file](https://apptainer.org/docs/user/main/definition_files.html)
2. It can build based on a [Dockerfile](https://docs.docker.com/engine/reference/builder)
3. It can pull an existing container image from a public container registry

## Building a container image using [build_container_image.sh](https://github.com/ComputeCanada/containers-recipes/blob/main/build_container_image.sh)
Our `build_container_image.sh` script has 5 mandatory options: 
* `-i` indicates the source type. This is one of `<def|Dockerfile|image>`
* `-s` indicates the source to use. For `def` and `Dockerfile` types of source, this is the relative path to the recipe. For `image` type of source, this is the argument that you would pass to a `podman pull` or `docker pull` command. 
* `-n` indicates the name of the container image being built
* `-v` indicates the version of the container image being built
* `-t` indicates the type of container image being built. It is either `sandbox` or `sif`. Note that for deploying containers on CVMFS, only `sandbox` images are supported. 

The script also has a `-h` option to show the help text, and `-d` option to run in dry-run mode. 

When working on a recipe, we strongly recommend cloning our [Github repository](https://github.com/ComputeCanada/containers-recipes) and working from it. 

### Building from an Apptainer definition file
Use the command
```
./build_container_image.sh -t sandbox -n <name> -v <version> -i def -s path/to/def-file
``` 
where `path/to/def-file` is within the git repository. 

### Building from an Dockerfile
Use the command
```
./build_container_image.sh -t sandbox -n <name> -v <version> -i Dockerfile -s path/to/Dockerfile
``` 
where `path/to/Dockerfile` is within the git repository. 

### Building from a container image from a public registry
To pull an image from the docker.io registry, use: 
```
./build_container_image.sh -t sandbox -n <name> -v <version> -i image -s docker.io/name:version
``` 
If at all possible, pull a specific version, and not `latest`. 

## Testing the container image produced
Before the container image is built to get on our CVMFS infrastructure, it must first be built and tested in your own account. Once the recipe works and produces a container image, please test it using our `apptainer` module.

## Building a container image to deploy on CVMFS
### Creating a recipe and opening a pull request
When building a container through an Apptainer definition file or a Dockerfile, the recipes must first submitted as a pull request to our [containers-recipes repository on Github](https://github.com/ComputeCanada/containers-recipes/tree/main). The pull request description should explain why the software package can not or should not be installed through our other methods. 

Recipes provided must meet the following requirements to be accepted:
* it must be added to the `recipes` folder
* it must clearly be identified by a name and version
* if there are additional files other than the definition file or the Dockerfile, the recipe must be in a folder identified by its name and version
* the recipe must be as reproducible as possible. This means that everything installed in the container should explicitly specify versions to be used. 
* the recipe must be tested with our build script (see below) and confirmed to produce a working container

Once you have built the container image in your own account and tested it, open the pull request on our repository, and ask in #rsnt-software for it to be reviewed and merged. You can only build centrally after this step has happened. 

### Updating central recipes on the build-nodes
Staff members with the appropriate permissions can update a central clone of our `containers-recipes` repository on our build-nodes using the command
```
sudo -iu containeruser  pull_containers-recipes
``` 

### Building the image centrally
Staff members with the appropriate permissions can build the container image to be stored under `/cvmfs/containers.computecanada.ca/content/containers/` on our build-nodes using the command
```
sudo -iu containeruser build_container_image.sh <options>
``` 

This will build the image and put it under the `/cvmfs/containers.computecanada.ca/content/containers/` path, from which it can then get deployed on CVMFS.

### Deploying containers with CVMFS

**See:** [Deploying containers with CVMFS](cvmfs.md#deploying-containers-with-cvmfs)

