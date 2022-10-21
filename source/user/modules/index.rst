Modules
=======

All  modules are found at the `DEEP Marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`_, the source code is
hosted under `Github's deephdc <https://github.com/deephdc>`__ organization and the corresponding Docker images are
hosted under `DockerHub's deephdc <https://hub.docker.com/u/deephdc/>`__ organization.

Github repositories follow the following convention:

* ``deephdc/some_module``: source code of the module
* ``deephdc/DEEP-OC-some_module``: Dockerfiles and metadata of that module.

Docker images have usually tags depending on whether they are using Github's ``master`` or ``test`` and
whether they use ``cpu`` or ``gpu``. Tags are usually:

* ``latest`` or ``cpu``: master + cpu
* ``gpu``: master + gpu
* ``cpu-test``: test + cpu
* ``gpu-test``: test + gpu
