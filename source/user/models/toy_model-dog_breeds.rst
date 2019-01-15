Toy example: dog's breed detection
==================================
A toy example to identify Dog's breed, "Dogs breed detector" [*]_, images for training come from `dog dataset <https://s3-us-west-1.amazonaws.com/udacity-aind/dog-project/dogImages.zip>`_.

+-----------------------------------------------------------------+---------------------+
| Neural Network type                                             |         CNN         |
+-----------------------------------------------------------------+---------------------+
| Deep Learning Framework(s)                                      |  Tensorflow, Keras  |
+-----------------------------------------------------------------+---------------------+
| Programming language                                            |      Python         |
+-----------------------------------------------------------------+---------------------+
|  GPU version                                                    |        yes          |
+-----------------------------------------------------------------+---------------------+
|  CPU version                                                    |        yes          |
+-----------------------------------------------------------------+---------------------+
| `DEEPaaS API <https://deepaas.readthedocs.io/en/stable/>`_      |        yes          |
+-----------------------------------------------------------------+---------------------+ 
| :doc:`DEEP UC template <../overview/cookiecutter-template>`     |        yes          |
+-----------------------------------------------------------------+---------------------+
| `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu/>`_ access |        yes          |
+-----------------------------------------------------------------+---------------------+


Keywords: image classification, CNN, transfer learning, Tensorflow

DEEP-OC Dockerfile: https://github.com/indigo-dc/DEEP-OC-dogs_breed_det

App source code: https://github.com/indigo-dc/dogs_breed_det

Pre-trained weights: https://nc.deep-hybrid-datacloud.eu/s/D7DLWcDsRoQmRMN 



Description
-----------

The project applies `Transfer learning <https://en.wikipedia.org/wiki/Transfer_learning>`_ for dog's breed identification, implemented with Tensorflow and Keras:
From a pre-trained model (VGG16 | VGG19 | Resnet50 | InceptionV3) the last layer is removed, 
then a new Fully Connected (FC) classification layer is added, which is trained. 
All images first pass through the pre-trained network and converted into the tensor with the shape of the 'before-last' layer of the pre-trained network, 
into so-called 'bottleneck_features'. These bottleneck_features are used then as input for the FC classification network.


Local Workflow
---------------
The described workflow supposes usage of downloaded from `DEEP Open Catalog <https://marketplace.deep-hybrid-datacloud.eu/>`_ Docker images, 
i.e. you need either `docker <https://docs.docker.com/install/#supported-platforms>`_ or `udocker <https://github.com/indigo-dc/udocker/releases>`_ tool.

1. Workflow intro
""""""""""""""""""

a. `DEEPaaS API <https://deepaas.readthedocs.io/en/stable/>`_ uses port 5000 for access, one therefore has to map the container and host ports, 
see :ref:`sec-examples`.

b. Following two directories inside the docker container are used for input and output:

+--------------------------------+----------------------------------------+
| **Directory inside container** |             **Description**            |
+--------------------------------+----------------------------------------+
| ``/srv/dogs_breed_det/data``   | raw data, ready-for-training data, etc |
+--------------------------------+----------------------------------------+
| ``/srv/dogs_breed_det/models`` | for the trained weights                |
+--------------------------------+----------------------------------------+

If you want to perform full training, then you need to mount your data into ``/srv/dogs_breed_det/data``.

If you want to keep trained weights in the persistant place, then you have to mount ``/srv/dogs_breed_det/models`` 
to a persistent volume. You can either use your local directories or connect your remote storage, see :ref:`sec-examples`.

2. Data input
""""""""""""""

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

Training labels are also created automatically based on the directory names, truncating leading numbers, e.g. '002.'.

**The minimum requirement for training is to make** `dogImages.zip <https://s3-us-west-1.amazonaws.com/udacity-aind/dog-project/dogImages.zip>`_ 
**available in** ``/srv/dogs_breed_det/data/raw/`` **directory.**

If you want to use your own dataset then it has to follow similar structure.


**If local directories are mounted into the container, the following directory structure is suggested:**
::
    data/
        dogImages/
        raw/
    models/

+------------------------+------------------------------------------+
| **Local dir to mount** | **Corresponding place in the container** |
+------------------------+------------------------------------------+
| ``LOCAL_DIR/data``     | ``/srv/dogs_breed_det/data``             |
+------------------------+------------------------------------------+
| ``LOCAL_DIR/models``   | ``/srv/dogs_breed_det/models``           |
+------------------------+------------------------------------------+

In the 'local' case, you place `dogImages.zip <https://s3-us-west-1.amazonaws.com/udacity-aind/dog-project/dogImages.zip>`_ in ``LOCAL_DIR/data/raw``, 
which makes it available in ``/srv/dogs_breed_det/data/raw``.

**If you connect a remote storage, the following directories have to be created there:**
::
    /Datasets/dogs_breed/data
    /Datasets/dogs_breed/data/dogImages
    /Datasets/dogs_breed/data/raw
    /Datasets/dogs_breed/models

In the 'remote' case, you place `dogImages.zip <https://s3-us-west-1.amazonaws.com/udacity-aind/dog-project/dogImages.zip>`_ in ``/Datasets/dogs_breed/data/raw``, 
which makes it available in ``/srv/dogs_breed_det/data/raw``.


3. Accessing application
""""""""""""""""""""""""

* In a minimum case to classify images with already trained Resnet50 model, start the container as::

    docker run -ti -p 5000:5000 deephdc/deep-oc-dogs_breed_det:cpu deepaas-run --listen-ip=0.0.0.0
    
    
* In more advanced cases (see :ref:`sec-examples`) you may need to mount various directories or pass environment settings.
    
* Direct your web browser to http://127.0.0.1:5000


4. Test the classifier
"""""""""""""""""""""""

* Go to **/models/{model_name}/predict** , click "**Try it out**" button

* Choose an image file for dog's breed identification (N.B. "URL to retrieve data" is not (yet) implemented)

* Type **model_name**, one of the ``Dogs_Resnet50``, ``Dogs_InceptionV3``, ``Dogs_VGG16``, ``Dogs_VGG19`` 

* The equivalent API call is::

    curl -X POST "http://127.0.0.1:5000/models/Dogs_Resnet50/predict" -H "accept: application/json" -H "Content-Type: multipart/form-data" -F "data=@YourDogImage.jpg;type=image/jpeg"

.. note:: By default only weigths for Dogs_Resnet50 are available (automatically downloaded from the shared link, see above "Pre-trained weights" URL), all other models have to be trained first!


5. Train the classifier
"""""""""""""""""""""""

* Connect your data storage with the corresponding directory inside the container (see "Data input" above and :ref:`sec-examples` below)
* Go to **/models/{model_name}/train** , click "**Try it out**" button
* Type **model_name**, one of the ``Dogs_Resnet50``, ``Dogs_InceptionV3``, ``Dogs_VGG16``, ``Dogs_VGG19``
* Execute training
* The equivalent API call is::

    curl -X PUT "http://127.0.0.1:5000/models/Dogs_Resnet50/train" -H "accept: application/json"


DEEP Pilot infrastructure submission
------------------------------------

Please, refer to :doc:`Quickstart Guide <../quickstart>`, section "Run model on DEEP Pilot infrastructure", 
on what is required to start the application on DEEP Pilot infrastructure.

.. _sec-examples:

Examples
--------

Mount local host directories
"""""""""""""""""""""""""""""

**Example 1 (GPU, default):**
::

    docker run -ti -p 5000:5000 -v ~/data:/srv/dogs_breed_det/data \
    -v ~/models:/srv/dogs_breed_det/models \
    deephdc/deep-oc-dogs_breed_det deepaas-run --listen-ip=0.0.0.0

**Example 2 (CPU):**
::

    docker run -ti -p 5000:5000 -v ~/data:/srv/dogs_breed_det/data \
    -v ~/models:/srv/dogs_breed_det/models \
    deephdc/deep-oc-dogs_breed_det:cpu deepaas-run --listen-ip=0.0.0.0


Connecting remote storage by using ``rclone.conf`` from your host
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

`rclone <https://rclone.org/>`_ tool allows to connect to a plenty of remote storages. 
The tool is already installed in the Docker image and expects your ``data/`` and ``models/`` sub-directories to be under ``deepnc:/Datasets/dogs_breed/``.
If no data found in your container, rclone attempts to connect to ``deepnc:/`` and download necessary data from there.

If you are familiar with the `rclone <https://rclone.org/>`_ tool, you probably have ``rclone.conf`` file on your host. 
You can rename one of the pre-configured remote storages to ``deepnc``, then mount host directory with your ``rclone.conf`` file into the container:

**Example 3:** using in the container ``rclone.conf`` from your host
::

    docker run -ti -p 5000:5000 -v $HOSTDIR_WITH_RCLONE_CONF:/srv/rclone \
    -e RCLONE_CONFIG=/srv/rclone/rclone.conf \
    deephdc/deep-oc-dogs_breed_det:cpu deepaas-run --listen-ip=0.0.0.0

`dogImages.zip <https://s3-us-west-1.amazonaws.com/udacity-aind/dog-project/dogImages.zip>`_  file is expected to be in ``/Datasets/dogs_breed/data/raw``

**Example 4:** ``rclone.conf`` with `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu/>`_ configured as ``deepnc`` remote storage:
::
    [deepnc]
    type = webdav
    url = https://nc.deep-hybrid-datacloud.eu/remote.php/webdav/
    vendor = nextcloud
    user = DEEP-IAM-XXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
    pass = YYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYYY


**Example 5:** ``rclone.conf`` with Google Drive configured as ``deepnc`` remote storage:
::
    [deepnc]
    type = drive
    scope = drive
    token = {"access_token":"ya29.XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX","token_type":"Bearer","refresh_token":"1/-XXXXXXXXXXXXXXXXXXXX","expiry":"2019-01-14T20:26:13.21767343Z"}


.. note:: Check `rclone <https://rclone.org/>`_ documentation on how to configure different types of remote storage.

Connecting remote storage by passing rclone configuration as environment settings
"""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

It is also possible to pass necessary rclone configuration parameters as environment settings during instantiation of the container, 
best is to create a runnable bash script:

**Example 6:** connecting `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu/>`_ remote storage

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
