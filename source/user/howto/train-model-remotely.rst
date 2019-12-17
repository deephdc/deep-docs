.. include:: <isonum.txt>

.. highlight:: console

**********************
Train a model remotely
**********************

This is a step by step guide on how to train a general model from the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`__ with your own dataset.


1. Choose a model
-----------------

The first step is to choose a model from the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`__.  For educational purposes We are going to use a `general model to identify images <https://marketplace.deep-hybrid-datacloud.eu/modules/deep-oc-image-classification-tensorflow.html>`_. Some of the model dependent details can change if using another model, but this tutorial will provide a general overview of the workflow to follow when using any of the models in the Marketplace. A demo showing the different steps in this HowTo has also be recorded and you can find it here :ref:`here <video-demo_train-remotely>`.



2. Prerequisites
----------------

Before being able to run your model at the Pilot Infraestructure you should first fulfill the following prerequisites:


    * `DEEP-IAM <https://iam.deep-hybrid-datacloud.eu/>`_ registration
    * Install rclone and configure it for DEEP-IAM. Instructions for this can be found  :doc:`here <rclone>`.
    * Install oidc-agent and configure it for DEEP-IAM. Instructions for this can be found  :doc:`here <oidc-agent>`. Make sure you follow the instructions in the *Usage with orchent* section.
    * Install `orchent <https://github.com/indigo-dc/orchent/releases>`_ tool

For this example we are going to use `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu>`__ for storing you data. This means you also have to:

    * Register at `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu>`__
    * Follow the **Nextcloud configuration for rclone** :doc:`here <rclone>`. This will give you <your_nextcloud_username> and <your_nextcloud_password>.


3. Upload your files to Nextcloud
---------------------------------

Upload the files you need for the training to DEEP-Nextcloud. For this, after login into `DEEP-Nextcloud  <https://nc.deep-hybrid-datacloud.eu/login>`__ with your DEEP-IAM credentials, go to: **(1) Settings (top right corner)** |rarr|  **(2) Security**  |rarr|  **(3) Devices & sessions**


.. image:: ../../_static/nc-access.png

Set a name for your application (for this example it will be **deepnc**) and clik on **Create new app password**. This will generate <your_nextcloud_username> and <your_nextcloud_password> that you will need in the next step.

Now you can create the folders that you need in order to store the inputs needed for the training and to retrieve the output. These folders will be visible from within the container. In this example we just need two folders:

.. image:: ../../_static/nc-folders.png

* A folder called **models** where the training weights will be stored after the training
* A folder called **data** that contains two different folders:
	- The folder **images** containing the input images needed for the training
        - The folder **dataset_files** containing a couple of scripts: *train.txt* indicating the relative path to the training images and *classes.txt* indicating which are the categories for the training

The folder structure and their content will of course depend on the model to be used. This structure is just an example in order to complete the workflow for this tutorial.

4. Orchent submission script
------------------------------------

With <your_nextcloud_username> and <your_nextcloud_password> from the previous step, you can generate a bash script (i.e deploy.sh) to create the orchent deployment:

.. code-block:: bash

    #!/bin/bash

    orchent depcreate ./TOSCA.yml '{ "rclone_url": "https://nc.deep-hybrid-datacloud.eu/remote.php/webdav/",
                                                "rclone_vendor": "nextcloud",
                                                "rclone_user": <your_nextcloud_username>
                                                "rclone_pass": <your_nextcloud_password> }'

This script will be the *only place* where you will have to indicate your username and password. This file should be stored locally.

.. important::
              **DO NOT** save the rclone credentials in the **CONTAINER** nor in the **TOSCA** file


5. The rclone configuration file
--------------------------------

For running the model remotely you need to include in your git repository either an empty or 'minimal' rclone.conf file, ensure that rclone_confg in the TOSCA file (section ) properly points to this file.The minimal rclone.conf content would correspond to something like::

	[deepnc]
	type = webdav
	config_automatic = yes

Remember that the first line should include the name of the remote Nextcloud application (**deepnc** for this example).

6. Prepare your TOSCA file
--------------------------

In the orchent submission script there is a call to a TOSCA file (TOSCA.yml). A generic template can be found `here <https://github.com/indigo-dc/tosca-templates/blob/master/deep-oc/deep-oc-mesos-gpu-webdav.yml>`__. The sections that should be modified are the following (TOSCA experts may modify the rest of the template to their will.)

* Docker image to deploy. In this case we will be using deephdc/deep-oc-image-classification-tf::

   dockerhub_img:
      type: string
      description: docker image from Docker Hub to deploy
      required: yes
      default: deephdc/deep-oc-image-classification-tf

* Location of the .rclone.conf (this file can be empty, but should be at the indicated location)::

   rclone_conf:
      type: string
      description: nextcloud link to access via webdav
      required: yes
      default: "/srv/image-classification-tf/rclone.conf"

For further TOSCA templates examples you can go `here <https://github.com/indigo-dc/tosca-templates/tree/master/deep-oc>`__.




7. Create the orchent deployment
-----------------------------------------

The submission is then done running the orchent submission script you generated in one of the previous steps::

	./deploy.sh

This will give you a bunch of information including your deployment ID. To check status of your job

    $ orchent depshow <Deployment ID>

Once your deployment is in status **CREATED**, you will be given a web URL::

	http://your_orchent_deployment:5000/

8. Go to the API, train the model
-----------------------------------------

Now comes the fun!

Go to *http://your_orchent_deployment:5000/* and look for the ``train`` method. Modify the training parameters you wish to
change and execute. If some kind of monitorization tool is available for this model you will be able to follow the training
progress from *http://your_orchent_deployment:6006/*

	.. image:: ../../_static/deepaas.png


9. Testing the training
---------------------------

Once the training has finished, you can directly test it by clicking on the ``predict`` method. There you can either upload the image your want to classify or give a URL to it.

