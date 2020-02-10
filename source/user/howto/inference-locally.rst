.. highlight:: console

*********************
Try a service locally
*********************


.. admonition:: Requirements

    * having `Docker <https://www.docker.com>`_ installed. For an up-to-date installation please follow
      the `official Docker installation guide <https://docs.docker.com/install>`_.


1. Choose your module
---------------------

The first step is to choose a module from the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`_.
For educational purposes we are going to use a `general model to identify images <https://marketplace.deep-hybrid-datacloud.eu/modules/train-an-image-classifier.html>`_. This will allow us to see the general workflow.

Once we have chosen the model at the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`_ we will
find that it has an associated docker container in `DockerHub <https://hub.docker.com/u/deephdc/>`_. For example, in the
example we are running here, the container would be ``deephdc/deep-oc-image-classification-tf``. This means that to pull the
docker image and run it you should:

.. code-block:: console

    $ docker pull deephdc/deep-oc-image-classification-tf

Docker images have usually tags depending on whether they are using ``master`` or ``test`` and whether they use
``cpu`` or ``gpu``. Tags are usually:

* ``latest`` or ``cpu``: master + cpu
* ``gpu``: master + gpu
* ``cpu-test``: test + cpu
* ``gpu-test``: test + gpu

So if you wanted to use gpu and the test branch you could run:

.. code-block:: console

    $ docker pull deephdc/deep-oc-image-classification-tf:gpu-test

Instead of pulling from Dockerhub you can choose to build the image yourself:

.. code-block:: console

    $ git clone https://github.com/deephdc/deep-oc-image-classification-tf
    $ cd deep-oc-image-classification-tf
    $ docker build -t deephdc/deep-oc-image-classification-tf .


.. Tip:: It's usually helpful to read the README in the source code of the module, in this case located at
   https://github.com/deephdc/image-classification-tf.


2. Launch the API and predict
-----------------------------

Run the container with::

	$ docker run -ti -p 5000:5000 deephdc/deep-oc-image-classification-tf

Once running, point your browser to http://127.0.0.1:5000/ui and you will see the API documentation, where you can
test the module's functionality, as well as perform other actions.

.. image:: ../../_static/deepaas.png
  :width: 500

Go to the  ``predict()`` function and upload the file/data you want to predict (in the case of the image classifier
this should be an image file). The appropriate data formats of the files you have to upload are often discussed
in the module's Marketplace page.

The response from the ``predict()`` function will vary from module to module but usually consists on a JSON dict
with the predictions. For example the image classifier return a list of predicted classes along with predicted accuracy.
Other modules might return files instead of a JSON.
