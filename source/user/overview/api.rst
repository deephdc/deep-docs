DEEPaaS API
===========

The `DEEPaaS API <https://github.com/indigo-dc/DEEPaaS>`_ enables a user friendly interaction with the underlying Deep
Learning modules and can be used both for training models and doing inference with services.

For a detailed up-to-date documentation please refer to the `official DEEPaaS documentation <https://docs.deep-hybrid-datacloud.eu/projects/deepaas/en/stable/>`_.


Integrate your model with the API
---------------------------------

.. tip::
    The best approach to integrate your code with DEEPaaS is to create an empty template
    using the :doc:`DEEP Modules Template <cookiecutter-template>`. Once the template is created,
    move your code inside your package and define the API methods that will interface with
    your existing code.

To make your Deep Learning model compatible with the DEEPaaS API you have to:

1. Define the API methods for your model
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Create a Python file (named for example ``api.py``) inside your package. In this file you can define any of the
`API methods <https://docs.deep-hybrid-datacloud.eu/projects/deepaas/en/stable/user/v2-api.html>`_.
You don't need to define all the methods, just the ones you need.
Every other method will return a ``NotImplementError`` when  queried from the API.
For example:

* **Enable prediction**: implement ``get_predict_args`` and ``predict``.
* **Enable training**: implement ``get_train_args`` and ``train``.
* **Enable model weights preloading**: implement ``warm``.
* **Enable model info**: implement ``get_metadata``.

If you don't feel like reading the DEEPaaS docs (you should), here are some
examples of files you can drawn inspiration from:

* `returning a JSON response <https://github.com/deephdc/demo_app/blob/master/demo_app/api.py>`__
  for ``predict()``.
* `returning a file (eg. image, zip, etc) <https://github.com/deephdc/demo_app/blob/return-files/demo_app/api.py>`__
  for ``predict()``.
* a `more complex example <https://github.com/deephdc/image-classification-tf/blob/master/imgclas/api.py>`__ which also includes ``train``.

.. tip::
    Try to keep you module's code as decoupled as possible from DEEPaaS code, so that
    any future changes in the API are easy to integrate.
    This means that the ``predict()`` in ``api.py`` should mostly be an interface to
    your true predict function. In pseudocode:

    .. code-block:: python

        import utils  # this is where your true predict function is

        def predict(**kwargs):
            args = preprocess(kwargs)  # transform deepaas input to your standard input
            out = utils.predict(args)  # make prediction
            resp = postprocess(out)    # transform your standard output to deepaas output
            return resp

2. Define the entrypoints to your model
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You must define the entrypoints pointing to this file in the ``setup.cfg`` as following:
::

    [entry_points]
    deepaas.v2.model =
        pkg_name = pkg_name.api

Here is an `example <https://github.com/deephdc/demo_app/blob/cca3cb8e0838b0b6473549c595674e92f561f435/setup.cfg#L25-L27>`__ of the entrypoint
definition in the ``setup.cfg`` file.


Running the API
---------------

To start the API run:
::

    deepaas-run --listen-ip 0.0.0.0

and go to http://0.0.0.0:5000/ui. You will see a nice UI with all the methods:

.. image:: ../../_static/deepaas.png
   :width: 500 px
