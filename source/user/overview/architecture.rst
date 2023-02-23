Architecture overview
=====================

There are several different components in the DEEP-HybridDataCloud project that are relevant for the users. Later on you will see
how each :doc:`different type of user <user-roles>` can take advantage of the different components.


The Marketplace
---------------

The `DEEP Marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`__ is a catalogue of modules that every user can have access to. Modules can be:

* **Trainable**: Those are modules that an user can train on their own data to create a new service. Like training an
  `image classifier <https://marketplace.deep-hybrid-datacloud.eu/modules/deep-oc-image-classification-tf.html>`__ with a
  plants dataset to create a `plant classifier <http://marketplace.deep-hybrid-datacloud.eu/modules/plants-species-classifier.html>`__
  service.
  Look for the ``trainable`` tag in the marketplace to find those modules.

* **Trained (inference-ready)**: Those are modules that have been pre-trained for a specific task (like the
  `plant classifier <https://marketplace.deep-hybrid-datacloud.eu/modules/deep-oc-plants-classification-tf.html>`__ mentioned earlier).
  Look for the ``inference`` and ``pre-trained`` tags in the marketplace to find those modules.

Some modules can both be trainable and trained.
For example the `image classifier <https://marketplace.deep-hybrid-datacloud.eu/modules/deep-oc-image-classification-tf.html>`__
can be trained to create other image classifiers but can also be deployed for inference as it comes pre-trained with a
general-purpose image classifier.


..
  TODO: uncomment when serverless is running back again

  DEEP as a Service
  -----------------

  `DEEP as a Service (or DEEPaaS) <https://deepaas.deep-hybrid-datacloud.eu/>`__ is a fully managed service that allows
  to easily and automatically deploy developed applications as services, with horizontal scalability thanks to a
  serverless approach. Module owners only need to care about the application development process, and incorporate
  new features that the automation system receives as an input.

  The serverless framework allows any user to automatically deploy from the browser any module in real time to try it.
  It only supports prediction. For training, which is more resource consuming, users must use the DEEP Training Dashboard.


The DEEPaaS API
---------------

The :doc:`DEEPaaS API <api>` is a key component for making the modules accessible to everybody (including non-experts), as it
provides a consistent and easy to use way to access the model's functionality. It is available for both inference and training.

Advanced users that want to create new modules can make them :ref:`compatible with the API <user/overview/api:Integrate your model with the API>`
to make them available to the whole community. This can be easily done, since it only requires minor changes in user's code.


The data storage resources
--------------------------

Storage is essential for user that want to create new services by training modules on their custom data. For the moment
we support hosting data in `DEEP-Nextcloud <https://data-deep.a.incd.pt/>`__ (up to 2 Terabytes by default), as well
as integration with popular cloud storage options like  `Google Drive <https://www.google.com/drive/>`__,
`Dropbox <https://www.dropbox.com/>`__, `Amazon S3 <https://aws.amazon.com/s3/>`__ and `many more <https://rclone.org/>`__.

We are currently exploring more advanced features such as data caching (see `OneData <https://onedata.org/>`__),
in cooperation with the `eXtreme-DataCloud <http://www.extreme-datacloud.eu/>`__ project.


The Dashboard
-------------

The :doc:`DEEP dashboard <dashboard>`. allow users to access computing resources to deploy, perform inference,
and train modules hosted at the `DEEP Open Catalog <https://marketplace.deep-hybrid-datacloud.eu/>`__, as well
as deploying external Docker images hosted in Dockerhub.
The Dashboard simplifies the deployment and hides some of the technical parts that most users do not need to worry about.
