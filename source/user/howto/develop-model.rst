.. highlight:: console

**************************************
Develop a model using DEEP UC template
**************************************


Prepare DEEP UC environment
---------------------------

Install cookiecutter (if not yet done)
::

	$ pip install cookiecutter
	
Run the DEEP UC cookiecutter template
::

	$ cookiecutter https://github.com/indigo-dc/cookiecutter-data-science
	
Answer all questions from DEEP UC cookiecutter template with attentions to 
``repo_name`` i.e. the name of your github repositories, etc.
This creates two project directories:
::

	~/DEEP-OC-your_project
	~/your_project
	
Go to ``github.com/your_account`` and 
create corresponding repositories: ``DEEP-OC-your_project`` and ``your_project``
Do ``git push origin master`` in both created directories. This puts your initial code to ``github``.


2. Improve the initial code of the model
----------------------------------------

The structure of ``your_project`` created using 
`DEEP UC template <https://github.com/indigo-dc/cookiecutter-data-science>`_ contains
the following core items needed to develop a DEEP UC model:
::

	requirements.txt
	data/
	models/
	{{repo_name}}/dataset/make_dataset.py
	{{repo_name}}/features/build_features.py
	{{repo_name}}/models/model.py


2.1 Installing development requirements
=======================================

Modify ``requirements.txt`` according to your needs (e.g. add more libraries) then run
::

	$ pip install -r requirements.txt
	
You can modify and add more ``source files`` and put them 
accordingly into the directory structure.


2.2 Make datasets 
=================

Source files in this directory aim to manipulate raw datasets.
The output of this step is also raw data, but cleaned and/or pre-formatted.
::

	{{cookiecutter.repo_name}}/dataset/make_dataset.py
	{{cookiecutter.repo_name}}/dataset/


2.3 Build features
===================

This step takes the output from the previous step `Make datasets` and
creates train, test as well as validation ML data from raw but cleaned and pre-formatted data.
The realisation of this step depends on the concrete Use Case, the aim of the application as well as
available technological backgrounds (e.g. high-performance supports for data processing).
::

	{{cookiecutter.repo_name}}/features/build_features.py
	{{cookiecutter.repo_name}}/features/


2.4 Develop models
==================

This step deals with the most interesting phase in ML i.e. modelling. 
The most important thing of DEEP UC models is located in ``model.py``
containing DEEP entry point implementations. 
DEEP entry points are defined using :ref:`API methods <user/overview/api:Methods>`. 
You don't need to implement all of them, just the ones you need.
::

	{{cookiecutter.repo_name}}/models/model.py
	{{cookiecutter.repo_name}}/models/
