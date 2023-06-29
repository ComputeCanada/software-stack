# Building and deploying containers

**Home:** [Software management](INDEX.md)

The following contains the essential parts you will need to build and deploy Apptainer containers on our
infrastructure. 

## Before building a container image
Container images in our environment are intended to be a last resort option. They are to be used only for software packages that are too complicated to install with our other methods such as [EasyBuild](easybuild.md) or [Python wheels](python.md). Such examples could be software packages that heavily depend on Anaconda, or complex workflows with a lot of intertwined dependencies.

## Building and deploying a container
Building a container on our infrastructure is done through a [script](https://github.com/ComputeCanada/containers-recipes/blob/main/build_container_image.sh) which supports building a container image based on three different types of sources:
1. It can build based on an [Apptainer definition file](https://apptainer.org/docs/user/main/definition_files.html)
2. It can build based on a [Dockerfile](https://docs.docker.com/engine/reference/builder)
3. It can pull an existing container image from a public container registry


### Creating or changing a recipe


### Deploying containers with CVMFS

**See:** [Deploying containers with CVMFS](cvmfs.md#deploying-containers-with-cvmfs)

