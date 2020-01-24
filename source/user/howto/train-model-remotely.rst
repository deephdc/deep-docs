.. include:: <isonum.txt>

.. highlight:: console

**********************
Train a model remotely
**********************

This is a step by step guide on how to train a general model from the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`__ with your own dataset.


1. Choose a model
-----------------

The first step is to choose a model from the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`__.  For educational purposes we are going to use a `general model to identify images <https://marketplace.deep-hybrid-datacloud.eu/modules/train-an-image-classifier.html>`_. Some of the model dependent details can change if using another model, but this tutorial will provide a general overview of the workflow to follow when using any of the models in the Marketplace. A demo showing the different steps in this HowTo has also be recorded and you can find it our `YouTube channel <https://www.youtube.com/playlist?list=PLJ9x9Zk1O-J_UZfNO2uWp2pFMmbwLvzXa>`_.



2. Prerequisites
----------------

Before being able to run your model at the Pilot Infrastructure you should first fulfill the following prerequisites:


    * `DEEP-IAM <https://iam.deep-hybrid-datacloud.eu/>`_ registration
    * To run it via web interface:
      access either `Training Dashboard <https://train.deep-hybrid-datacloud.eu/>`_ or `General purpose Dashboard <https://paas.cloud.cnaf.infn.it/>`_  with DEEP-IAM credentials
    * To run it via command-line interface (CLI):

       * `oidc-agent <https://github.com/indigo-dc/oidc-agent/releases>`_ installed and configured for `DEEP-IAM <https://iam.deep-hybrid-datacloud.eu/>`_ (see :ref:`Configure oidc-agent <user/howto/oidc-agent:Configure oidc-agent>`).
       * `orchent <https://github.com/indigo-dc/orchent/releases>`_ tool

For this example we are going to use `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu>`__ for storing data. This means you also have to:

    * Register at `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu>`__ (i.e. login with your DEEP-IAM credentials)
    * Follow the **Nextcloud configuration for rclone** :doc:`here <rclone>`. This will give you <your_nextcloud_username> and <your_nextcloud_password>.


3. Upload your files to Nextcloud
---------------------------------

We login to DEEP-Nextcloud with DEEP-IAM credentials and upload there the files needed for training. We create the folders that you need in order to store the inputs needed for the training and to retrieve the output. These folders will be visible from within the container. In this example we just need two folders (``models`` and ``data``):

.. image:: ../../_static/nc-folders.png


* A folder called **models** where the training weights will be stored after the training
* A folder called **data** that contains two different folders:

   * The folder **images** containing the input images needed for the training
   * The folder **dataset_files** containing a couple of scripts: *train.txt* indicating the relative path to the training images and *classes.txt* indicating which are the categories for the training

The folder structure and their content will of course depend on the model to be used. This structure is just an example in order to complete the workflow for this tutorial.


4. The rclone configuration file
--------------------------------

For running the model remotely you need to include in your git repository either an empty or 'minimal' rclone.conf file, ensure that rclone_confg in the TOSCA file (section ) properly points to this file. The minimal rclone.conf content would correspond to something like::

	[rshare]
	type = webdav
	config_automatic = yes

Remember that the first line should include the name of the remote Nextcloud application (**rshare** for this example).

.. tip::
    When developing an application with the :ref:`Data Science template <user/overview/cookiecutter-template:DEEP Data Science template>`, 
    the Dockerfile already includes creation of an empty rclone.conf


5. Deployment with the Training Dashboard (easy!)
-------------------------------------------------

* Go to the `Training Dashboard <https://train.deep-hybrid-datacloud.eu/>`_ 
* Login with DEEP-IAM credentials
* Find "Train an image classifier" module, click "Configure"
* Fill the webform, e.g. provide <your_nextcloud_username> for ``rclone_user`` and <your_nextcloud_password> for ``rclone_password``
* Submit
* Click "Access" and choose "DEEPaaS", you will be redirected to ``deepaas_endpoint``


6. Deployment with CLI (orchent)
--------------------------------

a) Prepare your TOSCA file (optional)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The orchent tool needs TOSCA YAML file to configure and establish the deployment. One can generate an application specific TOSCA template or use a general one, `deep-oc-mesos-webdav.yml <https://github.com/indigo-dc/tosca-templates/blob/master/deep-oc/deep-oc-mesos-webdav.yml>`__, while providing necessary inputs in the bash script (see next subsecion).

If you create your own TOSCA YAML file, the following sections should be modified (TOSCA experts may modify the rest of the template to their will):

* Docker image to deploy. In this case we will be using deephdc/deep-oc-image-classification-tf::

   docker_img:
      type: string
      description: docker image from Docker Hub to deploy
      required: yes
      default: deephdc/deep-oc-image-classification-tf

* Location of the .rclone.conf (this file can be empty, but should be at the indicated location)::

   rclone_conf:
      type: string
      description: rclone.conf location
      required: yes
      default: "/srv/image-classification-tf/rclone.conf"

For further TOSCA templates examples you can go `here <https://github.com/indigo-dc/tosca-templates/tree/master/deep-oc>`__.

.. important::
              **DO NOT** save the rclone credentials in the **CONTAINER** nor in the **TOSCA** file


b) Orchent submission script
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can use the general template, `deep-oc-mesos-webdav.yml <https://github.com/indigo-dc/tosca-templates/blob/master/deep-oc/deep-oc-mesos-webdav.yml>`__,  but provide necessary parameters in a bash script. Here is an example for such a script, e.g. *submit_orchent.sh* :

.. code-block:: bash

    #!/bin/bash

    orchent depcreate ./deep-oc-mesos-webdav.yml '{ "docker_image": "deephdc/deep-oc-image-classification-tf"
                                                    "rclone_url": "https://nc.deep-hybrid-datacloud.eu/remote.php/webdav/",
                                                    "rclone_vendor": "nextcloud",
                                                    "rclone_conf": "/srv/image-classification-tf/rclone.conf"
                                                    "rclone_user": <your_nextcloud_username>
                                                    "rclone_pass": <your_nextcloud_password> }'

This script will be the **only place** where you will have to indicate <your_nextcloud_username> and <your_nextcloud_password>. This file should be stored locally and secured.

.. important::
              **DO NOT** save the rclone credentials in the **CONTAINER** nor in the **TOSCA** file

.. tip::
    When developing an application with the :ref:`Data Science template <user/overview/cookiecutter-template:DEEP Data Science template>`, 
    the DEEP-OC-<your_project> repository will contain an exampled script, named *submit_orchent_tmpl.sh*

c) Submit your deployment
^^^^^^^^^^^^^^^^^^^^^^^^^

The submission is then done by running the orchent submission script you generated in the previous step::

	./submit_orchent.sh

This will give you a bunch of information including your deployment ID. To check status of your job::

    $ orchent depshow <Deployment ID>

Once your deployment is in status **CREATED**, you will be given various endpoints::

	http://deepaas_endpoint
	http://monitor_endpoint

N.B.: to check all your deployments::

    $ orchent depls -c me


7. Go to the API, train the model
---------------------------------

Now comes the fun!

Go to *http://deepaas_endpoint* and look for the ``train`` POST method. Modify the training parameters you wish to
change and execute. If some kind of monitorization tool is available for the module, you will be able to follow the training
progress at *http://monitor_endpoint*

	.. image:: ../../_static/deepaas.png


8. Testing the training
-----------------------

Once the training has finished, you can directly test it by clicking on the ``predict`` POST method. There you can either upload the image your want to classify or give an URL to it.

