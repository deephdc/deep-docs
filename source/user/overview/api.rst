DEEPaaS API
===========

The `DEEPaaS API <https://github.com/indigo-dc/DEEPaaS>`_ enables a user friendly interaction with the underlying Deep
Learning modules and can be used both for training models and doing inference with services.

For a detailed up-to-date documentation please refer to the `official DEEPaaS documentation <https://docs.deep-hybrid-datacloud.eu/projects/deepaas/en/stable/>`_.


Integrate your model with the API
---------------------------------

To make your Deep Learning model compatible with the DEEPaaS API you have to:

1. **Define the API methods for your model**

Create a Python file (named for example ``api.py``) inside your package. In this file you can define any of the
:ref:`API methods <user/overview/api:Methods>`. You don't need to define all the methods, just the ones you need.
Every other method will return a ``NotImplementError`` when  queried from the API.

Here is `example <https://github.com/indigo-dc/image-classification-tf/blob/master/imgclas/api.py>`__ of the
implementation of the methods

2. **Define the entrypoints to your model**

You must define the entrypoints pointing to this file in the ``setup.cfg`` as following:
::

    [entry_points]
    deepaas.v2.model =
        pkg_name = pkg_name.api

Here is an `example <https://github.com/indigo-dc/image-classification-tf/blob/master/setup.cfg#L25-L27>`__ of the entrypoint
definition in the ``setup.cfg`` file.

.. tip::
    When developing a model with the :doc:`DEEP Cookiecutter template <cookiecutter-template>`, the Python file
    with the API methods will automatically be created at ``pkg_name/models/model.py``, as well as the entrypoints
    pointing to it. This path can of course be modified.


Methods
-------

Please check the `official DEEPaaS documentation <https://docs.deep-hybrid-datacloud.eu/projects/deepaas/en/stable/>`_
for an updated documentation on the available methods.
