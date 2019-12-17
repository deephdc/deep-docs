.. highlight:: console

*********************
Try a service locally
*********************


1. Get Docker
-------------

The first step is having `Docker <https://www.docker.com>`_ installed. To have an up-to-date installation please follow
the `official Docker installation guide <https://docs.docker.com/install>`_.


2. Search for a model in the marketplace
----------------------------------------

The next step is to look for a service in the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`_ you want to try locally.


3. Get the model
----------------

You will find that each model has an associate Docker container in DockerHub. For example
`DEEP OC Image Classification <https://marketplace.deep-hybrid-datacloud.eu/modules/deep-oc-image-classification-tensorflow.html>`_
is associated with `deephdc/deep-oc-image-classification-tf <https://hub.docker.com/r/deephdc/deep-oc-image-classification-tf>`_,
`DEEP OC Massive Online Data Streams <https://marketplace.deep-hybrid-datacloud.eu/modules/deep-oc-massive-online-data-streams.html>`_
is associated with `deephdc/deep-oc-mods <https://hub.docker.com/r/deephdc/deep-oc-mods>`_, etc.

Let call the service you selected ``deep-oc-service_of_interest``.
Please, download the container with:
::

    $ docker pull deephdc/deep-oc-service_of_interest


4. Run the model
----------------

Run the container with:
::

	$ docker run -ti -p 5000:5000 deephdc/deep-oc-service_of_interest


5. Go to the API, get the results
---------------------------------

Once running, point your browser to `http://127.0.0.1:5000/ <http://127.0.0.1:5000/>`_
and you will see the API documentation, where you can test the service functionality, as well as perform other actions.

.. image:: ../../_static/deepaas.png

The concrete results can vary from service to service. For example the plant classifier return a list of plant species
along with the predicted probabilities.
