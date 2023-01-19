Modules
=======

All  modules are found at the `DEEP Marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`_, the source code is
hosted under `Github's deephdc <https://github.com/deephdc>`__ organization and the corresponding Docker images are
hosted under `DockerHub's deephdc <https://hub.docker.com/u/deephdc/>`__ organization.

Modules developed by **deephdc members** follow the following convention:

* ``deephdc/<project-name>``: source code of the module
* ``deephdc/DEEP-OC-<project-name>``: Dockerfiles and metadata of that module.

Modules developed by **external users** follow the following convention:

* ``deephdc/UC-<github-user>-<project-name>``: source code of the module
* ``deephdc/UC-<github-user>-DEEP-OC-<project-name>``: Dockerfiles and metadata of that module.

Docker images have usually tags depending on whether they are using Github's ``master`` or ``test`` and
whether they use ``cpu`` or ``gpu``. Tags are usually:

* ``latest`` or ``cpu``: master + cpu
* ``gpu``: master + gpu
* ``cpu-test``: test + cpu
* ``gpu-test``: test + gpu
