# Container definition files

These files define containers useful for [RTE+RRTMGP](https://github.com/earth-system-radiation/rte-rrtmgp) and other projects.

At present these mostly define minimal Docker containers providing stable environments for continuous integration. They may be built without a context, e.g.

```
docker build - < Dockerfile-nvhpc-netcdf-minimal
```

## Docker <a name="docker"></a>

We start with images that are effectively Ubuntu OS base images, then add FORTRAN compilers from Intel oneAPI and Nvidia HPC SDK toolchains. These are separate images, and we use them as bases for the netCDF library installation. Finally, we add the Python libraries necessary for RTE-RRTMGP continuous integration, then push the images to publicly available repositories on DockerHub and GitHub Container Registry. We use tags to indicate which toolchain is used in the resulting images.

For the `oneapi` image:

```
docker build . -t minimal-toolchain:oneapi -f Dockerfile-oneapi-minimal
docker build . -t add-netcdf:oneapi -f Dockerfile-add-netcdf --build-arg TOOLCHAIN=oneapi
docker build . -t add-python:oneapi -f Dockerfile-add-python --build-arg TOOLCHAIN=oneapi
docker build . -t rte-rrtmgp-ci:oneapi -f Dockerfile-finalize --build-arg TOOLCHAIN=oneapi
docker tag rte-rrtmgp-ci:oneapi earthsystemradiation/rte-rrtmgp-ci:oneapi
docker push earthsystemradiation/rte-rrtmgp-ci:oneapi
docker tag rte-rrtmgp-ci:oneapi ghcr.io/earth-system-radiation/rte-rrtmgp-ci:oneapi
docker push ghcr.io/earth-system-radiation/rte-rrtmgp-ci:oneapi
```

And for `nvhpc`:

```
docker build . -t minimal-toolchain:nvhpc -f Dockerfile-nvhpc-minimal
docker build . -t add-netcdf:nvhpc -f Dockerfile-add-netcdf --build-arg TOOLCHAIN=nvhpc
docker build . -t add-python:nvhpc -f Dockerfile-add-python --build-arg TOOLCHAIN=nvhpc
docker build . -t rte-rrtmgp-ci:nvhpc -f Dockerfile-finalize --build-arg TOOLCHAIN=nvhpc
docker tag rte-rrtmgp-ci:nvhpc earthsystemradiation/rte-rrtmgp-ci:nvhpc
docker push earthsystemradiation/rte-rrtmgp-ci:nvhpc
docker tag rte-rrtmgp-ci:nvhpc ghcr.io/earth-system-radiation/rte-rrtmgp-ci:nvhpc
docker push ghcr.io/earth-system-radiation/rte-rrtmgp-ci:nvhpc
```

To elaborate on the common steps in these processes:

1. **Build local starter image**: using direction from a toolchain-dependent Dockerfile (`-f Dockerfile-*-minimal`) in the current working directory (`build .`), build an image with the OS and toolchain requirements, then tag it as `-t minimal-toolchain:$TOOLCHAIN` (`$TOOLCHAIN` either `oneapi` or `nvhpc`).
2. **Build local netCDF image**: using direction from `Dockerfile-add-netcdf` in the current working directory (`build .`), build an image on top of the starter, then tag it as `-t add-netcdf:$TOOLCHAIN` (`$TOOLCHAIN` either `oneapi` or `nvhpc`).
3. **Build Python image**: using direction from `Dockerfile-add-python` in the current working directory (`build .`), build an image on top of the netCDF image, then tag it as `-t add-python:$TOOLCHAIN` (`$TOOLCHAIN` either `oneapi` or `nvhpc`).
4. **Build final CI image**: using direction from `Dockerfile-finalize` in the current working directory (`build .`), build an image on top of the Python image that will be used in RTE+RRTMGP Continuous Integration (CI), then tag it as `-t rte-rrtmgp-ci:$TOOLCHAIN` (`$TOOLCHAIN` either `oneapi` or `nvhpc`).
5. **Tag for and push to public repository**: tag the local images from Step 4 with their counterparts in the `earthsystemradion` DockerHub repository and in the `ghcr.io/earth-system-radiation` GitHub Container Registry, again separating by toolchain, then push to the repositories.

Local builds can be bypassed by replacing local image names in the build with the DockerHub repository names, e.g.:

```
docker build . -t earthsystemradiation/rte-rrtmgp-ci:oneapi -f Dockerfile-add-python --build-arg TOOLCHAIN=oneapi
docker build . -t ghcr.io/earth-system-radiation/rte-rrtmgp-ci:oneapi -f Dockerfile-add-python --build-arg TOOLCHAIN=oneapi
```

Since `rte-rrtmgp-ci` is dependent on the starter and netCDF images, there is likely no reason to push either to DockerHub or to the GitHub Container Registry. The local names of these two images are used in `Dockerfile-add-netcdf` and `Dockerfile-add-python`.

## Docker Compose <a name="compose"></a>

Alternatively, users can use `docker-compose` to build the images and run the containers. Docker Compose reads build and run parameters from a YAML file. For RTE-RRTMGP Continuous Integration, `rte-rrtmgp-ci.yml` is the configuration file with the "service stack." Images can be built with:

```
docker-compose -f rte-rrtmgp-ci.yml build
```

The previous command will build all images. For specific images, users can provide their service name after the `build` option. Currently, the services correspond to [the builds that were done separately](#docker) with each `docker build` command and are named:

1. `min-oneapi`
2. `min-nvhpc`
3. `nc-oneapi`
4. `nc-nvhpc`
5. `py-oneapi`
6. `py-nvhpc`
7. `ci-oneapi`
8. `ci-nvhpc`

Note that images 3-8 depend on previous services, but Docker Compose only has a `depends_on` option for [running the services](https://stackoverflow.com/a/37945466) (i.e., running the container) so one either has to build the images at the beginning of the chain or already have them in their local image repository.

Again, `docker-compose` can be used to run services as well, but the service stack will need more additions before anything of substance can be performed.
