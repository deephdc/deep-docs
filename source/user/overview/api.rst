DEEPaaS API
===========

.. _api-methods:

Methods
-------

The following methods must be defined in a python file (named for example `api.py`) inside your package. You don't need
to define all the methods, just the ones you needs. Everyother method will return a `NotImplementError` when  queried
from the API.

.. _api-methods_get-metadata:

.. py:function:: get_metadata(self)

    Return metadata from the exposed model.

    :return: dictionary containing the model's metadata. Possible keys of this dict can include: 'Name', 'Version',
    'Summary', 'Home-page', 'Author', 'Author-email', 'License', etc.


.. _api-methods_get-train-args:

.. py:function:: get_train_args(self, *args)

    Function to expose to the API what are the typical parameters needed to ru the training.

    :return: a dict of dicts with the following structure to feed the DEEPaaS API parser:

    ::

        { 'arg1' : {'default': '1',     #value must be a string (use json.dumps to convert Python objects)
                    'help': '',         #can be an empty string
                    'required': False   #bool
                    },
          'arg2' : {...
                    },
        ...
        }


.. _api-methods_train:

.. py:function:: train(self, user_conf)

    Function to train your model.

    :param dict user_conf: User configuration to train a model with custom parameters.
     It has the same keys as the dict returned by :py:func:`get_train_args`.
     The values of this dict can be loaded back as Python objects using `json.loads`.
     An example of a possible `user_conf` could be:

    ::

        {'num_classes': 'null',
         'lr_step_decay': '0.1',
         'lr_step_schedule': '[0.7, 0.9]',
         'use_early_stopping': 'false'}


.. _api-methods_predict-file:

.. py:function:: predict_file(self, path, **kwargs)

   Perform a prediction from a file in the local filesystem.

   :param str path: The person sending the message
   :return: dictionary of predictions
   :rtype: dict


.. _api-methods_predict-data:

.. py:function:: predict_data(self, data, **kwargs)

    Perform a prediction from the data passed in the arguments.
    This method will use the raw data that is passed in the `data` argument to perfom the prediction.

    :param data: raw data to be analized


.. _api-methods_predict-url:

.. py:function:: predict_url(self, *args)

    Perform a prediction from a remote URL.
    This method will perform a prediction based on the data stored in the URL passed as argument.

    :param str url: URL pointing to the data to be analized


References
----------

Here are some pointers of a the integration of an image classification model with the API:

* An `example <https://github.com/indigo-dc/image-classification-tf/blob/master/imgclas/api.py>`__ of the implementation of the methods

* An `example <https://github.com/indigo-dc/image-classification-tf/blob/master/setup.cfg>`__ of the entrypoint definition in the ``setup.cfg`` file.