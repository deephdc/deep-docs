.. include:: <isonum.txt>
.. highlight:: console

Train a model locally
=====================

.. admonition:: Useful video demos
   :class: important

    - `Running a module locally with docker <https://www.youtube.com/watch?v=3ORuymzO7V8&list=PLJ9x9Zk1O-J_UZfNO2uWp2pFMmbwLvzXa&index=13>`__

This is a step by step guide on how to train a module from the Marketplace with your own dataset, on your local machine.

A practical application of this tutorial would be to train a `generic image classifier <https://github.com/deephdc/DEEP-OC-image-classification-tf>`__
on a custom dataset to create a `plant classifier <https://github.com/deephdc/DEEP-OC-plants-classification-tf>`__, for example.

.. admonition:: Requirements

    * having `Docker <https://www.docker.com>`__ installed. For an up-to-date installation please follow
      the `official Docker installation guide <https://docs.docker.com/install>`__.



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

In our case, this looks like the following we have to mount one folder for the data
and one folder for the model weights (where we will later retrieve the newly trained model):

.. code-block:: console

	$ docker run -ti -p 5000:5000 -p 6006:6006  -p 8888:8888 -v /home/ubuntu/data:/srv/image-classification-tf/data -v /home/ubuntu/models:/srv/image-classification-tf/models deephdc/deep-oc-image-classification-tf


4. Open the DEEPaaS API and train the model
-------------------------------------------

Go to `<http://0.0.0.0:5000/ui>`__ and look for the ``train`` POST method.
Modify the training parameters you wish to change and execute.

If some kind of monitorization tool is available for this model you will be able to follow the training
progress from `<http://0.0.0.0:6006>`__.


5. Test and export the newly trained model
------------------------------------------

Once the training has finished, you can directly test it by clicking on the ``predict`` POST method.
Select you new model weights upload the image your want to classify.

If you are satisfied with your model, you have to save your new model weights to a remote storage.
For the next step, you need to make them publicly available through an URL so they can be downloaded in your Docker container
(see `Nextcloud Public Links <https://docs.nextcloud.com/server/latest/user_manual/en/files/sharing.html>`__).


6. Create a Docker repo for your new module
-------------------------------------------

Now, let's say you want to share your new application with your colleagues.
The process is much simpler that when :doc:`developing a new module from scratch <develop-model>`,
as your code is the same as the original application, only your model weights
are different.

To account for this simpler process, we have prepared a version of the
:doc:`the DEEP Modules Template <../overview/cookiecutter-template>`
specially tailored to this task.

In your local machine, run the Template with the ``child`` branch.

.. code-block::

    $ pip install cookiecutter
    $ cookiecutter https://github.com/deephdc/cookiecutter-deep -b child-module

This will create a single repo (``DEEP-OC-**``) with the Docker code.

Now:

**(1)** Modify ``metadata.json`` with the proper description of your new module.
This is the information that will be displayed in the Marketplace.
Check you didn't mess up the JSON formatting by running:

.. code-block:: console

    $ pip install git+https://github.com/deephdc/schema4apps
    $ deep-app-schema-validator metadata.json

**(2)** Then go to the ``Dockerfile``. You will see that the base Docker image
is the image of the original repo. Modify the appropriate lines to replace
the original model weights with the new model weights.
In our case, this could look something like this:
::
    ENV SWIFT_CONTAINER https://api.cloud.ifca.es:8080/swift/v1/phytoplankton-tf/
    ENV MODEL_TAR phytoplankton.tar.xz

    RUN rm -rf image-classification-tf/models/*
    RUN curl --insecure -o ./image-classification-tf/models/${MODEL_TAR} \
        ${SWIFT_CONTAINER}${MODEL_TAR}

Check your Dockerfile works correctly by building it locally and running it:

  .. code-block:: console

    $ docker build --no-cache -t your_project .
    $ docker run -ti -p 5000:5000 -p 6006:6006 -p 8888:8888 your_project

Your module should be visible in http://0.0.0.0:5000/ui

Once you are fine with the state of your module, push the changes to Github.


7. Share your new module in the Marketplace
-------------------------------------------

Once your repo is set, it's time to make a PR to add your model to the marketplace!

For this you have to fork the code of the DEEP catalog repo (`deephdc/deep-oc <https://github.com/deephdc/deep-oc>`__)
and add your Docker repo name at the end of the ``MODULES.yml``.
You can do this directly `online on GitHub <https://github.com/deephdc/deep-oc/edit/master/MODULES.yml>`__ or via the command line:

.. code-block:: console

    $ git clone https://github.com/[my-github-fork]
    $ cd [my-github-fork]
    $ echo '- module: https://github.com/deephdc/UC-[my-account-name]-DEEP-OC-[my-app-name]' >> MODULES.yml
    $ git commit -a -m "adding new module to the catalogue"
    $ git push

Once the changes are done, make a PR of your fork to the original repo and wait for approval.
Check the `GitHub Standard Fork & Pull Request Workflow <https://gist.github.com/Chaser324/ce0505fbed06b947d962>`__ in case of doubt.

When your module gets approved, you may need to commit and push a change to ``metadata.json``
in ``DEEP-OC-your_project`` (`ref <https://github.com/deephdc/DEEP-OC-demo_app/blob/726e068d54a05839abe8aef741b3ace8a078ae6f/Jenkinsfile#L104>`__)
so that the Pipeline is run for the first time, and your module gets rendered in the marketplace.


8. [optional] Add your new module to the original Continuous Integration pipeline
---------------------------------------------------------------------------------

Your module is already in the Marketplace.
But what happens if the code in the original image-classification module changes?
This should trigger a rebuild of your Docker container as it is based on that code.

This can be achieved by modifying the ``Jenkinsfile`` in the `image-classification Docker repo <https://github.com/deephdc/DEEP-OC-image-classification-tf/blob/master/Jenkinsfile>`__.
One would add an additional stage to the Jenkins pipeline like so:

.. code-block::

    stage("Re-build DEEP-OC Docker images for derived services") {
        when {
            anyOf {
               branch 'master'
               branch 'test'
               buildingTag()
            }
        }
        steps {

            // Wait for the base image to be correctly updated in DockerHub as it is going to be used as base for
            // building the derived images
            sleep(time:5, unit:"MINUTES")

            script {
                def derived_job_locations =
                ['Pipeline-as-code/DEEP-OC-org/DEEP-OC-plants-classification-tf',
                 'Pipeline-as-code/DEEP-OC-org/DEEP-OC-conus-classification-tf',
                 'Pipeline-as-code/DEEP-OC-org/DEEP-OC-seeds-classification-tf',
                 'Pipeline-as-code/DEEP-OC-org/DEEP-OC-phytoplankton-classification-tf'
                 ]

                for (job_loc in derived_job_locations) {
                    job_to_build = "${job_loc}/${env.BRANCH_NAME}"
                    def job_result = JenkinsBuildJob(job_to_build)
                    job_result_url = job_result.absoluteUrl
                }
            }
        }
    }

So if you want this step to be performed, you must submit a PR to the original module Docker repo with similar changes as above.

