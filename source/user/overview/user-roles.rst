Our different user roles
========================

The DEEP-HybridDataCloud project is focused on three different types of users, depending on what you want to achieve you should enter into one or more of the following categories:


.. image:: ../../_static/DEEP_WP2-User_Viewpoint.png


The basic user
--------------

This user wants to use the services that have been already developed and :doc:`test them with their data <../howto/try-service-locally>`,
and therefore don't need to have any machine learning knowledge. For example, they can take an already trained network
for `plant classification <https://marketplace.deep-hybrid-datacloud.eu/modules/deep-oc-plant-classification.html>`_
that has been containerized, and use it to classify their own plant images.

**What DEEP can offer to you:**

* a :ref:`catalogue <user/overview/architecture:The marketplace>` full of ready-to-use services to make inference with your data
* an :ref:`API <user/overview/architecture:The API>` to easily interact with the services
* solutions to run the inference in the Cloud or in your local resources
* the ability to develop complex topologies by composing different modules


The intermediate user
---------------------

The intermediate user wants to :doc:`retrain an available model <../howto/train-model-locally>` to perform the same task but fine
tuning it to their own data.
They still might not need high level knowledge on modelling of machine learning problems, but typically do need basic
programming skills to prepare their own data into the appropriate format.
Nevertheless, they can re-use the knowledge being captured in a trained network and adjust the network to their problem
at hand by re-training the network on their own dataset.
An example could be a user who takes the generic `image classifier <https://marketplace.deep-hybrid-datacloud.eu/modules/deep-oc-image-classification-tensorflow.html>`_
model and retrain it to perform `seed classification <https://marketplace.deep-hybrid-datacloud.eu/modules/deep-oc-seeds-classification.html>`_.

**What DEEP can offer to you:**

* the ability to train out-of-the-box a model of the :ref:`catalogue <user/overview/architecture:The marketplace>` on your personal dataset
* an :ref:`API <user/overview/architecture:The API>` to easily interact with the model
* access to data :ref:`storage resources <user/overview/architecture:The storage resources>` via
  `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu>`_ to save your dataset
* the ability to deploy the developed service on Cloud resources
* the ability to share the service with other users in the user's catalogue


The advanced user
-----------------

The advanced users are the ones that will :doc:`develop their own machine learning models <../howto/develop-model>`
and therefore need to be competent in machine learning. This would be the case for example if we provided an image
classification model but the users wanted to perform object localization, which is a fundamentally different task.
Therefore they will design their own neural network architecture, potentially re-using parts of the code from other
models.

**What DEEP can offer to you:**

* a ready-to-use environment with the main DL frameworks running in a dockerized solution running on different types of
  hardware (CPUs, GPUs, etc)
* access to data :ref:`storage resources <user/overview/architecture:The storage resources>` via
  `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu>`_ to save your dataset
* the ability to deploy the developed module on Cloud resources
* the ability to share the module with other users in the user's :ref:`catalogue <user/overview/architecture:The marketplace>`
* the possibility to :ref:`integrate your module <user/overview/api:Integrate your model with the API>` with
  the :ref:`API <user/overview/architecture:The API>` to enable easier user interaction
