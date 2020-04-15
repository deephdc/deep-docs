.. highlight:: console

**********************
Try a service remotely
**********************


The first step is to go to the `DEEP as a Service (or DEEPaaS) <https://deepaas.deep-hybrid-datacloud.eu/>`_.
For educational purposes we are going to use a `general model to identify images <https://marketplace.deep-hybrid-datacloud.eu/modules/train-an-image-classifier.html>`_. This will allow us to see the general workflow.

Go to ``Swagger Interfaces`` and click ``Swagger UI for "image-classification-tf"``.
You will see the API documentation, where you can test the module's functionality, as well as perform other actions.

.. image:: ../../_static/deepaas.png
  :width: 500

Go to the  ``predict()`` function and upload the file/data you want to predict (in the case of the image classifier
this should be an image file). The appropriate data formats of the files you have to upload are often discussed
in the module's Marketplace page.

The response from the ``predict()`` function will vary from module to module but usually consists on a JSON dict
with the predictions. For example the image classifier return a list of predicted classes along with predicted accuracy.
Other modules might return files instead of a JSON.
