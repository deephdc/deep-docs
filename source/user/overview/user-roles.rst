Our different user roles
========================

DEEPaaS is focused with three different types of users in mind, whcih vary in their machine learning knowledge and their
usage of DEEPaaS.


The basic user
--------------

This user wants to use the tools that have been already developed and :doc:`test them with their data <try-model-locally>`,
and therefore don't need to have any machine learning knowledge. For example, they can take an already trained network
for plant classification that has been containerized, and use it to classify their own plant images.

**What DEEPaaS can offer to you:**

* a :ref:`catalogue <user/overview/architecture:The marketplace>` full of ready-to-use modules to make inference with your data
* an :ref:`API <user/overview/architecture:The API>` to easily interact with the modules
* solutions to run the inference in the Cloud or in your local resources
* the ability to develop complex topologies by composing different modules


The intermediate user
---------------------

The intermediate user wants to :doc:`retrain an available model <train-model-locally>` to perform the same task but fine
tuning it to their own data.
They still might not need high level knowledge on modelling of machine learning problems, but typically do need basic
programming skills to prepare their own data into the appropriate format.
Nevertheless, they can re-use the knowledge being captured in a trained network and adjust the network to their problem
at hand by re-training the network on their own dataset.
An example could be a user who takes the plant classification tool but retrains it to perform animal classification.

**What DEEPaaS can offer to you:**

* the ability to train out-of-the-box a model of the :ref:`catalogue <user/overview/architecture:The marketplace>` on your personal dataset
* an :ref:`API <user/overview/architecture:The API>` to easily interact with the model
* access to data :ref:`storage resources <user/overview/architecture:The storage resources>` via NextCloud to save your dataset
* the ability to deploy the developed tool on Cloud resources
* the ability to share the tool with other users in the user's catalogue


The advanced user
-----------------

The advanced users are the ones that will :doc:`develop their own machine learning models <cookiecutter-develop-model>`
and therefore need to be competent in machine learning. This would be the case for example if we provided an image
classification tool but the users wanted to perform object localisation, which is a fundamentally different task.
Therefore they will design their own neural network architecture, potentially re-using parts of the code from other
tools.

**What DEEPaaS can offer to you:**

* a ready-to-use environment with the main DL frameworks running in a dockerized solution running on different types of
  hardware (CPUs, GPUs, etc)
* access to data :ref:`storage resources <user/overview/architecture:The storage resources>` via NextCloud to save your dataset
* the ability to deploy the developed module on Cloud resources
* the ability to share the module with other users in the user's catalogue
* the possibility to :ref:`integrate your module <user/overview/api:Integrate your model with the API>` with
  the :ref:`API <user/overview/architecture:The API>` to enable easier interaction.
