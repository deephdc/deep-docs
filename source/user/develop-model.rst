Develop a model using DEEP UC template
======================================

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


Develop a model according to DEEP UC template
---------------------------------------------

The structure of ``your_project`` created using 
`DEEP UC template <https://github.com/indigo-dc/cookiecutter-data-science>`__ contains 
the following core items needed to develop a model
::
	requirements.txt
	data/
	models/
	{{cookiecutter.repo_name}}/dataset/make_dataset.py
	{{cookiecutter.repo_name}}/features/build_features.py
	{{cookiecutter.repo_name}}/models/model.py
	
**Installing development requirements**

Modify ``requirements.txt`` according to your needs (e.g. add more libraries) then run
::
	$ pip install -r requirements.txt
	

**Improve the initial code**

You can modify as well as add more source files and put them accordingly into the directory structure.

**1. Make datasets:** source files in this directory aim to manipulate with raw dataset(s). 
The output of this step is raw data, which can be cleaned and/or pre-formatted.
::
	{{cookiecutter.repo_name}}/dataset/make_dataset.py
	{{cookiecutter.repo_name}}/dataset/

**2. Build features** takes the output from the previous step (Make datasets) and 
creates ML train, test as well as validation data from raw data.
The concrete realisation is depend on concrete UC, the aim of the application as well as 
technological background (e.g. high-performance supports).
::
	{{cookiecutter.repo_name}}/features/build_features.py
	{{cookiecutter.repo_name}}/features/

**3. Develop models** dealing with the most interesting ML core i.e. modelling. 
The most important thing in the ``model.py`` are implementations for DEEP entry points, 
which are defined according to :ref:`API methods <user/overview/api:Methods>`. 
You don't need to implement all the methods, just the ones you need.
::
	{{cookiecutter.repo_name}}/models/model.py
	{{cookiecutter.repo_name}}/models/

