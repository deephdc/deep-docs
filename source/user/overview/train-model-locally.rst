Train a model locally
---------------------

This is a step by step guide on how to train a model from the Marketplace with your own dataset.
You can also look at `video demo <https://www.youtube.com/watch?v=Mh6rdlqX-7I&feature=youtu.be>`_ on how to do this.

1. Get Docker
=============

The first step is having `Docker <https://www.docker.com>`_ installed. To have an up-to-date installation please follow
the `official Docker installation guide <https://docs.docker.com/install>`_.

2. Search for a model in the marketplace
========================================

.. todo:: Add link to 'model' tag in the marketplace, add link to image classifier

The next step is to look for a model in the marketplace you want to retrain on your own data.
Possible option include image classifiers, etc.

3. Get the model
================

.. todo:: Check that names of the docker containers are correct for the image classifier example.

You will find that your model has an associate Docker container in DockerHub. Please download and run the container with:

.. code-block:: console

    $ docker pull dockerhub_url
    $ docker run -p 5000:5000 -p 6006:6006 -ti container_name /bin/bash

For example if you wanted to download the image classifier model you would have to run:

.. code-block:: console

    $ docker pull https://hub.docker.com/r/deephdc/deep-oc-image-classification-tf
    $ docker run -p 5000:5000 -p 6006:6006 -ti imgclas-tf-normal /bin/bash

We are using the port ``5000`` to deploy the API and the port ``6006`` to monitor the training (for example using
`Tensorboard <https://www.tensorflow.org/guide/summaries_and_tensorboard>`_).

4. Upload your data to storage resources
========================================

.. todo:: Lara and Valentin: Fill this section. Look at the possibility of using rclone with Google Drive, Dropbox, etc.

5. Train the model
==================

Now comes the fun! Go to `<http://0.0.0.0:5000>`_ and look for the train mehod. Modify the training parameters you wish to
change and execute. If some kind of monitorization tool is available for this model you will be able to folllow the training
progress from `<http://0.0.0.0:6006>`_.

