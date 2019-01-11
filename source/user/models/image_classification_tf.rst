DEEP Open Catalogue: Image classification on TensorFlow
============================================


This is a plug-and-play tool to train and evaluate an image classifier on a custom dataset using deep neural networks running on TensorFlow. Further information on the package structure and the requirements can be found in the documentation in the `git repository <https://github.com/indigo-dc/image-classification-tf>`_ 


Workflow
-----------


**1. Data preprocessing**

The first step to train yout image classifier if to have the data correctly set up. 

1.1 Prepare the images

Put your images in the`./data/images` folder. If you have your data somewhere else you can use that location by setting the `image_dir` parameter in the  `./etc/config.yaml` file.

Please use a standard image format (like `.png` or `.jpg`). 

1.2 Prepare the data splits

First you need add to the `./data/dataset_files` directory the following files:

+-----------------------------+-------------------------------------+
|       Mandatory files       |           Optional files            |
+=============================+=====================================+
|  `classes.txt`, `train.txt` |  `val.txt`, `test.txt`, `info.txt`  |
+-----------------------------+-------------------------------------+

* `train.txt`, `val.txt` and `test.txt` files associate an image name (or relative path) to a label number (that has to *start at zero*).
* `classes.txt` file translates those label numbers to label names.
* `info.txt` allows you to provide information (like number of images in the database) about each class. This information will be shown when launching a webpage of the classifier.

You can find examples of these files `here <https://github.com/indigo-dc/image-classification-tf/tree/master/data/demo-dataset_files>`_. 

**2. Train the classifier**

Before training the classifier you can customize the default parameters of the configuration file. To have an idea of what parameters you can change, you can explore them using the `dataset exploration notebook <https://github.com/indigo-dc/image-classification-tf/blob/master/notebooks/1.0-Dataset_exploration.ipynb>`_. This step is optional and training can be launched with the default configurarion parameters and still offer reasonably good results.

Once you have customized the configuration parameters in the  `./etc/config.yaml` file you can launch the training running `./imgclas/train_runfile.py`. You can monitor the training status using Tensorboard.

After training check the `prediction statistics notebook <https://github.com/indigo-dc/image-classification-tf/blob/master/notebooks/3.1-Prediction_statistics.ipynb>`_. to see how to visualize the training statistics.

**3. Test the classifier**

You can test the classifier on a number of tasks: predict a single local image (or url), predict multiple images (or urls), merge the predictions of a multi-image single observation, etc. All these tasks are explained in the computing prediction notebooks.

.. image:: ../../_static/seeds1.png

You can also make and store the predictions of the `test.txt` file (if you provided one). Once you have done that you can visualize the statistics of the predictions like popular metrics (accuracy, recall, precision, f1-score), the confusion matrix, etc by running the  
`predictions statistics notebook <https://github.com/indigo-dc/image-classification-tf/blob/master/notebooks/3.1-Prediction_statistics.ipynb>`_. 

By running the `saliency maps notebook <https://github.com/indigo-dc/image-classification-tf/blob/master/notebooks/3.2-Saliency_maps.ipynb>`_ you can also visualize the saliency maps of the predicted images, which show what were the most relevant pixels in order to make the prediction.

.. image:: ../../_static/seeds2.png

Finally you can `launch a simple webpage <https://github.com/indigo-dc/image-classification-tf/tree/master/imgclas/webpage/README.md>`_ to use the trained classifier to predict images (both local and urls) on your favorite brownser.


Launching the full API
----------------------------

**Preliminaries for prediction**

If you want to use the API for prediction,  you have to do some preliminary steps to select the model you want to predict with:

* Copy your desired `.models/[timestamp]` to `.models/api`. If there is no `.models/api` folder, the default is to use the last available timestamp.
* In the `.models/api/ckpts` leave only the desired checkpoint to use for prediction. If there are more than one chekpoints, the default is to use the last available checkpoint.

Running the API
--------------------

To access this package's complete functionality (both for training and predicting) through an API you have to follow the instructions here: :ref:`api-integration`

