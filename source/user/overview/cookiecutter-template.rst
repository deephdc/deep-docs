.. highlight:: console

DEEP Data Science template
==========================

To simplify the development and in an easy way integrate your model with the :doc:`DEEPaaS API <api>`,
a project template, `cookiecutter-data-science <https://github.com/indigo-dc/cookiecutter-data-science>`__ [*]_, is provided in our GitHub.

In order to create your project based on the template, one has to `install <https://cookiecutter.readthedocs.io/en/latest/installation.html>`_ and then run
`cookicutter <https://cookiecutter.readthedocs.io/en/latest/>`_ tool as follows::

    $ cookiecutter https://github.com/indigo-dc/cookiecutter-data-science

You are first provided with [Info] line about the parameter and in the next line you configure this parameter. You will be asked to configure:

* Remote URL to host your new repositories (git), e.g. https://github.com/deephdc, ``git_base_url``
* Project name, ``project_name``
* Name of your new repository, to be added after \"git_base_url\" (see above)", ``repo_name`` (aka <your_project> in the following)
* Author name(s) (and/or your organization/company/team). If many, separate by comma, ``author_name``
* E-Mail(s) of main author(s) (or contact person). If many, separate by comma, ``author_email``
* Short description of the project, ``description``
* Application version (expects X.Y.Z (Major.Minor.Patch)), ``app_version``
* Choose open source license, default is MIT. For more info: https://opensource.org/licenses, ``open_source_license``
* User account at hub.docker.com, e.g. 'deephdc' in https://hub.docker.com/u/deephdc, ``dockerhub_user``
* Docker image your Dockerfile starts from (FROM <docker_baseimage>) (don't provide the tag here), e.g. tensorflow/tensorflow, ``docker_baseimage``
* CPU tag for the baseimage, e.g. 1.14.0-py3. Has to match python3!, ``baseimage_cpu_tag``
* GPU tag for the baseimage, e.g. 1.14.0-gpu-py3. Has to match python3!, ``baseimage_gpu_tag``

.. note::  These parameters are defined in ``cookiecutter.json`` in the `cookiecutter-data-science <https://github.com/indigo-dc/cookiecutter-data-science>`__ source.

When these questions are answered, following two repositories will be created locally and immediately linked to your ``git_base_url``::

	~/DEEP-OC-your_project
	~/your_project

each repository has two branches: 'master' and 'test'.

<your_project> repo
-------------------

Main repository to integrate model with the following structure::

    |
    ├── data                   Placeholder for the data
    │   └── raw                   The original, immutable data dump.
    │
    ├── docs                   Documentation on the project; see sphinx-doc.org for details
    │
    ├── models                 Trained and serialized models, model predictions, or model summaries
    │
    ├── notebooks              Jupyter notebooks. Naming convention is a number (for ordering),
    │                            the creator's initials (if many user development),
    │                            and a short `_` delimited description,
    │                            e.g. `1.0-jqp-initial_data_exploration.ipynb`.
    │
    ├── references             Data dictionaries, manuals, and all other explanatory materials.
    │
    ├── reports                Generated analysis as HTML, PDF, LaTeX, etc.
    │
    ├── your_project           Main source code of the project
    │    │
    │    ├── __init__.py          Makes your_project a Python module
    │    │
    │    ├── dataset              Scripts to download and manipulate raw data
    │    │   └── make_dataset.py
    │    │
    │    ├── features             Scripts to prepare raw data into features for modeling
    │    │   └── build_features.py
    │    │
    │    ├── models               Scripts to train models and then use trained models to make predictions
    │    │   └── deep_api.py         Main script for the integration with DEEP API
    │    │
    │    ├── tests                Scripts to perfrom code testing
    │    │
    │    └── visualization        Scripts to create exploratory and results oriented visualizations
    │        └── visualize.py
    │
    ├── .dockerignore          Describes what files and directories to exclude for building a Docker image
    │
    ├── .gitignore             Specifies intentionally untracked files that Git should ignore
    │
    ├── Jenkinsfile            Describes basic Jenkins CI/CD pipeline
    │
    ├── LICENSE                License file
    │
    ├── README.md              The top-level README for developers using this project.
    │
    ├── requirements.txt       The requirements file for reproducing the analysis environment,
    │                             e.g. generated with `pip freeze > requirements.txt`
    │
    ├── setup.cfg              makes project pip installable (pip install -e .)
    │
    ├── setup.py               makes project pip installable (pip install -e .)
    │
    ├── test-requirements.txt  The requirements file for the test environment
    │
    └── tox.ini                tox file with settings for running tox; see tox.testrun.org


Certain files, e.g. ``README.md``, ``Jenkinsfile``, ``setup.cfg``, ``tox.ini``, etc are pre-populated
based on the answers you provided during cookiecutter call (see above).


<DEEP-OC-your_project>
----------------------

Repository for the integration of the :doc:`DEEPaaS API <api>` and your_project in one Docker image.
::

    ├─ Dockerfile     Describes main steps on integrationg DEEPaaS API and
    │                     your_project application in one Docker image
    │
    ├─ Jenkinsfile    Describes basic Jenkins CI/CD pipeline
    │
    ├─ LICENSE        License file
    │
    ├─ README.md      README for developers and users.
    │
    ├─ docker-compose.yml     Allows running the application with various configurations via docker-compose
    │
    ├─ metadata.json          Defines information propagated to the DEEP Open Catalog, https://marketplace.deep-hybrid-datacloud.eu


All files get filled with the info provided during cookiecutter execution (see above).

Step-by-step guide
-------------------
#. (if not yet done) install cookiecutter, as e.g. ``pip install cookiecutter``
#. run ``cookiecutter https://github.com/indigo-dc/cookiecutter-data-science``
#. answer all the questions, pay attention about docker tags!
#. two directories will be created: <your_project> and <DEEP-OC-your_project> (each with two git branches: master and test)
#. go to github.com/user_account and create corresponding repositories <your_project> and <DEEP-OC-your_project>
#. go to your terminal, <your_project>, ``git push origin --all``
#. go to your terminal, <DEEP-OC-your_project>, ``git push origin --all``
#. your github repositories are now updated with initial commits
#. you can build <deep-oc-your_project> Docker image locally: go to <DEEP-OC-your_project> directory, do ``docker build -t dockerhubuser/deep-oc-your_project .``
#. you can now run deepaas as ``docker run -p 5000:5000 dockerhubuser/deep-oc-your_project``

------------------

.. [*] The more general `cookiecutter-data-science <http://drivendata.github.io/cookiecutter-data-science/>`__ template was adapted for the purpose of DEEP.
