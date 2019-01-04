Quickstart guide
----------------

.. todo:: Provide information on (at least):

   1. How to download and test a model (i.e. get docker, go to the marketplace, get the model, run it).

   2. Use cookiecutter to develop a model

   3. Integrate an existing model with DEEPaaS

   4. Create a container from a model


Integrate a model with the API
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The `DEEPaaS API <https://github.com/indigo-dc/DEEPaaS>`_ enables a user friendly interaction with the underlying Deep
Learning models and can be used both for training and inference with the models.

.. image:: ../_static/deepaas.png


To :ref:`integrate your model with the API <user/overview/api:Integrate your model with the API>` with the API you need
to define the :ref:`API methods <user/overview/api:Methods>` on your model.
Those methods can (optionally) include any of the following:

* :ref:`get_metadata <api-methods_get-metadata>`
* :ref:`get_train_args <api-methods_get-train-args>`
* :ref:`train <api-methods_train>`
* :ref:`predict_file <api-methods_predict-file>`
* :ref:`predict_data <api-methods_predict-data>`
* :ref:`predict_url <api-methods_predict-url>`

To test the API locally, install the API with ``pip install deepaas`` and run it with ``deepaas-run --listen-ip 0.0.0.0``.

Check the full :doc:`API guide <overview/api>` for more info.