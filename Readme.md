# Container definition files

These files define containers useful for [RTE+RRTMGP](https://github.com/earth-system-radiation/rte-rrtmgp) and other projects.

At present these mostly define minimal Docker containers providing stable environments for continuous integration.They may be
 built without a context, e.g.
 ```docker build - < Dockerfile-nvidia-netcdf-minimal```

## Pernak Refinements

```
docker build . -t earthsystemradiation/rte-rrtmgp:ifort -f Dockerfile-ifort-minimal
docker build . -t earthsystemradiation/rte-rrtmgp:nvidia -f Dockerfile-nvidia-minimal
docker build . -t add-netcdf -f Dockerfile-add-netcdf --build-arg BASE=ifort
docker build . -t add-netcdf -f Dockerfile-add-netcdf --build-arg BASE=nvidia
docker tag add-netcdf add-netcdf:ifort
docker tag add-netcdf add-netcdf:nvidia
docker build . -t add-python -f Dockerfile-add-python --build-arg BASE=add-netcdf:ifort
docker build . -t add-python -f Dockerfile-add-python --build-arg BASE=add-netcdf:nvidia
```
