.. highlight:: console

=================
Quickstart Guide
=================

.. todo:: Provide information on (at least):

   1. How to download and test a model (i.e. get docker, go to the marketplace, get the model, run it).

   2. Use cookiecutter to develop a model

   3. Integrate an existing model with DEEPaaS

   4. Create a container from a model


Download module from the marketplace
------------------------------------

#. go to `DEEP Open Catalog <https://marketplace.deep-hybrid-datacloud.eu/>`_
#. `Browse <https://marketplace.deep-hybrid-datacloud.eu/#model-list>`_ available modules
#. Find the module and get it either from `Docker Hub <https://hub.docker.com/u/deephdc>`_ (easy) or `Github <https://github.com/topics/deep-hybrid-datacloud>`_ (pro)


Run a module locally
--------------------

.. _docker-hub-way:

Docker Hub way (easy)
^^^^^^^^^^^^^^^^^^^^^

.. admonition:: Prerequisites

    * `docker <https://docs.docker.com/install/#supported-platforms>`_
    * If you want GPU support you can install `nvidia-docker <https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0)>`_
      along with docker or install `udocker <https://github.com/indigo-dc/udocker/releases>`_ instead of docker.
      udocker is entirely a user tool, i.e. it can be installed and used without any root priveledges, e.g. in a user
      environment at HPC cluster.

1. **Run the container**

To run the Docker container directly from Docker Hub and start using the `API <https://github.com/indigo-dc/DEEPaaS>`_
simply run the following:

    Via docker command::

        $ docker run -ti -p 5000:5000 -p 6006:6006 deephdc/deep-oc-module_of_interest

    With GPU support::

        $ nvidia-docker run -ti -p 5000:5000 -p 6006:6006 deephdc/deep-oc-module_of_interest
    
    Via udocker::

        $ udocker run -p 5000:5000 -p 6006:6006 deephdc/deep-oc-module_of_interest
    
    Via udocker with GPU support::

        $ udocker pull deephdc/deep-oc-module_of_interest
        $ udocker create --name=module_of_interest deephdc/deep-oc-module_of_interest
        $ udocker setup --nvidia module_of_interest
        $ udocker run -p 5000:5000 -p 6006:6006 module_of_interest
    
2. **Access the module via API**

To access the downloaded module via `API <https://github.com/indigo-dc/DEEPaaS>`_, direct your web browser to http://127.0.0.1:5000.
If you are training a model, you can go to http://127.0.0.1:6006 to monitor the training progress (if such monitoring is
available for the model).

For more details on particular models, please, read :doc:`model <models/index>` documentation.


Github way (pro)
^^^^^^^^^^^^^^^^

.. admonition:: Prerequisites

   * `docker <https://docs.docker.com/install/#supported-platforms>`_

Using Github way allows to modify the Dockerfile for including additional packages, for example.

1. Clone the DEEP-OC-module_of_interest github repository::

    $ git clone https://github.com/indigo-dc/DEEP-OC-module_of_interest

2. Build the container::

    $ cd DEEP-OC-module_of_interest
    $ docker build -t deephdc/deep-oc-module_of_interest .

3. Run the container and access the module via API as described :ref:`above <docker-hub-way>`

.. note:: One can also clone the source code of the module, usually located in the 'module_of_interest' repository.

.. _api-integration:


Run a module on DEEP Pilot Infrastructure
-----------------------------------------

.. admonition:: Prerequisites

    * `DEEP-IAM <https://iam.deep-hybrid-datacloud.eu/>`_ registration
    * `oidc-agent <https://github.com/indigo-dc/oidc-agent/releases>`_ installed and configured for `DEEP-IAM <https://iam.deep-hybrid-datacloud.eu/>`_ (see :doc:`rclone howto <howto/oidc-agent>`).
    * `orchent <https://github.com/indigo-dc/orchent/releases>`_ tool

    If your are going to use `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu>`_ for storing you data you also have to:

    * Register at `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu>`_
    * Include `rclone <https://rclone.org/install/>`_ installation in your Dockerfile (see :doc:`rclone howto <howto/rclone>`)
    * Include call to rclone in your code (see :doc:`rclone howto <howto/rclone>`)

In order to submit your job to DEEP Pilot Infrastructure one has to create
`TOSCA YAML file <https://github.com/indigo-dc/tosca-templates/tree/master/deep-oc>`_.

The submission is then done via
::

    $ orchent depcreate ./topology-orchent.yml '{}'
    
If you also want to access `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu>`_ from your container via rclone, 
you can create a following bash script for job submission:

.. code-block:: bash

    #!/bin/bash
 
    orchent depcreate ./topology-orchent.yml '{ "rclone_url": "https://nc.deep-hybrid-datacloud.eu/remote.php/webdav/",
                                                "rclone_vendor": "nextcloud",
                                                "rclone_user": <your_nextcloud_username>
                                                "rclone_pass": <your_nextcloud_password> }'


To check status of your job
::

    $ orchent depshow <Deployment ID>


Integrate your model with the API
---------------------------------

.. image:: ../_static/deepaas.png

The `DEEPaaS API <https://github.com/indigo-dc/DEEPaaS>`_ enables a user friendly interaction with the underlying Deep
Learning modules and can be used both for training models and doing inference with the services.
Check the full :doc:`API guide <overview/api>` for the detailed info.

An easy way to integrate your model with the API and create Dockerfiles for building the Docker image with the integrated
:doc:`DEEPaaS API <overview/api>` is to use our :doc:`DEEP UC template <overview/cookiecutter-template>` when developing
your model.
