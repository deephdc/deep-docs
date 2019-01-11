.. highlight:: console

**********************
Try a service remotely
**********************


1. Search for a model in the marketplace
----------------------------------------

The next step is to look for a service
in the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`_
you want to try locally. Possible services include
`Retinopathy diagnosing <https://marketplace.deep-hybrid-datacloud.eu/models/deep-oc-retinopathy.html>`_,
plant classification,
seed classification,
etc.

.. todo:: Add link to marketplace services tag + ling to plant classifier seed clasifier in tensorflow.


2. Prepare your TOSCA file
--------------------------

Let's call the service you selected ``deep-oc-service_of_interest``.


3. Run the model
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
