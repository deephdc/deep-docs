Useful Machine Learning resources
=================================

This is a piece of documentation trying to offer some advice on tools to 
use to answer common problems (non ML expert) users might face.


Tutorials
---------

Here are some basic resources to get you quickly started in the Deep Learning / Machine Learning world.

Books
^^^^^

* *Deep Learning with Python*, F. Chollet
* *The FastAI book*
* *Deep Learning Book*, Ian Goodfellow  


Courses
^^^^^^^

* `Google Machine Learning Crash Course <https://developers.google.com/machine-learning/crash-course>`__
* `Machine Learning Mastery <https://machinelearningmastery.com/start-here/>`__
* `DAIR ML YouTube Courses <https://github.com/dair-ai/ML-YouTube-Courses>`__
* `Stanford CS231N (2017): Introduction to Convolutional Neural Networks for Visual Recognition <https://www.youtube.com/playlist?list=PL3FW7Lu3i5JvHM8ljYj-zLfQRF3EO8sYv>`__
* `Stanford CS230 (2018): Deep Learning <https://www.youtube.com/playlist?list=PLoROMvodv4rOABXSygHTsbvUz4G_YQhOb>`__




Datasets
--------

Dataset labeling
^^^^^^^^^^^^^^^^

Some tools to help you getting started creating your dataset.

* `LabelStudio <https://labelstud.io/>`__ - General annotation (text, images, etc)
* `LabelImg <https://github.com/tzutalin/labelImg>`__ - Image annotation
* `refinery <https://github.com/code-kern-ai/refinery>`__ - Labeling for NLP
* `superintendent <https://github.com/janfreyberg/superintendent>`__ - ipywidget-based interactive labelling tool for your data.
* `VGG Image Annotator (VIA) <https://www.robots.ox.ac.uk/~vgg/software/via/>`__ - Image annotation
* `Roboflow <https://roboflow.com/annotate>`__ - only free is your dataset is public
* `Labelbox <https://labelbox.com/>`__ - paid tool (free with educational license)


Find a dataset
^^^^^^^^^^^^^^

If you don't have any data, try find an open dataset that suits you.

* `Google Dataset search <https://datasetsearch.research.google.com/>`__
* `Graviti Open Datassets <https://gas.graviti.com/open-datasets>`__
* `DataHub <https://datahub.io/collections>`__
* `Kaggle <https://www.kaggle.com/>`__
* `Paperwithcode Datasets <https://paperswithcode.com/datasets>`__
* `OpenML datasets <https://www.openml.org/search?type=data&status=active>`__
* `Dataworld datasets <https://data.world/datasets/agriculture>`__
* `Eden library <https://edenlibrary.ai/>`__ - agriculture datasets



Explore your dataset
^^^^^^^^^^^^^^^^^^^^

Less make sure the dataset does not contain errors.

* `Google's Know your data <https://knowyourdata.withgoogle.com/>`__ - only valid for common Tensorflow Datasets
* `Sweetviz <https://github.com/fbdesignpro/sweetviz>`__ - explore and compare tabular data
* `cleanlab <https://github.com/cleanlab/cleanlab>`__ - dataset cleaning
* `FastDup <https://github.com/visualdatabase/fastdup>`__ - dataset cleaning. Find anomalies, duplicate and near duplicate images, clusters of similarity, broken images, image statistics, wrong labels.
* `deepchecks <https://github.com/deepchecks/deepchecks>`__ - checks related to various types of issues, such as model performance, data integrity, distribution mismatches, and more.
* `kangas <https://github.com/comet-ml/kangas>`__ -  exploring, analyzing, and visualizing large-scale multimedia data
* `Impyute <https://github.com/eltonlaw/impyute>`__ - missing data



Feature selection
^^^^^^^^^^^^^^^^^

Some times less is more. Learn how to select the appropriate features of your dataset.

* `sklearn - feature selection <https://scikit-learn.org/stable/modules/classes.html#module-sklearn.feature_selection>`__
* `mlxtend <https://rasbt.github.io/mlxtend/>`__


Imbalanced learning
^^^^^^^^^^^^^^^^^^^

Do you have too much data from one class and too few from others. Let's balance things out!

* `Sklearn imbalanced <https://github.com/scikit-learn-contrib/imbalanced-learn>`__


Data augmentation
^^^^^^^^^^^^^^^^^

Do you have few data? Make the most out of it!

* `Augly <https://github.com/facebookresearch/AugLy>`__ - General augmentation (text, images, etc.)
* `imgaug <https://github.com/aleju/imgaug>`__ - Image augmentation


Dataset shift
^^^^^^^^^^^^^

Is your dataset likely to degrade over time (eg. cam gets dirty). Keep on eye on it!

* `Alibi-detect <https://github.com/SeldonIO/alibi-detect>`__
* `Avalanche <https://github.com/ContinualAI/avalanche>`__ - Continual Learning library based on Pytorch
* `River <https://github.com/online-ml/river>`__ - Online learning
* `Frouros <https://github.com/IFCA/frouros>`__ - Jaime's library
* `Cinnamon <https://github.com/zelros/cinnamon>`__
* `Eurybia <https://github.com/MAIF/eurybia>`__


Models
------

Model development
^^^^^^^^^^^^^^^^^

If you want to develop a model from scratch don't try to be a hero!
`Papers with Code <https://paperswithcode.com/>`__ gathers top performing models
for multiple tasks with their corresponding code. Reuse them for your usecases! Try not to look
for the top model but for the one with the cleanest code.


Training monitoring
^^^^^^^^^^^^^^^^^^^

Let's keep an eye on the training status.

* `Tensorboard <https://github.com/tensorflow/tensorboard>`__ - only works with Tensorflow
* `TensorboardX <https://github.com/lanpa/tensorboardX>`__ - framework agnostic
* `LabML <https://github.com/labmlai/labml>`__


Training debugging
^^^^^^^^^^^^^^^^^^

Is your training failing for some reason?

* `Netron <https://github.com/lutzroeder/netron>`__ - visualize DL models 
* `Cockpit <https://github.com/f-dangel/cockpit>`__ - debug training


Model optimization
^^^^^^^^^^^^^^^^^^

Do you need your model to go faster?

* `VoltaML <https://github.com/VoltaML/voltaML>`__ - accelerate ML models with a single line of code
* `sparse-ml <https://github.com/neuralmagic/sparseml>`__
* `deep-sparse <https://github.com/neuralmagic/deepsparse>`__
* `Pytorch quantization <https://pytorch.org/docs/stable/quantization.html>`__
* `AItemplate <https://github.com/facebookincubator/AITemplate>`__ - transforms deep neural networks into CUDA (NVIDIA GPU) / HIP (AMD GPU) C++ code for lightning-fast inference serving
* `Hummingbird <https://github.com/microsoft/hummingbird>`__ - transform traditional Ml models (eg. Random Forest) to neural networks, and benefit from hardware acceleration
