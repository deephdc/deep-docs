.. include:: <isonum.txt>

.. highlight:: console


Train a model locally
---------------------

This is a step by step guide on how to train a model from the Marketplace with your own dataset.


1. Get Docker
=============

The first step is having `Docker <https://www.docker.com>`_ installed. To have an up-to-date installation please follow
the `official Docker installation guide <https://docs.docker.com/install>`_.


1. Search for a model in the marketplace
========================================

The first step is to choose a model from the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`_.  For educational purposes we are going to use a `general model to identify images <https://marketplace.deep-hybrid-datacloud.eu/modules/deep-oc-image-classification-tensorflow.html>`_. This will allow us to see the general workflow.



3. Get the model
================

.. todo:: Check that names of the docker containers are correct for the image classifier example.

Once we have chosen the model at the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`_ we will find that it has an associated docker container in `DockerHub <https://hub.docker.com/u/deephdc/>`_. For example, in the example we are running here, the container would be deephdc/deep-oc-image-classification-tf. This means that to pull the docker image and run it you should ::

.. code-block:: console

    $ docker pull deephdc/deep-oc-image-classification-tf



4. Export rclone.conf file

When running the container you should export the rclone.conf file so that it can be reached from within the docker. You can see an example on how to do this here::


	$ docker run -ti -v  -p 5000:5000 -p 6006:6006 -v  host_path_to_rclone.conf:/root/.config/rclone/rclone.conf <your-docker-image>

You can see this last step explained more in detail :doc:`here <rclone>`.

We are using the port ``5000`` to deploy the API and the port ``6006`` to monitor the training (for example using
`Tensorboard <https://www.tensorflow.org/guide/summaries_and_tensorboard>`_).
	
4. Upload your data to storage resources
========================================

To run locally you have two options:

1. Have your data stored locally

You should make sure that you export inside of the container all the folders you need for the training::

	$ docker run -ti -v  -p 5000:5000 -p 6006:6006 -v path_to_local_folder:path_to_docker_folder -v host_path_to_rclone.conf:/root/.config/rclone/rclone.conf <your-docker-image>

2. Have your data at a remote storage resource

* Nextcloud

If you have the files you need for the training stored in Nextcloud you need first to login into `DEEP-Nextcloud  <https://nc.deep-hybrid-datacloud.eu/login>`_ with your DEEP-IAM credentials. Then you have to go to: **(1) Settings (top right corner)** |rarr|  **(2) Security**  |rarr|  **(3) Devices & sessions**


.. image:: ../../_static/nc-access.png

Set a name for your application (for this example it will be **deepnc**) and clik on **Create new app password**. This will generate <your_nextcloud_username> and <your_nextcloud_password> that you should to include in your rclone.conf file (see  :doc:`more details <rclone>`.).

Now you can create the folders that you need in order to store the inputs needed for the training and to retrieve the output. In order to be able to see these folders locally you should run either on your local host or on the docker container::

	import subprocess

	# from deep-nextcloud into the container
	command = (['rclone', 'copy', 'deepnc:/path_to_remote_nextcloud_folder', 'path_to_local_folder'])

	results= subprocess.Popen(command, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
	output, error = result.communicate()

* Google Drive

TBC

* Dropbox

TBC

5. Train the model
==================

Now comes the fun! Go to `<http://0.0.0.0:5000>`_ and look for the ``train`` method. Modify the training parameters you wish to
change and execute. If some kind of monitorization tool is available for this model you will be able to follow the training
progress from `<http://0.0.0.0:6006>`_.


6. Testing the training
=======================

Once the training has finished, you can directly test it by clicking on the ``predict`` method. There you can either upload the image your want to classify or give a URL to it. 
