.. highlight:: console

********************
Try a model remotely
********************



1. Search for a model in the marketplace
----------------------------------------

The next step is to look for a model 
in the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`_
you want to try remotely.  
The marketplace contains an extensible list of existing models e.g. 
`DEEP OC Image Classification <https://marketplace.deep-hybrid-datacloud.eu/models/deep-oc-image-classification-tensorflow.html>`_,
`DEEP OC Retinopathy <https://marketplace.deep-hybrid-datacloud.eu/models/deep-oc-retinopathy.html>`_,
`DEEP OC Massive Online Data Streams <https://marketplace.deep-hybrid-datacloud.eu/models/deep-oc-massive-online-data-streams.html>`_,
`DEEP OC Seed Classification <https://marketplace.deep-hybrid-datacloud.eu/models/deep-oc-seed-classification-theano.html>`_,
`DEEP OC Phytopankton (Theano) <https://marketplace.deep-hybrid-datacloud.eu/models/deep-oc-phytopankton-theano.html>`_,
`DEEP OC Conus Classification <https://marketplace.deep-hybrid-datacloud.eu/models/deep-oc-conus-classification-theano.html>`_,
`DEEP OC dogs breed determination <https://marketplace.deep-hybrid-datacloud.eu/models/deep-oc-dogs-breed-determination.html>`_,
and many more.


2. Prepare your TOSCA file
----------------



Let call the model you selected ``deep-oc-model_of_interest``. 


   

3. Run the model
----------------

Run the container with:
::

	$ docker run -ti -p 5000:5000 deephdc/deep-oc-model_of_interest
	

5. Go to the API, get the results
---------------------------------

Once running, point your browser to `http://127.0.0.1:5000/ <http://127.0.0.1:5000/>`_ 
and you will see the API documentation, 
where you can test the model functionality, as well as perform other actions.

All models in the `DEEP Open Catalog marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`_
utilize `DEEPaaS API <https://github.com/indigo-dc/DEEPaaS>`_.
The API enables a user friendly interaction with the underlying Deep Learning models and 
can be used both for training and inference with the models.

.. image:: ../../_static/deepaas.png

The concrete results can vary from model to model e.g. 
the results of ``deephdc/deep-oc-image-classification-tf`` are image type(s) and picture(s),
the results of ``deephdc/deep-oc-mods`` are predicted values (float numbers).
