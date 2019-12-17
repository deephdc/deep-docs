Architecture overview
---------------------

There are several different components in the DEEP-HDC project that are relevant for the users. Later on you will see
how each :doc:`different type of user <user-roles>` can take advantage of the different components.


The marketplace
===============

This is one of the most important components. It is a catalogue of modules that every user can have access to. We can
divide the modules in two different categories:

* **Models** are modules (eg. an `image classifier <https://marketplace.deep-hybrid-datacloud.eu/modules/deep-oc-image-classification-tensorflow.html>`_)
  that an user can train on their own data to create a new service. They have not already been trained to perform any particular task.

* **Services** are models that have been trained for a specific task (eg. an `plant classifier <https://marketplace.deep-hybrid-datacloud.eu/modules/deep-oc-plant-classification.html>`_).
  They usually derive from model, although sometimes, for very specific tasks, a module can be both a model and a service.

For more information have a look at the `marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`_.


The API
=======

The DEEPaaS API is a key component for making the modules accessible to everybody. It is available for both inference and training. Advanced users that want to create new modules can make them :ref:`compatible with the API <user/overview/api:Integrate your model with the API>`
to make them available to the whole community. This can be easily done, with just some minor changes to the module itsef, since it only requires adding additional entry points.

For more information take a look on the full :doc:`API guide <api>`.


The storage resources
=====================

Storage is essential for user that want to create new services by training models on their custom data. For the moment we
support saving data into `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu>`_. We are currently working on adding additional storage support, as well as more advanced features such as data caching, in cooperation with the `eXtreme-DataCloud <www.extreme-datacloud.eu>`_ project.
