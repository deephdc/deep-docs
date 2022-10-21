.. include:: <isonum.txt>
.. highlight:: console

Train a model remotely
======================

.. admonition:: Useful video demos
   :class: important

    - `Training your model in the cloud with the DEEP training dashboard <https://www.youtube.com/watch?v=uqFXrEwtNNs&list=PLJ9x9Zk1O-J_UZfNO2uWp2pFMmbwLvzXa&index=6>`__

This is a step by step guide on how to train a general model from the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`__ with your own dataset.

.. admonition:: Requirements

    Before being able to run your model at the Pilot Infrastructure you should first fulfill the following prerequisites:

    * `DEEP-IAM <https://iam.deep-hybrid-datacloud.eu/>`__ registration

    For this example we are going to use `DEEP-Nextcloud <https://data-deep.a.incd.pt/>`__ for storing data. This means you also have to:

        * Register at `DEEP-Nextcloud <https://data-deep.a.incd.pt/>`__ (i.e. login with your DEEP-IAM credentials)
        * :ref:`Obtain Nextcloud security parameters for rclone <user/howto/rclone:Nextcloud configuration for rclone>`. This will provide you with ``<your_nextcloud_username>`` and
          ``<your_nextcloud_password>``.


1. Choose a model
-----------------

The first step is to choose a model from the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`__.  For educational purposes we are going to use a `general model to identify images <https://marketplace.deep-hybrid-datacloud.eu/modules/train-an-image-classifier.html>`_. Some of the model dependent details can change if using another model, but this tutorial will provide a general overview of the workflow to follow when using any of the models in the Marketplace. A demo showing the different steps in this HowTo has also be recorded and you can find it our `YouTube channel <https://www.youtube.com/playlist?list=PLJ9x9Zk1O-J_UZfNO2uWp2pFMmbwLvzXa>`_.


2. Upload your files to Nextcloud
---------------------------------

When training a model, the data has usually to be in a specific format and folder structure.
It's usually helpful to read the README in the source code of the module
(in this case located `here <https://github.com/deephdc/image-classification-tf>`__)
to learn the correct way to setting it up.

We login to DEEP-Nextcloud with DEEP-IAM credentials and upload there the files needed for training. We create the folders that you need in order to store the inputs needed for the training and to retrieve the output. These folders will be visible from within the container. In this example we just need two folders (``models`` and ``data``):

.. image:: ../../_static/nc-folders.png


* A folder called **models** where the training weights will be stored after the training
* A folder called **data** that contains two different folders:

   * The folder **images** containing the input images needed for the training
   * The folder **dataset_files** containing a couple of scripts: *train.txt* indicating the relative path to the training images and *classes.txt* indicating which are the categories for the training

The folder structure and their content will of course depend on the model to be used. This structure is just an example in order to complete the workflow for this tutorial.


3. Deploy with the Training Dashboard
-------------------------------------

* Go to the `Training Dashboard <https://train.deep-hybrid-datacloud.eu/>`__
* Login with DEEP-IAM credentials
* Find the **Train an image classifier** module, click **Train module**
* Fill the webform, e.g. provide <your_nextcloud_username> for ``rclone_user`` and <your_nextcloud_password> for ``rclone_password`` (see :ref:`Nextcloud configuration for rclone <user/howto/rclone:Nextcloud configuration for rclone>`)
* Click **Submit**
* Look for **Access** and choose **DEEPaaS**, you will be redirected to ``http://deepaas_endpoint``

See the :doc:`Dashboard guide <../overview/dashboard>` for more details.


4. Go to the API, train the model
---------------------------------

Now comes the fun!

Go to ``http://deepaas_endpoint/ui`` and look for the ``train`` POST method. Modify the training parameters you wish to
change and execute. If some kind of monitorization tool is available for the module, you will be able to follow the training
progress at ``http://monitor_endpoint``.

.. image:: ../../_static/deepaas.png
   :width: 500 px

In the Dashboard you will be able to view the training history of that deployment:

.. image:: ../../_static/dashboard-history.png


5. Testing the training
-----------------------

Once the training has finished, you can directly test it by clicking on the ``predict`` POST method. There you can either upload the image your want to classify or give an URL to it.


6. Share your new module in the Marketplace
-------------------------------------------

.. tip::
    This section share lots of steps with :doc:`How to develop a model <../howto/develop-model>`
    so you can always cross-check there in case of doubt.

Let's say you managed to create a plant classifier by retraining the  `general model to identify images <https://marketplace.deep-hybrid-datacloud.eu/modules/train-an-image-classifier.html>`__ on your specific dataset. Now you want to share it with other users.

Look for the Docker repo of the original module (here `deephdc/DEEP-OC-image-classification-tf <https://github.com/deephdc/DEEP-OC-image-classification-tf>`__) and make a fork of it. Rename the repo to a new name (here `deephdc/DEEP-OC-plants-classification-tf <https://github.com/deephdc/DEEP-OC-plants-classification-tf>`_)

Now, in your fork, edit  the following files according to your needs:

* ``Dockerfile``: For example make sure that the model know downloads the `new weights <https://github.com/deephdc/DEEP-OC-plants-classification-tf/blob/ae5eca2bda6a2a7bbbf68f2767954dc819e50c4f/Dockerfile#L19-L21>`__ instead of the `original weights <https://github.com/deephdc/DEEP-OC-image-classification-tf/blob/080584f0ad9c660ec0e55240d4b4979f2d45a2bb/Dockerfile#L112-L114>`_.
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
For this you have to fork the code of the DEEP catalog repo (`deephdc/deep-oc <https://github.com/deephdc/deep-oc>`__)
and add your Docker repo name at the end of the ``MODULES.yml``.

.. code-block:: console

    git clone https://github.com/[my-github-fork]
    cd [my-github-fork]
    echo '- module: https://github.com/deephdc/UC-[my-account-name]-DEEP-OC-[my-app-name]' >> MODULES.yml
    git commit -a -m "adding new module to the catalogue"
    git push

You can also make it `online on GitHub <https://github.com/deephdc/deep-oc/edit/master/MODULES.yml>`__.

Once the changes are done, make a PR of your fork to the original repo and wait for approval.
Check the `GitHub Standard Fork & Pull Request Workflow <https://gist.github.com/Chaser324/ce0505fbed06b947d962>`__ in case of doubt.
