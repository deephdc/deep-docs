Quickstart guide
----------------

.. todo:: Provide information on (at least):

   1. How to download and test a model (i.e. get docker, go to the marketplace, get the model, run it).

   2. Use cookiecutter to develop a model

   3. Integrate an existing model with DEEPaaS

   4. Create a container from a model


Integrate a model with the API
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Check the full :doc:`API guide <overview/api>` for more info.

To integrate a model with the `DEEPaaS API <https://github.com/indigo-dc/DEEPaaS>`_ you need to define an python script (named for example ``api.py``).
You must define the entrypoints to this file in the ``setup.cfg`` as following:
::
    [entry_points]
    deepaas.model =
        pkg_name = pkg_name.api

The ``api.py`` file can (optionally) include any of the  following :ref:`methods <user/overview/api:Methods>` to apply to your model:

* :ref:`get_metadata <api-methods_get-metadata>`
* :ref:`get_train_args <api-methods_get-train-args>`
* :ref:`train <api-methods_train>`
* :ref:`predict_file <api-methods_predict-file>`
* :ref:`predict_data <api-methods_predict-data>`
* :ref:`predict_url <api-methods_predict-url>`

To test the API locally, install the API with ``pip install deepaas`` and run it with ``deepaas-run --listen-ip 0.0.0.0``.

