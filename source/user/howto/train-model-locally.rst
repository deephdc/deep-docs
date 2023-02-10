.. include:: <isonum.txt>
.. highlight:: console

Train a model locally
=====================

.. admonition:: Useful video demos
   :class: important

    - `Running a module locally with docker <https://www.youtube.com/watch?v=3ORuymzO7V8&list=PLJ9x9Zk1O-J_UZfNO2uWp2pFMmbwLvzXa&index=13>`__

This is a step by step guide on how to train a module from the Marketplace with your own dataset, on your local machine.

In this tutorial we will see how to retrain a `generic image classifier <https://github.com/deephdc/DEEP-OC-image-classification-tf>`__
on a custom dataset to create a `phytoplankton classifier <https://github.com/deephdc/DEEP-OC-phytoplankton-classification-tf>`__.
If you want to follow along, you can download the toy phytoplankton dataset :fa:`download` `here <https://api.cloud.ifca.es:8080/swift/v1/public-datasets/phytoplankton-mini.zip>`__.

.. admonition:: Requirements

    * having `Docker <https://www.docker.com>`__ installed. For an up-to-date installation please follow
      the `official Docker installation guide <https://docs.docker.com/install>`__.
      As you will likely be using GPUs for training, you also have to install the `nvidia-container-toolkit <https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#installing-on-ubuntu-and-debian>`__
      to make them visible from inside the container.


1. Choose a module from the Marketplace
---------------------------------------

The first step is to choose a model from the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`__. Make sure to select a module with the ``trainable`` tag.
For educational purposes we are going to use a `general model to identify images <https://marketplace.deep-hybrid-datacloud.eu/modules/train-an-image-classifier.html>`__. This will allow us to see the general workflow.

Once we have chosen the model at the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`__ we will
find that it has an associated docker container in `DockerHub <https://hub.docker.com/u/deephdc/>`__. For example, in the
example we are running here, the container would be ``deephdc/deep-oc-image-classification-tf``. So let's pull the
docker image from DockerHub:

.. code-block::

    $ docker pull deephdc/deep-oc-image-classification-tf

Docker images have usually tags depending on whether they are using ``master`` or ``test`` and whether they use
``cpu`` or ``gpu``. Tags are usually:

* ``latest`` or ``cpu``: master + cpu
* ``gpu``: master + gpu
* ``cpu-test``: test + cpu
* ``gpu-test``: test + gpu

You tipically want to run your training on master with a gpu:

.. code-block::

    $ docker pull deephdc/deep-oc-image-classification-tf:gpu

.. tip::

    Instead of pulling from Dockerhub, it's also possible to build the image yourself:

    .. code-block::

        $ git clone https://github.com/deephdc/deep-oc-image-classification-tf
        $ cd deep-oc-image-classification-tf
        $ docker build -t deephdc/deep-oc-image-classification-tf .


2. Prepare your dataset
-----------------------

For this tutorial, we will assume that you also have your data stored locally.
If you have your data in a remote storage, check the :doc:`rclone docs <rclone>`
to see of you can copy them to your local machine.

When training a model, the data has usually to be in a specific format and folder structure.
It's usually helpful to read the README in the source code of the module
(in this case located `here <https://github.com/deephdc/image-classification-tf>`__)
to learn the correct way to setting it up.

In the case of the **image classification module**, we will create the following folders:

.. image:: ../../_static/nc-folders.png

* A folder called ``models`` where the new training weights will be stored after the training is completed
* A folder called ``data`` that contains two different folders:

    * The sub folder ``images`` containing the input images needed for the training
    * The sub folder ``dataset_files`` containing a couple of files:

        * ``train.txt`` indicating the relative path to the training images
        * ``classes.txt`` indicating which are the categories for the training

Again, the folder structure and their content will of course depend on the module to be used.
This structure is just an example in order to complete the workflow for this tutorial.


3. Run your module
------------------

When running the Docker container, you have to make sure that the data folder is
accessible from inside the container. This is done via the Docker volume ``-v`` flag:

.. code-block:: console

	$ docker run -ti -p 5000:5000 -p 6006:6006  -p 8888:8888 -v path_to_local_folder:path_to_docker_folder deephdc/deep-oc-image-classification-tf

We also need to make GPUs visible from inside the container using the ``--runtime=nvidia``
(or the ``--gpus all`` flag).

In our case, the final command, mounting the data folder and the model weights folder
(where we will later retrieve the newly trained model), looks as following:

.. code-block:: console

	$ docker run -ti -p 5000:5000 -p 6006:6006  -p 8888:8888 -v /home/ubuntu/data:/srv/image-classification-tf/data -v /home/ubuntu/models:/srv/image-classification-tf/models --runtime=nvidia deephdc/deep-oc-image-classification-tf:gpu


4. Open the DEEPaaS API and train the model
-------------------------------------------

Go to `<http://0.0.0.0:5000/ui>`__ and look for the ``train`` POST method.
Modify the training parameters you wish to change and execute.

If some kind of monitorization tool is available for this model you will be able to follow the training
progress from `<http://0.0.0.0:6006>`__.


5. Test and export the newly trained model
------------------------------------------

Once the training has finished, you can directly test it by clicking on the ``predict`` POST method.
For this you have to kill the process running deepaas, and launch it again.

.. code-block:: console

    $ kill -9 $(ps aux | grep '[d]eepaas-run' | awk '{print $2}')
    $ kill -9 $(ps aux | grep '[t]ensorboard' | awk '{print $2}')  # optionally also kill monitoring process

This is because the user inputs for deepaas are generated at the deepaas launching.
Thus it is not aware of the newly trained model. Once deepaas is restarted, head to the
``predict`` POST method, select you new model weights and upload the image your want to classify.

If you are satisfied with your model, then it's time to save it into your remote storage,
so that you still have access to it if your machine is deleted.
For this we have to create a ``tar`` file with the model folder (in this case, the foldername is
the timestamp at which the training was launched) so that we can download in our Docker container.

For the next step, you need to make them `publicly available <https://docs.nextcloud.com/server/latest/user_manual/en/files/sharing.html>`__
through an URL so they can be downloaded in your Docker container.
In Nextcloud, go to the ``tar`` file you just created:
:fa:`share-nodes` ➜ Share Link ➜ :fa:`square-plus` (Create a new share link)

6. Next steps
-------------

The next steps are common with the :ref:`remote training tutorial <user/howto/train-model-remotely:7. Create a Docker repo for your new module>`.
