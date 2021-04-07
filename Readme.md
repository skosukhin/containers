# Container definition files

These files define containers useful for [RTE+RRTMGP](https://github.com/earth-system-radiation/rte-rrtmgp) and other projects.

At present these mostly define minimal Docker containers providing stable environments for continuous integration.They may be built without a context, e.g.

```
docker build - < Dockerfile-nvidia-netcdf-minimal
```

## Pernak Refinements

We start with images that are effectively Ubuntu OS base images, then add FORTRAN compiler dependencies for Intel and Nvidia -- `ifort` and `nvfortran`, respectively. These are separate images, and we use them as bases for the netCDF library installation. Finally, we add the Python libraries necessary for RTE-RRTMGP continuous integration, then push the images to a publicly available repository on DockerHub. We use tags to indicate which compiler is used in the resulting images.

For the `ifort` image:

```
docker build . -t minimal-compiler:ifort -f Dockerfile-ifort-minimal
docker build . -t add-netcdf:ifort -f Dockerfile-add-netcdf --build-arg COMPILER=ifort
docker build . -t rte-rrtmgp-ci:ifort -f Dockerfile-add-python --build-arg COMPILER=ifort
docker tag rte-rrtmgp-ci:ifort earthsystemradiation/rte-rrtmgp-ci:ifort
docker push earthsystemradiation/rte-rrtmgp-ci:ifort
```

And for `nvfortran`:

```
docker build . -t minimal-compiler:nvidia -f Dockerfile-nvidia-minimal # authentication error for some reason
docker pull nvcr.io/nvidia/nvhpc:21.1-devel-cuda11.2-ubuntu20.04 # worked fine
docker build . -t minimal-compiler:nvidia -f Dockerfile-nvidia-minimal
docker build . -t add-netcdf:nvidia -f Dockerfile-add-netcdf --build-arg COMPILER=nvidia
docker build . -t rte-rrtmgp-ci:nvidia -f Dockerfile-add-python --build-arg COMPILER=nvidia
docker tag rte-rrtmgp-ci:nvidia earthsystemradiation/rte-rrtmgp-ci:nvidia
docker push earthsystemradiation/rte-rrtmgp-ci:nvidia
```

To elaborate on the common steps in these processes:

1. **Build local starter image**: using direction from a compiler-dependent Dockerfile (`-f Dockerfile-*-minimal`) in the current working directory (`build .`), build an image with the OS and compiler requirements, then tag it as `-t minimal-compiler:$COMPILER` (`$COMPILER` either `ifort` or `nvidia`).
2. **Build local netCDF image**: using direction from `Dockerfile-add-netcdf` in the current working directory (`build .`), build an image on top of the starter, then tag it as `-t add-netcdf:$COMPILER` (`$COMPILER` either `ifort` or `nvidia`).
3. **Build Python image**: using direction from `Dockerfile-add-python` in the current working directory (`build .`), build an image that will be used in RTE+RRTMGP Continuous Integration (CI), then tag it as `-t rte-rrtmgp-ci:$COMPILER` (`$COMPILER` either `ifort` or `nvidia`).
4. **Tag for and push to public repository**: tag the local images from Step 3 with their counterparts in the `earthsystemradion` DockerHub repository, again separating by compiler, then push to the repository

Local builds can be bypassed by replacing local image names in the build with the DockerHub repository names, e.g.:

```
docker build . -t earthsystemradiation/rte-rrtmgp-ci:ifort -f Dockerfile-add-python --build-arg COMPILER=ifort
```

Since `rte-rrtmgp-ci` is dependent on the starter and netCDF images, there is likely no reason to push either to DockerHub. The local names of these two images are used in `Dockerfile-add-netcdf` and `Dockerfile-add-python`.
