.. include:: <isonum.txt>

.. highlight:: console

**********************
Train a model remotely
**********************

This is a step by step guide on how to train a general model from the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`__ with your own dataset.

.. admonition:: Requirements

    Before being able to run your model at the Pilot Infrastructure you should first fulfill the following prerequisites:

    * `DEEP-IAM <https://iam.deep-hybrid-datacloud.eu/>`_ registration

    For this example we are going to use `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu>`__ for storing data. This means you also have to:

        * Register at `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu>`__ (i.e. login with your DEEP-IAM credentials)
        * :doc:`Configure Nextcloud for rclone <rclone>`. This will give you ``<your_nextcloud_username>`` and
          ``<your_nextcloud_password>``.


1. Choose a model
-----------------

The first step is to choose a model from the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`__.  For educational purposes we are going to use a `general model to identify images <https://marketplace.deep-hybrid-datacloud.eu/modules/train-an-image-classifier.html>`_. Some of the model dependent details can change if using another model, but this tutorial will provide a general overview of the workflow to follow when using any of the models in the Marketplace. A demo showing the different steps in this HowTo has also be recorded and you can find it our `YouTube channel <https://www.youtube.com/playlist?list=PLJ9x9Zk1O-J_UZfNO2uWp2pFMmbwLvzXa>`_.


2. Upload your files to Nextcloud
---------------------------------

We login to DEEP-Nextcloud with DEEP-IAM credentials and upload there the files needed for training. We create the folders that you need in order to store the inputs needed for the training and to retrieve the output. These folders will be visible from within the container. In this example we just need two folders (``models`` and ``data``):

.. image:: ../../_static/nc-folders.png


* A folder called **models** where the training weights will be stored after the training
* A folder called **data** that contains two different folders:

   * The folder **images** containing the input images needed for the training
   * The folder **dataset_files** containing a couple of scripts: *train.txt* indicating the relative path to the training images and *classes.txt* indicating which are the categories for the training

The folder structure and their content will of course depend on the model to be used. This structure is just an example in order to complete the workflow for this tutorial.


3. The rclone configuration file
--------------------------------

For running the model remotely you need to include in your git repository either an empty or 'minimal' rclone.conf file, ensure that rclone_confg in the TOSCA file (section ) properly points to this file. The minimal rclone.conf content would correspond to something like::

	[rshare]
	type = webdav
	config_automatic = yes

Remember that the first line should include the name of the remote Nextcloud application (**rshare** for this example).

.. tip::
    When developing an application with the :ref:`Data Science template <user/overview/cookiecutter-template:DEEP Data Science template>`,
    the Dockerfile already includes creation of an empty rclone.conf


4. Deploy with the Training Dashboard
-------------------------------------

* Go to the `Training Dashboard <https://train.deep-hybrid-datacloud.eu/>`_
* Login with DEEP-IAM credentials
* Find the **Train an image classifier** module, click **Train module**
* Fill the webform, e.g. provide <your_nextcloud_username> for ``rclone_user`` and <your_nextcloud_password> for ``rclone_password``
* Click **Submit**
* Look for **Access** and choose **DEEPaaS**, you will be redirected to ``http://deepaas_endpoint``

If you prefer deploying using a more advanced deploying via the command-line-interface check the
:doc:`deploy with CLI <deploy-orchent>` guide.

See the :doc:`Dashboard guide <../overview/dashboard>` for more details.


5. Go to the API, train the model
---------------------------------

Now comes the fun!

Go to ``http://deepaas_endpoint/ui`` and look for the ``train`` POST method. Modify the training parameters you wish to
change and execute. If some kind of monitorization tool is available for the module, you will be able to follow the training
progress at ``http://monitor_endpoint``.

.. image:: ../../_static/deepaas.png
   :width: 500 px

In the Dashboard you will be able to view the training history of that deployment:

.. image:: ../../_static/dashboard-history.png

6. Testing the training
-----------------------

Once the training has finished, you can directly test it by clicking on the ``predict`` POST method. There you can either upload the image your want to classify or give an URL to it.

