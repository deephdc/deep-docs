DEEP Open Catalogue: Massive Online Data Streams
================================================

This is a service to analyze online data streams in order to generate alerts in real-time. 
The core part is built as ML application using ML/DL techniques for modelling in co-function 
with underlying Intrusion Detection Systems (IDS) supervising traffic networks of compute centers. 
The service is running on TensorFlow backend. 
Further information on the package structure and the requirements can be found in the
documentation in the `git repository <https://github.com/deephdc/mods>`_.

+-----------------------------------------------------------------+---------------------+
| Data Science application                                        |   ML/DL, NN, RNN    |
+-----------------------------------------------------------------+---------------------+
| Deep Learning Framework(s)                                      |  Tensorflow, Keras  |
+-----------------------------------------------------------------+---------------------+
| Programming language                                            |      Python         |
+-----------------------------------------------------------------+---------------------+
| GPU version                                                     |        yes          |
+-----------------------------------------------------------------+---------------------+
| CPU version                                                     |        yes          |
+-----------------------------------------------------------------+---------------------+
| `DEEPaaS API <https://deepaas.readthedocs.io/en/stable/>`_      |        yes          |
+-----------------------------------------------------------------+---------------------+ 
| :doc:`DEEP DS template <../overview/cookiecutter-template>`     |        yes          |
+-----------------------------------------------------------------+---------------------+
| `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu/>`_ access |        yes          |
+-----------------------------------------------------------------+---------------------+

Keywords: ML/DL, NN, RNN, time-series data, cyber-security, Tensorflow

DEEP-OC DockerHub image: https://hub.docker.com/r/deephdc/deep-oc-mods

DEEP-OC Dockerfile: https://github.com/deephdc/DEEP-OC-mods

Application source code: https://github.com/deephdc/mods


Description
-----------

The principle of the security/anomaly detection is proactive time-series prediction adopting artificial NNs 
to build prediction models capable to predict next step(s) in near future based on given current and past steps. 
The discrepancy between the prediction and the reality gives an indication of warning levels
when supervising networks activities.

The challenge of the solution is also it aims to scalable edge technologies to support extensive data analysis and modelling 
as well as to improve the cyber-resilience by adopting an heuristic approach combining misuse detection 
in real-time with the building intelligence module using ML, NN and DL techniques.


Workflow
--------
The described workflow supposes usage of downloaded from `DEEP Open Catalog <https://marketplace.deep-hybrid-datacloud.eu/>`_ Docker images, 
i.e. you need either `docker <https://docs.docker.com/install/#supported-platforms>`_ or `udocker <https://github.com/indigo-dc/udocker/releases>`_ tool.


1. Data preprocessing
^^^^^^^^^^^^^^^^^^^^^

ML/DL learns from data, then the first important thing is to have the data correctly set up.


**1.1 Feature building and ML/DL data pool**

.. note:: The description of this step from massive online data through raw private datasets and consequently to ML/DL data using in-memory operations of Apache Spark will be available later due to raw data sensitiveness!


**1.2 Prepare ML/DL data**

Put your ML/DL dataset (i.e. already prepared time-series data file) in the ``./data/`` folder. 
The location of your data can be set also in the ``./mods/config.py`` file.
Currently, the accepted ML/DL format is either ``.tsv`` or ``.csv``. 
You don't have to use all features in your dataset, the selected features are specified in ``./mods/config.py``.

You can find an example of ML/DL dataformat `here <https://github.com/deephdc/mods/blob/master/data/features-20180414-20181015-win-1_hour-slide-10_minutes.tsv>`_.


2. Predict using a existing model
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You can test prediction functionality with an existing model by running e.g.::

	./mods/models/predict.py predict.py --model_name MODEL_NAME --file FILE
	
Most of the parameters is defaultly, except the ``model`` and ``data``, which must be specified by user. 
Manual is available as ``./mods/models/predict.py --help``	


3. Train the model
^^^^^^^^^^^^^^^^^^

Before training the model you can customize the default parameters of the configuration file ``./mods/config.py``. 
This step is optional and training can be launched with the default configurarion parameters and still offers reasonably good results.

Once you have customized the configuration parameters, you can launch the train model using default configuration by running::

   ./mods/models/train.py --model_name MODEL_NAME --data DATA

Most of the parameters is defaultly, except the ``model_name`` and ``data``,  which must be specified by user. 
Manual is available as ``./mods/models/train.py --help``

The prediction using the created model goes through DEEPaaS API
``./mods/models/model.py --method train fullpath_to_data fullpath_to_model [args ...]``

After training, the trained model is packed together with the model scaler and the model configuration in one ``.zip`` file located in the ``./models/`` folder.  

.. note:: Work-in-progress. 

4. Test the model
^^^^^^^^^^^^^^^^^

You can test the created model using default configuration by running
``./mods/models/test.py --model_name MODEL_NAME --file FILE``. 

.. image:: ../../_static/mods_20181015_lstm_6m_1h_1h.png
Fig. 1 Train and test on 6 month monitoring dataset. 
Blue=dataset, green=prediction on train dataset, red=prediction on test (unseen) dataset.

.. image:: ../../_static/mods_20181018-lstm-3days.png
Fig. 2 Train and test on three day dataset for better visualisation (monitoring of two aspects simultaneously).
Blue=dataset, green=prediction on train dataset, red=prediction on test (unseen) dataset.

.. note:: Work-in-progress.



Launching the full DEEPaas API
------------------------------

1. Prediction and train through DEEPaaS
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

* You can easily try the default configuration by start the container as::

    $ docker run -ti -p 5000:5000 deephdc/deep-oc-mods   
       
* Direct your web browser to http://127.0.0.1:5000

* Go to ``POST /models/mods/predict`` for prediction OR ``PUT /models/mods/train`` for retrain, click ``Try it out`` button

* Go to ``Data file``, select some ``.tsv`` file containing observations like `here <https://github.com/deephdc/mods/blob/master/data/sample_data.tsv>`_. Set parameters for retrain if needed.

* Click ``Execute`` and get predicted values in JSON format OR new retrained model in the ``./models/`` folder.

The prediction using the created model goes through DEEPaaS API
``./mods/models/model.py --method predict_data [args ...]``

.. note:: The model scaler and model configuration are required for prediction using the trained model. All available MODS models are packed in ``.zip`` with all three files.


2. DEEPaaS API functionality
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To access this package's complete functionality (both for training and predicting) through the DEEPaaS API 
you have to follow the instructions here: :ref:`api-integration`