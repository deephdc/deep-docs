DEEP Dashboard
==============

The `DEEP dashboard <https://train.deep-hybrid-datacloud.eu/>`_ allows users to access computing resources to deploy, perform inference,
and train modules hosted at the `DEEP Open Catalog <https://marketplace.deep-hybrid-datacloud.eu/>`_, as well
as deploying external Docker images hosted in Dockerhub.
The Dashboard simplifies the deployment and hides some of the technical parts that most users do not need to worry about.

To be able to access the Dashboard you need to register for `IAM credentials <https://iam.deep-hybrid-datacloud.eu/>`_.

Selecting the modules
---------------------

Once you log into the Dashboard, you are able to see all possible modules for deploying.
Those are basically:

* all the Marketplace modules.
* ``Run your own module``: deploy external docker images that are not part of the Marketplace
* ``Development module``: special module especially designed to develop new modules (also called ``DEEP Development Environment``).

.. image:: ../../_static/dashboard-home.png


Making a deployment
-------------------

Once you choose the module to deploy click in ``Train module``. This allow a user to select:

* **The computing resources** of the new deployment. A user can select multiple CPUs and GPUs, the machine RAM as well as
  optionally choosing the physical site where the machine must be deployed.
* **The service** to run. Currently, options include running the :doc:`DEEPaaS API <api>` (recommended for fully
  developed modules than only need to be trained) and `JupyterLab <https://jupyterlab.readthedocs.io/en/stable/>`_
  (recommended for developing code as well for cases where access to the bash console is needed).

.. image:: ../../_static/dashboard-configure.png

Click ``Submit`` and you will be redirected to the page listing all the current deployments.


Managing the deployments
------------------------

In the ``Deployments`` tab you have a view of all the deployments you have made:

.. image:: ../../_static/dashboard-deployments.png

If your deployment is running DEEPaaS, you can click on the UUID of the deployments to access the deployments personal page and view things such
as the training history, etc.

.. image:: ../../_static/dashboard-history.png
