.. highlight:: console

************************************************
Develop a model using DEEP Data Science template
************************************************


1. Prepare DEEP Data Science environment
========================================


Install cookiecutter (if not yet done)
::

	$ pip install cookiecutter

Run the DEEP DS cookiecutter template
::

	$ cookiecutter https://github.com/indigo-dc/cookiecutter-data-science

Answer all questions from :ref:`Data Science template <user/overview/cookiecutter-template:DEEP Data Science template>`
with attentions to ``repo_name`` i.e. the name of your github repositories, etc.
This creates two project directories:
::

	~/DEEP-OC-your_project
	~/your_project

Go to ``github.com/your_account`` and create corresponding repositories: ``DEEP-OC-your_project`` and ``your_project``
Do ``git push origin --all`` in both created directories. This puts your initial code to ``github``.


2. Improve the initial code of the model
========================================

The structure of ``your_project`` created using
:ref:`Data Science template <user/overview/cookiecutter-template:DEEP Data Science template>` contains
the following core items needed to integrate your Machine learning model with DEEPaaS API:
::

	requirements.txt
	test-requirements.txt
	data/
	models/
	{{repo_name}}/dataset/make_dataset.py
	{{repo_name}}/features/build_features.py
	{{repo_name}}/models/deep_api.py
	{{repo_name}}/tests/test_unit_model.py


2.1 Installing project requirements
-----------------------------------

Modify ``requirements.txt`` according to your needs (e.g. add more libraries) then run
::

	$ pip install -r requirements.txt

You can modify and add more ``source files`` and put them
accordingly into the directory structure.


2.2 Make datasets
-----------------

Source files in this directory aim to manipulate raw datasets.
The output of this step is also raw data, but cleaned and/or pre-formatted.
::

	{{repo_name}}/dataset/make_dataset.py
	{{repo_name}}/dataset/


2.3 Build features
------------------

This step takes the output from the previous step "2.2 Make datasets" and
creates train, test as well as validation Machine Learning data from raw but cleaned and pre-formatted data.
The realisation of this step depends on the concrete use case, the aim of the application as well as
available technological backgrounds (e.g. high-performance supports for data processing).
::

	{{repo_name}}/features/build_features.py
	{{repo_name}}/features/


2.4 Develop models
------------------

This step deals with the most interesting phase in Machine Learning i.e. modelling.
The most important thing is located in ``deep_api.py`` containing DEEP entry point implementations.
DEEP entry points are defined using :ref:`API methods <user/overview/api:1. Define the API methods for your model>`.
You don't need to implement all of them, just the ones you need.
::

	{{repo_name}}/models/deep_api.py
	{{repo_name}}/models/


2.5 Code testing
----------------

Code testing, including unit testing, is a necessary part of modern application development.
If your project is built based on :ref:`Data Science template <user/overview/cookiecutter-template:DEEP Data Science template>`,
you get an example on how to test ``get_metadata()`` method and can certainly add more tests.
Also ``test-requirements.txt`` file requests installation of two python testing tools:
`stestr <https://pypi.org/project/stestr>`_ and `pytest <https://docs.pytest.org/en/latest>`_.
::

	{{repo_name}}/tests/test_unit_model.py
	{{repo_name}}/tests/


3. Connect with a remote storage
================================

If you expect your model to use remotely hosted data, you can upload the data in `DEEP-Nextcloud <https://nc.deep-hybrid-datacloud.eu>`__ and 
later trigger copying of data to your container using :doc:`rclone <rclone>` tool. The tool requires ``rclone.conf`` file to exist, even if it is an empty one. In the :doc:`"How to use rclone" <rclone>` document you find an extended information and :ref:`examples on how to use it from python<user/howto/rclone:Example code on usage of rclone from python>`.

.. tip::
    When developing an application with the :ref:`Data Science template <user/overview/cookiecutter-template:DEEP Data Science template>`,
    the Dockerfile already includes creation of an empty rclone.conf

.. important::
    **DO NOT** save the rclone credentials in the **CONTAINER**

4. Create a python installable package 
=======================================

To create a python installable package the initial directory structure should look something like this::

	your_model_package/
		your_model_package/
			__init__.py
		setup.py
		setup.cfg
		requirements.txt
		LICENSE
                README

* The top level directory will be the root of your repo, e.g. your_model_package.git. The subdir, also called your_model_package, is the actual python module.
* ``setup.py`` is the build script for setuptools. It tells setuptools about your package (such as the name and version) as well as which code files to include. You can find an example of a setup.py file `here <https://github.com/deephdc/image-classification-tf/blob/master/setup.py>`__. For the official documentation on how to write your setup script, you can go `here <https://docs.python.org/3.6/distutils/setupscript.html>`__.
* ``setup.cfg`` can be used to get some information from the user, or from the user's system in order to proceed. Configuration files also let you providedefault values for any command option. An example of a setup.cfg file can be found `here <https://github.com/deephdc/image-classification-tf/blob/master/setup.cfg>`__. The official python documentation on the setup configuration file can be found `here <https://docs.python.org/3.6/distutils/configfile.html>`__.
* ``requirements.txt`` contains any external requirement needed to run the package. An example of a requirements file can be found `here <https://github.com/deephdc/image-classification-tf/blob/master/requirements.txt>`_.
* The ``README`` file will contain information on how to run the package or anything else that you may find useful for someone running your package.
* ``LICENSE`` It’s important for every package uploaded to the Python Package Index to include a license. This tells users who install your package the terms under which they can use your package. For help choosing a license, go `here <https://choosealicense.com/>`__.

To see how to install your model package, check the Dockerfile in the next section.


5. Create a docker container for your model
===========================================

Once your model is well in place, you can encapsulate it by creating a docker image. For this you need to modify the Dockerfile created during execution of the :ref:`Data Science template <user/overview/cookiecutter-template:DEEP Data Science template>`. The Dockerfile is pre-populated with the information you provided while running the cookiecutter template. You may need, however, add packages you need installed to make your project run.

The simplest Dockerfile could look like this::

	FROM tensorflow/tensorflow:1.14.0-py3

	# Install ubuntu updates and python related stuff
	# Remember: DEEP API V2 only works with python 3.6 [!]
	RUN DEBIAN_FRONTEND=noninteractive apt-get update && \
	    apt-get install -y --no-install-recommends \
	         git \
	         curl \
	         wget \
	         python3-setuptools \
	         python3-pip \
	         python3-wheel && \
	    apt-get clean && \
	    rm -rf /var/lib/apt/lists/* && \
	    rm -rf /root/.cache/pip/* && \
	    rm -rf /tmp/*

	# Set LANG environment
	ENV LANG C.UTF-8

	WORKDIR /srv

	# Install rclone
	RUN wget https://downloads.rclone.org/rclone-current-linux-amd64.deb && \
	    dpkg -i rclone-current-linux-amd64.deb && \
	    apt install -f && \
	    mkdir /srv/.rclone/ && touch /srv/.rclone/rclone.conf && \
	    rm rclone-current-linux-amd64.deb && \
	    apt-get clean && \
	    rm -rf /var/lib/apt/lists/* && \
	    rm -rf /root/.cache/pip/* && \
	    rm -rf /tmp/*

	# Install DEEPaaS and FLAAT
	RUN pip install --no-cache-dir \
	    deepaas \
	    flaat

	# Download and install your project
	RUN git clone https://github.com/your_git/your_project && \
	    cd your_project && \
	    python -m pip install -e . && \
	cd ..

	# Expose API on port 5000 and monitoring port 6006
	EXPOSE 5000 6006

	CMD ["deepaas-run", "--openwhisk-detect", "--listen-ip", "0.0.0.0", "--listen-port", "5000"]


Check the :doc:`rclone guide <rclone>` and :doc:`DEEPaaS guide <../overview/api>` for more details.

If you want to see examples of more complex Dockerfiles, you can check various applications `here <https://github.com/deephdc?utf8=%E2%9C%93&q=DEEP-OC&type=&language=>`__ (look for DEEP-OC-xxx repositories), e.g. `this Dockerfile <https://github.com/deephdc/DEEP-OC-image-classification-tf/blob/master/Dockerfile>`_.

In order to compile the Dockerfile, you should choose a name for the docker image and use the docker build command::

	docker build -t your_docker_image -f Dockerfile .


You can then upload it to Docker hub so that you can download the already compiled image directly. To do so, follow the instructions `here <https://docs.docker.com/docker-hub/repos/>`__.

