.. highlight:: console

=================
Quickstart Guide
=================

#. go to `DEEP Marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`_
#. `Browse <https://marketplace.deep-hybrid-datacloud.eu/#model-list>`_ available modules
#. Find the module you are interested in and get it

Let's explore what we can do with it!


Run a module locally
--------------------

.. admonition:: Requirements

    * `docker <https://docs.docker.com/install/#supported-platforms>`_
    * If GPU support is needed:

       * you can install `nvidia-docker <https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0)>`_
         along with docker, OR
       * install `udocker <https://github.com/indigo-dc/udocker/releases>`_ instead of docker.
         `udocker <https://github.com/indigo-dc/udocker/releases>`_ is entirely a user tool, i.e. it can be installed
         and used without any root privileges, e.g. in a user environment at HPC cluster.

     **N.B.:** Starting from version 19.03 docker supports NVIDIA GPUs, i.e. no need for nvidia-docker
     (see `Release notes <https://docs.docker.com/engine/release-notes/>`_ and `moby/moby#38828 <https://github.com/moby/moby/pull/38828>`_)


Run the container
^^^^^^^^^^^^^^^^^

Run the Docker container directly from Docker Hub:

    Via docker command::

        $ docker run -ti -p 5000:5000 -p 6006:6006 deephdc/deep-oc-module_of_interest

    Via udocker::

        $ udocker run -p 5000:5000 -p 6006:6006 deephdc/deep-oc-module_of_interest

    With GPU support::

        $ nvidia-docker run -ti -p 5000:5000 -p 6006:6006 deephdc/deep-oc-module_of_interest

    If docker version is 19.03 or above::

        $ docker run -ti --gpus all -p 5000:5000 -p 6006:6006 deephdc/deep-oc-module_of_interest

    Via udocker with GPU support::

        $ udocker pull deephdc/deep-oc-module_of_interest
        $ udocker create --name=module_of_interest deephdc/deep-oc-module_of_interest
        $ udocker setup --nvidia module_of_interest
        $ udocker run -p 5000:5000 -p 6006:6006 module_of_interest

Access the module via API
^^^^^^^^^^^^^^^^^^^^^^^^^

To access the downloaded module via :doc:`the DEEPaaS API <overview/api>`, direct your web browser to http://0.0.0.0:5000/ui.
If you are training a model, you can go to http://0.0.0.0:6006 to monitor the training progress (if such monitoring is
available for the model).

For more details on particular models, please read :doc:`the module's <modules/index>` documentation.

.. image:: ../_static/deepaas.png
   :width: 500 px

**Related HowTo's:**

* :doc:`How to perform inference locally <howto/inference-locally>`
* :doc:`How to perform training locally <howto/train-model-locally>`


Train a module on DEEP Pilot Infrastructure
-------------------------------------------

.. admonition:: Requirements

    * `DEEP-IAM <https://iam.deep-hybrid-datacloud.eu/>`_ registration

Sometimes running a module locally is not enough as one may need more powerful computing resources (like GPUs) in order
to train a module faster. You may request `DEEP-IAM <https://iam.deep-hybrid-datacloud.eu/>`_ registration and 
then use the DEEP Pilot Infrastructure to deploy a module. For that you can use the :doc:`DEEP Dashboard <overview/dashboard>`. 
There you select a module you want to run and the computing resources you need. Once you have your module deployed, you will be able
to train the module and view the training history:

.. image:: ../_static/dashboard-history.png

**Related HowTo's:**

* :doc:`How to train a model remotely <howto/train-model-remotely>`
* :doc:`How to deploy with CLI <howto/deploy-orchent>`


Develop and share your own module
---------------------------------

The best way to develop a module is to start from :doc:`the DEEP Data Science template <overview/cookiecutter-template>`. It will create a project structure and files necessary for an easy :ref:`integration with the DEEPaaS API <user/overview/api:Integrate your model with the API>`. 
The :doc:`DEEPaaS API <overview/api>` enables a user-friendly interaction with the underlying Deep Learning modules and can be used both for training models and doing inference with the services. The integration with the API is based on the definition of entrypoints to the model and the creation of standard API methods (eg. train, predict, etc).



**Related HowTo's:**

* :doc:`How to use the DEEP Data Science template for model development <overview/cookiecutter-template>`
* :doc:`How to develop your own machine learning model <howto/develop-model>`
* :ref:`How to integrate your model with the DEEPaaS API <user/overview/api:Integrate your model with the API>`
* :doc:`How to add your model to the DEEP Marketplace <howto/add-to-DEEP-marketplace>`
