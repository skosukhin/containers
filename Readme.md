# Container definition files

These files define containers useful for [RTE+RRTMGP](https://github.com/earth-system-radiation/rte-rrtmgp) and other projects.

At present these mostly define minimal Docker containers providing stable environments for continuous integration.They may be built without a context, e.g.

```
docker build - < Dockerfile-nvidia-netcdf-minimal
```

## Docker <a name="docker"></a>

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
docker build . -t minimal-compiler:nvfortran -f Dockerfile-nvfortran-minimal # authentication error for some reason
docker pull nvcr.io/nvidia/nvhpc:21.1-devel-cuda11.2-ubuntu20.04 # worked fine
docker build . -t minimal-compiler:nvfortran -f Dockerfile-nvfortran-minimal
docker build . -t add-netcdf:nvfortran -f Dockerfile-add-netcdf --build-arg COMPILER=nvfortran
docker build . -t rte-rrtmgp-ci:nvfortran -f Dockerfile-add-python --build-arg COMPILER=nvfortran
docker tag rte-rrtmgp-ci:nvfortran earthsystemradiation/rte-rrtmgp-ci:nvfortran
docker push earthsystemradiation/rte-rrtmgp-ci:nvfortran
```

To elaborate on the common steps in these processes:

1. **Build local starter image**: using direction from a compiler-dependent Dockerfile (`-f Dockerfile-*-minimal`) in the current working directory (`build .`), build an image with the OS and compiler requirements, then tag it as `-t minimal-compiler:$COMPILER` (`$COMPILER` either `ifort` or `nvfortran`).
2. **Build local netCDF image**: using direction from `Dockerfile-add-netcdf` in the current working directory (`build .`), build an image on top of the starter, then tag it as `-t add-netcdf:$COMPILER` (`$COMPILER` either `ifort` or `nvfortran`).
3. **Build Python image**: using direction from `Dockerfile-add-python` in the current working directory (`build .`), build an image that will be used in RTE+RRTMGP Continuous Integration (CI), then tag it as `-t rte-rrtmgp-ci:$COMPILER` (`$COMPILER` either `ifort` or `nvfortran`).
4. **Tag for and push to public repository**: tag the local images from Step 3 with their counterparts in the `earthsystemradion` DockerHub repository, again separating by compiler, then push to the repository

Local builds can be bypassed by replacing local image names in the build with the DockerHub repository names, e.g.:

```
docker build . -t earthsystemradiation/rte-rrtmgp-ci:ifort -f Dockerfile-add-python --build-arg COMPILER=ifort
```

Since `rte-rrtmgp-ci` is dependent on the starter and netCDF images, there is likely no reason to push either to DockerHub. The local names of these two images are used in `Dockerfile-add-netcdf` and `Dockerfile-add-python`.

## Docker Compose <a name="compose"></a>

Alternatively, users can use `docker-compose` to build the images and run the containers. Docker Compose reads bulid and run parameters a YAML file. For RTE-RRTMGP Continuous Integration, `rte-rrtmgp-ci.yml` is the configuration file with the "service stack." Images can be built with:

```
docker-compose -f rte-rrtmgp-ci.yml build
```

The previous command will build all images. For specific images, users can provide their service name after the `build` option. Currently, the services correspond to [the builds that were done separately](#docker) with each `docker build` command and are named:

1. `min-ifort`
2. `min-nvfortran`
3. `nc-ifort`
4. `nc-nvfortran`
5. `ci-ifort`
6. `ci-nvfortran`

Note that images 3-6 depend on previous services, but Docker Compose only has a `depends_on` option for [running the services](https://stackoverflow.com/a/37945466) (i.e., running the container) so one either has to build the images at the beginning of the chain or build them first.

Again, `docker-compose` can be used to run services as well, but the service stack will need more additions before anything of substance can be performed.
