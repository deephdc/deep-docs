.. include:: <isonum.txt>
.. highlight:: console

DEEP Modules Template
=====================

.. admonition:: Useful video demos
   :class: important

    - `Data science cookiecutter template <https://www.youtube.com/watch?v=mCxz8LQJWWA&list=PLJ9x9Zk1O-J_UZfNO2uWp2pFMmbwLvzXa&index=7>`__


Overview
--------

To simplify the development and in an easy way integrate your model with the :doc:`DEEPaaS API <api>`,
a `standard template <https://github.com/deephdc/cookiecutter-deep>`__ for modules is provided.

There are different versions of this template:

* `master <https://github.com/deephdc/cookiecutter-deep/tree/master>`__: this is what 99% of users are probably
  looking for. Simple, minimal template, with the minimum requirements to integrate your code in DEEP.
* `advanced <https://github.com/deephdc/cookiecutter-deep/tree/advanced>`__: this is a more advanced template.
  It makes more assumptions on how to structure projects and adds more files than those strictly needed for integration.
  Unless you are looking for some specific feature, you are probably safer using master.
* `child-module <https://github.com/deephdc/cookiecutter-deep/tree/child-module>`__: this is a fork of the `master` branch specifically tailored to users performing a retraining of an existing module. It only creates a Docker repo whose container is based on an existing module's Docker image.

In order to create your project based on the template, one has to `install <https://cookiecutter.readthedocs.io/en/latest/installation.html>`__
and then run the `cookiecutter <https://cookiecutter.readthedocs.io>`__ tool as follows
::

    $ pip install cookiecutter
    $ cookiecutter https://github.com/deephdc/cookiecutter-deep -b master

You are first provided with ``[Info]`` line about the parameter and in the next line you configure this parameter.
You will be asked to configure:

* ``git_base_url``: Remote URL to host your new git repositories (e.g. https://github.com/deephdc ).
* ``project_name``: Project name.
* ``repo_name``: Name of your new repository, to be added after \"git_base_url\" (see above)", (aka <your_project> in the following).
* ``author_name``: Author name(s) (and/or your organization/company/team). If many, separate by comma.
* ``author_email``: E-Mail(s) of main author(s) (or contact person). If many, separate by comma.
* ``description``: Short description of the project.
* ``app_version``: Application version (expects X.Y.Z (Major.Minor.Patch)).
* ``open_source_license``: Choose open source license, default is MIT. `More info <https://opensource.org/licenses>`__.
* ``docker_baseimage``: Docker image your Dockerfile starts from (`FROM <docker_baseimage>`) (don't provide the tag here), (e.g. tensorflow/tensorflow ).
* ``baseimage_cpu_tag``: CPU tag for the baseimage, e.g. 2.9.1. Has to match python3!
* ``baseimage_gpu_tag``: GPU tag for the baseimage, e.g. 2.9.1-gpu. Has to match python3!
* ``failure_notify``: whether you want to receive updates if your model fails to build.

When these questions are answered, following two repositories will be created locally and immediately linked to your ``git_base_url``:

* ``~/your_project``: Main repository to integrate model
* ``~/DEEP-OC-your_project``: Repository for the integration of the :doc:`DEEPaaS API <api>` and your_project in one Docker image.

Each repository of these has two branches: ``master`` (to commit stable changes) and ``test`` (to test features
without disrupting your users).

Project structure
-----------------

Based on the on the branch you choose, the template will create different files, being master the most minimal option (see above).
The content of these files is populated based on your answer to the questions.

**Master branch**

.. code-block:: console

    <your_project>
    ##############
    ├── LICENSE                <- License file
    │
    ├── README.md              <- The top-level README for developers using this project.
    │
    ├── requirements.txt       <- The requirements file for reproducing the analysis
    │                              environment (`pip freeze > requirements.txt`)
    │
    ├── setup.py, setup.cfg    <- makes project pip installable (pip install -e .) so
    │                             {{cookiecutter.repo_name}} can be imported
    │
    ├── {{cookiecutter.repo_name}}    <- Source code for use in this project.
    │   │
    │   ├── __init__.py        <- Makes {{cookiecutter.repo_name}} a Python module
    │   │
    │   ├── api.py             <- Main script for the integration with DEEP API
    │   │
    │   ├── misc.py            <- Misc functions that were helpful across projects
    │   │
    │   └── tests              <- Scripts to perform code testing
    │
    └── Jenkinsfile            <- Describes basic Jenkins CI/CD pipeline


    DEEP-OC-<your_project>
    ######################
    ├─ Dockerfile             <- Describes main steps on integration of DEEPaaS API and
    │                            <your_project> application in one Docker image
    │
    ├─ Jenkinsfile            <- Describes basic Jenkins CI/CD pipeline
    │
    ├─ LICENSE                <- License file
    │
    ├─ README.md              <- README for developers and users.
    │
    └─ metadata.json          <- Defines information propagated to the DEEP Marketplace


**Advanced branch**

.. code-block:: console

    <your_project>
    ##############
    ├── LICENSE
    ├── README.md              <- The top-level README for developers using this project.
    ├── data
    │   └── raw                <- The original, immutable data dump.
    │
    ├── docs                   <- A default Sphinx project; see sphinx-doc.org for details
    │
    ├── models                 <- Trained and serialized models, model predictions, or model
    │                             summaries
    │
    ├── notebooks              <- Jupyter notebooks. Naming convention is a number
    │                             (for ordering), the creator's initials (if many
    │                             user development), and a short `_` delimited
    │                             description.
    │                             e.g.`1.0-jqp-initial_data_exploration.ipynb`.
    │
    ├── references             <- Data dictionaries, manuals, and all other explanatory
    │                             materials.
    │
    ├── reports                <- Generated analysis as HTML, PDF, LaTeX, etc.
    │   └── figures            <- Generated graphics and figures to be used in reporting
    │
    ├── requirements.txt       <- The requirements file for reproducing the analysis
    │                             environment, (`pip freeze > requirements.txt`)
    │
    ├── test-requirements.txt  <- The requirements file for the test environment
    │
    ├── setup.py               <- makes project pip installable (pip install -e .) so
    │                             {{cookiecutter.repo_name}} can be imported
    │
    ├── {{cookiecutter.repo_name}}    <- Source code for use in this project.
    │   │
    │   ├── __init__.py        <- Makes {{cookiecutter.repo_name}} a Python module
    │   │
    │   ├── dataset            <- Scripts to download or generate data
    │   │   └── make_dataset.py
    │   │
    │   ├── features           <- Scripts to turn raw data into features for modeling
    │   │   └── build_features.py
    │   │
    │   ├── models             <- Scripts to train models and make predictions
    │   │   └── deep_api.py    <- Main script for the integration with DEEP API
    │   │
    │   ├── tests              <- Scripts to perform code testing
    │   │
    │   └── visualization      <- Scripts to create exploratory and results oriented
    │       └── visualize.py      visualizations
    │
    └── tox.ini                <- tox file with settings for running tox; see tox.testrun.org

    DEEP-OC-<your_project>
    ######################
    ├─ Dockerfile             <- Describes main steps on integration DEEPaaS API and
    │                            <your_project> application in one Docker image
    │
    ├─ Jenkinsfile            <- Describes basic Jenkins CI/CD pipeline
    │
    ├─ LICENSE                <- License file
    │
    ├─ README.md              <- README for developers and users.
    │
    ├─ docker-compose.yml     <- Allows running the application with various configurations
    │                            via docker-compose
    │
    └─ metadata.json          <- Defines information propagated to the DEEP Marketplace
