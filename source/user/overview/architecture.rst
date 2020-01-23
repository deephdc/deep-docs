Architecture overview
---------------------

There are several different components in the DEEP-HDC project that are relevant for the users. Later on you will see
how each :doc:`different type of user <user-roles>` can take advantage of the different components.


The marketplace
===============

The DEEP Marketplace is a catalogue of modules that every user can have access to. Modules can be:

* **Trained**: Those are modules that an user can train on their own data to create a new service. Like training an
  `image classifier <https://marketplace.deep-hybrid-datacloud.eu/modules/train-an-image-classifier.html>`_ with a
  plants dataset to create a `plant classifier <http://marketplace.deep-hybrid-datacloud.eu/modules/plants-species-classifier.html>`_
  service.
  Look for the ``trainable`` tag in the marketplace to find those modules.

* **Deployed for inference**: Those are modules that have been trained for a specific task (like the
  `plant classifier <http://marketplace.deep-hybrid-datacloud.eu/modules/plants-species-classifier.html>`_ mentioned earlier).
  Look for the ``inference`` and ``pre-trained`` tags in the marketplace to find those modules.

Some modules can both be trained and deployed for inference.
For example the `image classifier <https://marketplace.deep-hybrid-datacloud.eu/modules/train-an-image-classifier.html>`_
can be trained to create other image classifiers but can also be deployed for inference as it comes pre-trained with a
general image classifier.

For more information have a look at the `marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`_.


The API
=======

The DEEPaaS API is a key component for making the modules accessible to everybody (including non-experts), as it
provides a consistent and easy to use way to access to the model's functionality. It is available for both inference and training.

Advanced users that want to create new modules can make them :ref:`compatible with the API <user/overview/api:Integrate your model with the API>`
to make them available to the whole community. This can be easily done, with just some minor changes to the module itsef,
since it only requires adding additional entry points.

For more information take a look on the full :doc:`API guide <api>`.


The data storage resources
==========================

Storage is essential for user that want to create new services by training modules on their custom data. For the moment
we support hosting data into `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu>`_. We are currently working on adding
additional storage support, as well as more advanced features such as data caching (see `OneData <https://onedata.org/>`_),
in cooperation with the `eXtreme-DataCloud <http://www.extreme-datacloud.eu/>`_ project.


The dashboards
==============

The DEEP dashboards allow users to access computing Testbed resources to deploy, perform inference, and train their modules, also to deploy a more complex instances. There are two available options:

* `Training dashboard <https://train.deep-hybrid-datacloud.eu/>`_
    This dashboard allows you to interact with the modules hosted at the `DEEP Open Catalog <https://marketplace.deep-hybrid-datacloud.eu/>`_. It simplifies the deployment and hides some of the technical parts that most users do not need to worry about. Most of DEEP users would use this dashboard.

* `General purpose dashboard <https://paas.cloud.cnaf.infn.it/>`_
    This dashboard allows you to interact with the underling TOSCA templates instead of modules and deploy more complex topologies (e.g. kubernetes cluster) than the modules from the DEEP Open Catalog.

The dashboards allow a user to select:

* **The module** to run. A user can also run the development container to develop new modules as well a external
  docker images that are not hosted in the `deephdc organization <https://hub.docker.com/u/deephdc/>`_ (that is from modules not available in the
  Marketplace).
* **The computing resources** to have available. A user can select multiple CPUs and GPUs, the machine RAM as well as optionally choosing
  the physical site where the machine must be deployed.
* **The service** to run. Currently, options include running the :doc:`DEEPaaS API <api>` (recommended for fully
  developed modules than only need to be trained) and `JupyterLab <https://jupyterlab.readthedocs.io/en/stable/>`_
  (recommended for developing code as well for cases where access to the bash console is needed).

Once the machine is deployed the user will access a page listing all the current deployments.
