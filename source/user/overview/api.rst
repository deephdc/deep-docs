DEEPaaS API
===========

The `DEEPaaS API <https://github.com/indigo-dc/DEEPaaS>`_ enables a user friendly interaction with the underlying Deep
Learning modules and can be used both for training models and doing inference with services.

For a detailed up-to-date documentation please refer to the `official DEEPaaS documentation <https://docs.deep-hybrid-datacloud.eu/projects/deepaas/en/stable/>`_.


Integrate your model with the API
---------------------------------

To make your Deep Learning model compatible with the DEEPaaS API you have to:

1. Define the API methods for your model
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create a Python file (named for example ``deep_api.py``) inside your package. In this file you can define any of the
`API methods <https://docs.deep-hybrid-datacloud.eu/projects/deepaas/en/stable/api.html#v2-models>`_.
You don't need to define all the methods, just the ones you need.
Every other method will return a ``NotImplementError`` when  queried from the API.
For example:

* **Enable prediction**: implement ``get_predict_args`` and ``predict``.
* **Enable training**: implement ``get_train_args`` and ``train``.
* **Enable model weights preloading**: implement ``warm``.
* **Enable model info**: implement ``get_metadata``.

Here is `an example of the implementation <https://github.com/indigo-dc/image-classification-tf/blob/master/imgclas/api.py>`__  
of the methods. You can also browse our `github repository <https://github.com/deephdc>`__ for more examples.

2. Define the entrypoints to your model
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You must define the entrypoints pointing to this file in the ``setup.cfg`` as following:
::

    [entry_points]
    deepaas.v2.model =
        pkg_name = pkg_name.deep_api

Here is an `example <https://github.com/indigo-dc/image-classification-tf/blob/master/setup.cfg#L25-L27>`__ of the entrypoint
definition in the ``setup.cfg`` file.

.. tip::
    When developing a model with the :doc:`DEEP Data Science template <cookiecutter-template>`, the Python file
    with the API methods will automatically be created at ``pkg_name/models/deep_api.py``, as well as the entrypoints
    pointing to it. This path can of course be modified.


Running the API
---------------

To start the API run:
::

    deepaas-run --listen-ip 0.0.0.0

and go to http://0.0.0.0:5000/ui. You will see a nice UI with all the methods:

.. image:: ../../_static/deepaas.png
   :width: 500 px
