.. highlight:: console

=================
Quickstart Guide
=================

#. Go to the `DEEP Marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`__
#. Browse `available modules <https://marketplace.deep-hybrid-datacloud.eu/#model-list>`_.
#. Find the module you are interested in and get it

Let's explore what we can do with it!


Run a module locally
--------------------

.. admonition:: Requirements

    This section requires having `docker <https://docs.docker.com/install/#supported-platforms>`__ installed.

    Starting from version 19.03 docker supports NVIDIA GPUs (see `here <https://docs.docker.com/engine/release-notes/>`__ and `here <https://github.com/moby/moby/pull/38828>`__).
    If you happen to be using an older version you can give a try to `nvidia-docker <https://github.com/nvidia/nvidia-docker/wiki/Installation-(version-2.0)>`__

    If you need to use docker in an envirronment without root privileges (eg. an HPC cluster)
    check `udocker <https://github.com/indigo-dc/udocker/releases>`__ instead of docker.

Run the container
^^^^^^^^^^^^^^^^^

We will pull the containers directly from DockerHub.

* Running on CPUs:

    * with docker::

        $ docker run -ti -p 5000:5000 -p 6006:6006 deephdc/deep-oc-module_of_interest

    * with udocker::

        $ udocker run -p 5000:5000 -p 6006:6006 deephdc/deep-oc-module_of_interest

* Running on GPUs:

    * with docker (19.03 or above)::

        $ docker run -ti --gpus all -p 5000:5000 -p 6006:6006 deephdc/deep-oc-module_of_interest

    * with nvidia-docker::

        $ nvidia-docker run -ti -p 5000:5000 -p 6006:6006 deephdc/deep-oc-module_of_interest

    * with udocker (GPU support enabled)::

        $ udocker pull deephdc/deep-oc-module_of_interest
        $ udocker create --name=module_of_interest deephdc/deep-oc-module_of_interest
        $ udocker setup --nvidia module_of_interest
        $ udocker run -p 5000:5000 -p 6006:6006 module_of_interest

Access the module via API
^^^^^^^^^^^^^^^^^^^^^^^^^

Once you have your container running, you have to access the downloaded module via :doc:`the DEEPaaS API <overview/api>`.
In your web browser go to http://0.0.0.0:5000/ui and start trying the module.

If you are training a model, you can go to http://0.0.0.0:6006 to monitor the training progress (if such monitoring is
available for the model).

For more details on particular models, please read :doc:`the module's <modules/index>` documentation.

.. image:: ../_static/deepaas.png
   :width: 500 px

**Related HowTo's:**

* :doc:`How to perform inference locally <howto/inference-locally>`
* :doc:`How to perform training locally <howto/train-model-locally>`


Train a module on DEEP Dashboard
--------------------------------

.. admonition:: Requirements

    For accessing the Dashboard, you will need to register a `DEEP-IAM <https://iam.deep-hybrid-datacloud.eu/>`__ credential.

Sometimes running a module locally is not enough as one may need more powerful computing resources (like GPUs) in order
to train a module faster. For that you can use the :doc:`DEEP Dashboard <overview/dashboard>`.

In the Dashboard select a module you want to run and the computing resources you need.
Once you have your module deployed, you will be able to train the module and view the training history.

.. image:: ../_static/dashboard-history.png

**Related HowTo's:**

* :doc:`How to train a model remotely <howto/train-model-remotely>`
* :doc:`How to perform inference remotely <howto/inference-remotely>`


Develop and share your own module
---------------------------------

The best way to develop a module is to start from the :doc:`DEEP Modules Template <overview/cookiecutter-template>`.
It will create a project structure and files necessary for an easy :ref:`integration with the DEEPaaS
API <user/overview/api:Integrate your model with the API>`.
The :doc:`DEEPaaS API <overview/api>` enables a user-friendly interaction with the underlying Deep Learning modules
and can be used both for training models and doing inference with the services. The integration with the API is based
on the definition of entrypoints to the model and the creation of standard API methods (eg. train, predict, etc).

**Related HowTo's:**

* :doc:`How to use the DEEP Modules Template for model development <overview/cookiecutter-template>`
* :doc:`How to develop your own machine learning model <howto/develop-model>`
* :ref:`How to integrate your model with the DEEPaaS API <user/overview/api:Integrate your model with the API>`
