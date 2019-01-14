Toy example: dog's breed detection
==================================
A toy example to identify Dog's breed, "Dogs breed detector" [*]_, images for training come from `dog dataset <https://s3-us-west-1.amazonaws.com/udacity-aind/dog-project/dogImages.zip>`_.

+----------------------------+---------------------------+
| Neural Network type        |         CNN               |
+----------------------------+---------------------------+
| Deep Learning Framework(s) |    Tensorflow, Keras      |
+----------------------------+---------------------------+
| Programming language       |        Python             |
+----------------------------+---------------------------+
|  GPU version               |         yes               |
+----------------------------+---------------------------+
|  CPU version               |         yes               |
+----------------------------+---------------------------+
|  DEEPaaS API               |         yes               |
+----------------------------+---------------------------+
|  DEEP UC template          |         yes               |
+----------------------------+---------------------------+
|  DEEP Nextcloud access     |         yes               |
+----------------------------+---------------------------+


Keywords: image classification, CNN, transfer learning, Tensorflow

DEEP-OC Dockerfile: https://github.com/indigo-dc/DEEP-OC-dogs_breed_det

App source code: https://github.com/indigo-dc/dogs_breed_det

Pre-trained weights: https://nc.deep-hybrid-datacloud.eu/s/D7DLWcDsRoQmRMN 


Description
-----------

The project applies `Transfer learning <https://en.wikipedia.org/wiki/Transfer_learning>`_ for dog's breed identification, implemented with Tensorflow and Keras:
From a pre-trained model (VGG16 | VGG19 | Resnet50 | InceptionV3 | Xception) the last layer is removed, 
then a new Fully Connected (FC) classification layer is added, which is trained. 
All images first pass through the pre-trained network and converted into the tensor with the shape of the 'before-last' layer of the pre-trained network, 
into so-called 'bottleneck_features'. These bottleneck_features are used then as input for the FC classification network.


Local Workflow
---------------
The described workflow supposes usage of downloaded from DEEP Open Catalogue Docker images, i.e. you need either 
`docker <https://docs.docker.com/install/#supported-platforms>`_ or `udocker <https://github.com/indigo-dc/udocker/releases>`_ tool.

**1. Workflow intro**

1) DEEPaaS API uses port 5000 for access, one therefore has to map the container and host ports, see Examples below.

2) Following two directories inside the docker container are used for input and output:

+--------------------------------+----------------------------------------+
| **Directory inside container** |             **Description**            |
+--------------------------------+----------------------------------------+
| ``/srv/dogs_breed_det/data``   | raw data, ready-for-training data, etc |
+--------------------------------+----------------------------------------+
| ``/srv/dogs_breed_det/models`` | for the trained weights                |
+--------------------------------+----------------------------------------+

If you want to perform full training, then you need to mount your data into ``/srv/dogs_breed_det/data``.

If you want only to classify images or want to keep trained weights in the persistant place, then you have to mount ``/srv/dogs_breed_det/models`` 
to a persistent volume. You can use either your local directories or connect your remote storage, see Examples below.

**2. Data input**

Original dataset consists of dog's images for 133 breeds. The images are compressed in 
one `dogImages.zip <https://s3-us-west-1.amazonaws.com/udacity-aind/dog-project/dogImages.zip>`_  file. 
The archive contains three directories for ``train``, ``test``, and ``valid`` datasets:
::
    test/
        001.Affenpinscher/
        002.Afghan_hound/
        ...
    train/
        001.Affenpinscher/
        002.Afghan_hound/
        ...
    valid/
        001.Affenpinscher/
        002.Afghan_hound/
        ...

These directories are automatically de-archived in ``/srv/dogs_breed_det/data/dogImages/``. 

Trainig labels are also created automatically based on the directory names, truncating leading numbers, e.g. '002.'.

**The minimum requirement for training is to place this** `dogImages.zip <https://s3-us-west-1.amazonaws.com/udacity-aind/dog-project/dogImages.zip>`_ 
**in** ``/srv/dogs_breed_det/data/raw/`` **directory.**

If you want to use your own dataset then it has to follow similar structure.


**3. Train the classifier**

**4. Test the classifier**


DEEP Pilot infrastructure submission
------------------------------------


Examples
--------

**1. Mount local host directories**

Example 1 (GPU, default):
::

    docker run -ti -p 5000:5000 -v ~/data:/srv/dogs_breed_det/data \
    -v ~/models:/srv/dogs_breed_det/models \
    deephdc/deep-oc-dogs_breed_det deepaas-run --listen-ip=0.0.0.0

Example 2 (CPU):
::

    docker run -ti -p 5000:5000 -v ~/data:/srv/dogs_breed_det/data \
    -v ~/models:/srv/dogs_breed_det/models \
    deephdc/deep-oc-dogs_breed_det:cpu deepaas-run --listen-ip=0.0.0.0


**2. Connect to remote storage by using rclone.conf from your host**

`rclone <https://rclone.org/>`_ tool allows to connect to a plenty of remote storages. 
The tool is already installed in the Docker image and expects your ``data/`` and ``models/`` sub-directories to be under ``deepnc:/Datasets/dogs_breed/``.
If no data found in your container, rclone attempts to connect to ``deepnc:/`` and download necessary data from there.

If you are familiar with the `rclone <https://rclone.org/>`_ tool, you probably have ``rclone.conf`` file on your host. 
You can rename one of the pre-configured remote storages to ``deepnc`` and mount the host directory with the ``rclone.conf`` file into the container:

Example 3:
::

    docker run -ti -p 5000:5000 -v $HOSTDIR_WITH_RCLONE_CONF:/srv/rclone \
    -e RCLONE_CONFIG=/srv/rclone/rclone.conf \
    deephdc/deep-oc-dogs_breed_det:cpu deepaas-run --listen-ip=0.0.0.0

On your remote storage create following directories:

* ``/Datasets/dogs_breed/data``
* ``/Datasets/dogs_breed/data/raw``
* ``/Datasets/dogs_breed/models``


put `dogImages.zip <https://s3-us-west-1.amazonaws.com/udacity-aind/dog-project/dogImages.zip>`_  file in ``/Datasets/dogs_breed/data/raw``

Example 4: ``rclone.conf`` with `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu/>`_ configured as ``deepnc`` remote storage:
::
    [deepnc]
    type = webdav
    url = https://nc.deep-hybrid-datacloud.eu/remote.php/webdav/
    vendor = nextcloud
    user = DEEP-IAM-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    pass = YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY


Example 5: ``rclone.conf`` with Google Drive configured as ``deepnc`` remote storage:
::
    [deepnc]
    type = drive
    scope = drive
    token = {"access_token":"ya29.XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX","token_type":"Bearer","refresh_token":"1/-XXXXXXXXXXXXXXXXXXXX","expiry":"2019-01-14T20:26:13.21767343Z"}


Check `rclone <https://rclone.org/>`_ documentation on how to configure different types of remote storage.

**3. Connect to remote storage by passing rclone configuration parameters as environment settings**

It is also possible to pass necessary configuration as environment settings during instantiation of the container, 
best is to create a runnable bash script:

Example 6: Connecting `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu/>`_ remote storage

.. code-block:: bash

    #!/bin/bash

    rclone_conf="/srv/.rclone.conf"
    rclone_url=https://nc.deep-hybrid-datacloud.eu/remote.php/webdav/
    rclone_vendor=nextcloud
    rclone_user=DEEP-IAM-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    rclone_pass=YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY

    docker run -ti -p 5000:5000 -e RCLONE_CONFIG=$rclone_conf \
       -e RCLONE_CONFIG_DEEPNC_TYPE="webdav" \
       -e RCLONE_CONFIG_DEEPNC_VENDOR="nextcloud" \
       -e RCLONE_CONFIG_DEEPNC_URL=$rclone_url \
       -e RCLONE_CONFIG_DEEPNC_USER=$rclone_user \
       -e RCLONE_CONFIG_DEEPNC_PASS=$rclone_pass \
       deephdc/deep-oc-dogs_breed_det:cpu deepaas-run --listen-ip=0.0.0.0


.. [*] Dogs breed detector is originally forked from `udacity/dogs-project <https://github.com/udacity/dog-project>`_
