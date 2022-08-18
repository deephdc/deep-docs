Our different user roles
========================

The DEEP-HybridDataCloud project is focused on three different types of users, depending on what you want to achieve you should enter into one or more of the following categories:


.. image:: ../../_static/DEEP_WP2-User_Viewpoint.png


The basic user
--------------

This user wants to use modules that are already pre-trained and :doc:`test them with their data <../howto/inference-locally>`,
and therefore don't need to have any machine learning knowledge. For example, they can take an already trained module
for `plant classification <https://marketplace.deep-hybrid-datacloud.eu/modules/deep-oc-plants-classification-tf.html>`_
that has been containerized, and use it to classify their own plant images.

**What DEEP can offer to you:**

* a :ref:`catalogue <user/overview/architecture:The marketplace>` full of ready-to-use modules to perform inference with your data
* an :ref:`API <user/overview/architecture:The DEEPaaS API>` to easily interact with the services
* solutions to run the inference in the Cloud or in your local resources
* the ability to develop complex topologies by composing different modules

**Related HowTo's:**

* :doc:`How to perform inference remotely <../howto/inference-remotely>`
* :doc:`How to perform inference locally <../howto/inference-locally>`


The intermediate user
---------------------

The intermediate user wants to :doc:`retrain an available module <../howto/train-model-locally>` to perform the same
task but fine tuning it to their own data.
They still might not need high level knowledge on modelling of machine learning problems, but typically do need basic
programming skills to prepare their own data into the appropriate format.
Nevertheless, they can re-use the knowledge being captured in a trained network and adjust the network to their problem
at hand by re-training the network on their own dataset.
An example could be a user who takes the generic `image classifier <https://marketplace.deep-hybrid-datacloud.eu/modules/deep-oc-image-classification-tf.html>`_
model and retrains it to perform `seed classification <https://marketplace.deep-hybrid-datacloud.eu/modules/deep-oc-seeds-classification-tf.html>`_.

**What DEEP can offer to you:**

* the ability to train out-of-the-box a module of the :ref:`catalogue <user/overview/architecture:The marketplace>` on your personal dataset
* an :ref:`API <user/overview/architecture:The DEEPaaS API>` to easily interact with the model
* data :ref:`storage resources <user/overview/architecture:The data storage resources>` to access your dataset
  (`DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu>`_, `OneData <https://onedata.org/>`_, ...)
* the ability to deploy the developed service on Cloud resources
* the ability to share the service with other users in the user's catalogue

**Related HowTo's:**

* :doc:`How to train a model locally <../howto/train-model-locally>`
* :doc:`How to train a model remotely <../howto/train-model-remotely>`


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
* data :ref:`storage resources <user/overview/architecture:The data storage resources>` to access your dataset
  (`DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu>`_, `OneData <https://onedata.org/>`_, ...)
* the ability to deploy the developed module on Cloud resources
* the ability to share the module with other users in the open :ref:`catalogue <user/overview/architecture:The marketplace>`
* the possibility to :ref:`integrate your module with the API <user/overview/api:Integrate your model with the API>`
  to enable easier user interaction


**Related HowTo's:**

* :doc:`How to use the DEEP Cookiecutter template for model development <cookiecutter-template>`
* :doc:`How to develop your own machine learning model <../howto/develop-model>`
* :ref:`How to integrate your model with the DEEPaaS API <user/overview/api:Integrate your model with the API>`
