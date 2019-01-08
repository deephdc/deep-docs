Develop a model, using cookiecutter
===================================

Prepare DEEP UC template environment
--------------------------------

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

Do ``git push origin master`` in both created directories. This puts your initial code to github.


Develop a model using the created DEEP UC template
--------------------------------------------------

The structure of ``your_project`` contains the following core items needed to develop a model
::
	requirements.txt
	{{cookiecutter.repo_name}}/dataset/make_dataset.py
	{{cookiecutter.repo_name}}/features/build_features.py
	{{cookiecutter.repo_name}}/models/model.py
	
You can also add more source files and put them into the directory structure.

**Installing development requirements**

Modify ``requirements.txt`` according to your needs (e.g. add more libraries)
::
	pip install -r requirements.txt
	

**Make datasets**


**Build features***


**Develop models***
	