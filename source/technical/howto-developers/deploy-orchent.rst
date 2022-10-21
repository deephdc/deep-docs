.. include:: <isonum.txt>

.. highlight:: console

Deployment with CLI (orchent)
=============================

This is a step by step guide on how to make a deployment using the command line interface (instead of the Training
Dashboard).

.. admonition:: Requirements

   * `oidc-agent <https://github.com/indigo-dc/oidc-agent/releases>`_ installed and configured for `DEEP-IAM <https://iam.deep-hybrid-datacloud.eu/>`_ (see :doc:`Configure oidc-agent <oidc-agent>`).
   * `orchent <https://github.com/indigo-dc/orchent/releases>`_ tool


Prepare your TOSCA file (optional)
----------------------------------

The orchent tool needs TOSCA YAML file to configure and establish the deployment. One can generate an application specific TOSCA template or use a general one, `deep-oc-marathon-webdav.yml <https://github.com/indigo-dc/tosca-templates/blob/master/deep-oc/deep-oc-marathon-webdav.yml>`__, while providing necessary inputs in the bash script (see next subsecion).

If you create your own TOSCA YAML file, the following sections should be modified (TOSCA experts may modify the rest of the template to their will):

* Docker image to deploy. In this case we will be using deephdc/deep-oc-image-classification-tf::

   docker_img:
      type: string
      description: docker image from Docker Hub to deploy
      required: yes
      default: deephdc/deep-oc-image-classification-tf

* Location of the ``rclone.conf`` (this file can be empty, but should be at the indicated location)::

   rclone_conf:
      type: string
      description: rclone.conf location
      required: yes
      default: "/srv/image-classification-tf/rclone.conf"

For further TOSCA templates examples you can go `here <https://github.com/indigo-dc/tosca-templates/tree/master/deep-oc>`__.

.. important::
    **DO NOT** save the rclone credentials in the **CONTAINER** nor in the **TOSCA** file


Orchent submission script
-------------------------

You can use the general template, `deep-oc-mesos-webdav.yml <https://github.com/indigo-dc/tosca-templates/blob/master/deep-oc/deep-oc-mesos-webdav.yml>`__,  but provide necessary parameters in a bash script. Here is an example for such a script, e.g. *submit_orchent.sh* :

.. code-block:: bash

    #!/bin/bash

    orchent depcreate ./deep-oc-marathon-webdav.yml '{ "docker_image": "deephdc/deep-oc-image-classification-tf"
                                                    "rclone_url": "https://data-deep.a.incd.pt/remote.php/webdav/",
                                                    "rclone_vendor": "nextcloud",
                                                    "rclone_conf": "/srv/image-classification-tf/rclone.conf"
                                                    "rclone_user": <your_nextcloud_username>
                                                    "rclone_pass": <your_nextcloud_password> }'

This script will be the **only place** where you will have to indicate <your_nextcloud_username> and <your_nextcloud_password>. This file should be stored locally and secured.

.. important::
    **DO NOT** save the rclone credentials in the **CONTAINER** nor in the **TOSCA** file

.. tip::
    When developing an application with the :doc:`DEEP Modules Template <../../user/overview/cookiecutter-template>`,
    the DEEP-OC-<your_project> repository will contain an exampled script, named *submit_orchent_tmpl.sh*

Submit your deployment
----------------------

The submission is then done by running the orchent submission script you generated in the previous step::

	./submit_orchent.sh

This will give you a bunch of information including your deployment ID. To check status of your job::

    $ orchent depshow <Deployment ID>

Once your deployment is in status **CREATED**, you will be given various endpoints::

	http://deepaas_endpoint
	http://monitor_endpoint

N.B.: to check all your deployments::

    $ orchent depls -c me
