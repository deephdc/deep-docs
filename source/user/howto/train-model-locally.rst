.. include:: <isonum.txt>
.. highlight:: console

Train a model locally
=====================

.. admonition:: Useful video demos
   :class: important

    - `Running a module locally with docker <https://www.youtube.com/watch?v=3ORuymzO7V8&list=PLJ9x9Zk1O-J_UZfNO2uWp2pFMmbwLvzXa&index=13>`__

This is a step by step guide on how to train a module from the Marketplace with your own dataset.

.. admonition:: Requirements

    * having `Docker <https://www.docker.com>`_ installed. For an up-to-date installation please follow
      the `official Docker installation guide <https://docs.docker.com/install>`_.
    * **Optional**: having a `DEEP IAM <https://iam.deep-hybrid-datacloud.eu/>`_ account if you want to use remote
      storage resources.


1. Choose your module
---------------------

The first step is to choose a module from the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`_.
For educational purposes we are going to use a `general model to identify images <https://marketplace.deep-hybrid-datacloud.eu/modules/train-an-image-classifier.html>`_. This will allow us to see the general workflow.

Once we have chosen the model at the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`_ we will
find that it has an associated docker container in `DockerHub <https://hub.docker.com/u/deephdc/>`_. For example, in the
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

So if you wanted to use gpu and the test branch you could run:

.. code-block::

    $ docker pull deephdc/deep-oc-image-classification-tf:gpu-test

Instead of pulling from Dockerhub you can choose to build the image yourself:

.. code-block::

    $ git clone https://github.com/deephdc/deep-oc-image-classification-tf
    $ cd deep-oc-image-classification-tf
    $ docker build -t deephdc/deep-oc-image-classification-tf .


2. Store your data
------------------

To run locally you have two options:

* Have your data stored locally
* Have your data at a remote storage resource

When training a model, the data has usually to be in a specific format and folder structure.
It's usually helpful to read the README in the source code of the module
(in this case located `here <https://github.com/deephdc/image-classification-tf>`_)
to learn the correct way to setting it up.

Have your data stored locally
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You should make sure that you export inside of the container all the folders you need for the training

.. code-block:: console

	$ docker run -ti -p 5000:5000 -p 6006:6006  -p 8888:8888 -v path_to_local_folder:path_to_docker_folder deephdc/deep-oc-image-classification-tf

Have your data at a remote storage resource
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

For the time being we support using the DEEP Nextcloud for remote storage, although we plan to support addition platforms
such as Google Drive, Dropbox and OneData. All of these platforms are supported through `rclone <https://rclone.org/>`_.

If you have the files you need for the training stored in Nextcloud you need first to login into
`DEEP-Nextcloud  <https://data-deep.a.incd.pt/>`__ with your DEEP-IAM credentials.
Then you have to go to: **(1) Settings (top right corner)** |rarr|  **(2) Security**  |rarr|  **(3) Devices & sessions**

.. image:: ../../_static/nc-access.png

Set a name for your application (for this example it will be ``rshare``) and clik on **Create new app password**.
This will generate ``<your_nextcloud_username>`` and ``<your_nextcloud_password>`` that you should to include in your
``rclone.conf`` file (see  :doc:`more details <rclone>`). Now you can create the folders that you need in order to data
the inputs needed for the training.

.. tip::

    When developing a model you should add some code to perform a sync to be able to see locally your remote data.
    If you are using a ``trainable`` module from the Marketplace that you have not developed yourself you can skip
    this tip as this will have been taken care of.

    In order to be able to see your NextCloud folders from your docker, you should run rclone from your module's code,
    which will synchronize your NextCloud contents with your local contents (or the other way around).
    It is kind of an `rsync <https://linux.die.net/man/1/rsync>`_ but for remote storage.

    So your module should run this synchronization before the ``train()`` function tries to access data.
    To run it from inside a python script you can use the following code:

    .. code-block:: python

        import subprocess

        def sync_nextcloud(frompath, topath):
            """
            Mount a NextCloud folder in your local machine or viceversa.
            """
            command = (['rclone', 'copy', frompath, topath])
            result = subprocess.Popen(command, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
            output, error = result.communicate()
            if error:
                warnings.warn("Error while mounting NextCloud: {}".format(error))
            return output, error

        sync_nextcloud('rshare:/your/dataset/folder', '/your/data/path/inside/the/container') # sync local with nextcloud
        sync_nextcloud('/your/data/path/inside/the/container', 'rshare:/your/dataset/folder') # sync nextcloud with local


    As you can see you can sync the local contents back to NextCloud, which is useful if you want to save your trained
    model back to NextCloud.

When running the container you should export the ``rclone.conf`` file so that it can be reached from within the docker.
You can see an example on how to do this here

.. code-block::

	$ docker run -ti -p 5000:5000 -p 6006:6006 -v host_path_to_rclone.conf:/root/.config/rclone/rclone.conf deephdc/deep-oc-image-classification-tf

You can see this last step explained more in detail :doc:`here <rclone>`.
We are using the port ``5000`` to deploy the API and the port ``6006`` to monitor the training (for example using
`Tensorboard <https://www.tensorflow.org/guide/summaries_and_tensorboard>`_).


3. Train the model
------------------

Now comes the fun! Go to `<http://0.0.0.0:5000/ui>`_ and look for the ``train`` method. Modify the training parameters you wish to
change and execute. If some kind of monitorization tool is available for this model you will be able to follow the training
progress from `<http://0.0.0.0:6006>`_.

Once the training has finished, you can directly test it by clicking on the ``predict`` method.
Upload the image your want to classify and check on the predicted classes.

.. image:: ../../_static/deepaas.png
   :width: 500 px


4. Share your new module in the Marketplace
-------------------------------------------

.. tip::
    This section share lots of steps with :doc:`How to develop a model <../howto/develop-model>`
    so you can always cross-check there in case of doubt.

Let's say you managed to create a plant classifier by retraining the  `general model to identify images <https://marketplace.deep-hybrid-datacloud.eu/modules/train-an-image-classifier.html>`_ on your specific dataset. Now you want to share it with other users.

Look for the Docker repo of the original module (here `deephdc/DEEP-OC-image-classification-tf <https://github.com/deephdc/DEEP-OC-image-classification-tf>`_) and make a fork of it. Rename the repo to a new name (here `deephdc/DEEP-OC-plants-classification-tf <https://github.com/deephdc/DEEP-OC-plants-classification-tf>`_)

Now, in your fork, edit  the following files according to your needs:

* ``Dockerfile``: For example make sure that the model know downloads the `new weights <https://github.com/deephdc/DEEP-OC-plants-classification-tf/blob/ae5eca2bda6a2a7bbbf68f2767954dc819e50c4f/Dockerfile#L19-L21>`_ instead of the `original weights <https://github.com/deephdc/DEEP-OC-image-classification-tf/blob/080584f0ad9c660ec0e55240d4b4979f2d45a2bb/Dockerfile#L112-L114>`_.
  Check your Dockerfile works correctly by building it locally and running it:
  ::

    docker build --no-cache -t your_project .
    docker run -ti -p 5000:5000 -p 6006:6006 -p 8888:8888 your_project
    # --> your module should be visible in http://0.0.0.0:5000/ui

* ``metadata.json``: this is the information that will be displayed in the Marketplace.
  Update and add the information you need to make sure it reflects your new usecase.
  Check you didn't mess up the JSON formatting by running:
  ::

    pip install git+https://github.com/deephdc/schema4apps
    deep-app-schema-validator metadata.json

Once you are fine with the state of your module, push the changes to Github.

Once your repos are set it's time to make a PR to add your model to the marketplace!
For this you have to fork the code of the DEEP catalog repo (`deephdc/deep-oc <https://github.com/deephdc/deep-oc>`_)
and add your Docker repo name at the end of the ``MODULES.yml``.

.. code-block:: console

    git clone https://github.com/[my-github-fork]
    cd [my-github-fork]
    echo '- module: https://github.com/[my-account-name]/DEEP-OC-[my-app-name]' >> MODULES.yml
    git commit -a -m "adding new module to the catalogue"
    git push

You can also make it `online on GitHub <https://github.com/deephdc/deep-oc/edit/master/MODULES.yml>`_.

Once the changes are done, make a PR of your fork to the original repo and wait for approval.
Check the `GitHub Standard Fork & Pull Request Workflow <https://gist.github.com/Chaser324/ce0505fbed06b947d962>`_ in case of doubt.
