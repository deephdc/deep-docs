**********************
Architecture overview
**********************

There are several different components in the DEEP-HybridDataCloud project that are relevant for the users. Later on you will see
how each :doc:`different type of user <user-roles>` can take advantage of the different components.


The marketplace
===============

The DEEP Marketplace is a catalogue of modules that every user can have access to. Modules can be:

* **Trained**: Those are modules that an user can train on their own data to create a new service. Like training an
  `image classifier <https://marketplace.deep-hybrid-datacloud.eu/modules/deep-oc-image-classification-tf.html>`_ with a
  plants dataset to create a `plant classifier <http://marketplace.deep-hybrid-datacloud.eu/modules/plants-species-classifier.html>`_
  service.
  Look for the ``trainable`` tag in the marketplace to find those modules.

* **Deployed for inference**: Those are modules that have been pre-trained for a specific task (like the
  `plant classifier <https://marketplace.deep-hybrid-datacloud.eu/modules/deep-oc-plants-classification-tf.html>`_ mentioned earlier).
  Look for the ``inference`` and ``pre-trained`` tags in the marketplace to find those modules.

Some modules can both be trained and deployed for inference.
For example the `image classifier <https://marketplace.deep-hybrid-datacloud.eu/modules/deep-oc-image-classification-tf.html>`_
can be trained to create other image classifiers but can also be deployed for inference as it comes pre-trained with a
general image classifier.

For more information have a look at the `marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`_.


DEEP as a Service
=================

`DEEP as a Service (or DEEPaaS) <https://deepaas.deep-hybrid-datacloud.eu/>`_ is a fully managed service that allows
to easily and automatically deploy developed applications as services, with horizontal scalability thanks to a
serverless approach. Module owners only need to care about the application development process, and incorporate
new features that the automation system receives as an input.  The serverless framework  allows any user to
automatically deploy from the browser any module in real time to try it. We only allow trying prediction, for training,
which is more resource consuming, users must use the DEEP Training Dashboard.


The DEEPaaS API
===============

The :doc:`DEEPaaS API <api>` is a key component for making the modules accessible to everybody (including non-experts), as it
provides a consistent and easy to use way to access the model's functionality. It is available for both inference and training.

Advanced users that want to create new modules can make them :ref:`compatible with the API <user/overview/api:Integrate your model with the API>`
to make them available to the whole community. This can be easily done, since it only requires minor changes in user's code and
adding additional entry points.

For more information take a look on the full :doc:`API guide <api>`.


The data storage resources
==========================

Storage is essential for user that want to create new services by training modules on their custom data. For the moment
we support hosting data in `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu>`_. We are currently working on adding
additional storage support, as well as more advanced features such as data caching (see `OneData <https://onedata.org/>`_),
in cooperation with the `eXtreme-DataCloud <http://www.extreme-datacloud.eu/>`_ project.


The dashboards
==============

The DEEP dashboard allow users to access computing resources to deploy, perform inference, and train their modules.
To be able to access the Dashboard you need `IAM credentials <https://iam.deep-hybrid-datacloud.eu/>`_.
There are two versions of the Dashboard:

* `Training dashboard <https://train.deep-hybrid-datacloud.eu/>`_
    This dashboard allows you to interact with the modules hosted at the `DEEP Open Catalog <https://marketplace.deep-hybrid-datacloud.eu/>`_,
    as well as deploying external Docker images hosted in Dockerhub. It simplifies the deployment and hides some of
    the technical parts that most users do not need to worry about. *Most of DEEP users would use this dashboard*.

* `General purpose dashboard <https://paas.cloud.cnaf.infn.it/>`_
    This dashboard allows you to interact with the underling `TOSCA templates <https://github.com/indigo-dc/tosca-templates/tree/master/deep-oc>`_
    (which configure the job requirements) instead of modules and deploy more complex topologies (e.g. a kubernetes cluster).
    Modules can either use a `general template <https://github.com/indigo-dc/tosca-templates/blob/master/deep-oc/deep-oc-marathon-webdav.yml>`_
    or create a dedicated one based on the `existing templates <https://github.com/indigo-dc/tosca-templates/tree/master/deep-oc>`__.

For more information take a look on the full :doc:`Dashboard guide <api>`.
