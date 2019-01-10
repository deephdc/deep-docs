Repository template
===================

To simplify development and in easy way integrate your model with DEEPaaS API, 
a project template, `cookiecutter-data-science <https://github.com/indigo-dc/cookiecutter-data-science>`_ [*]_, is provided in our GitHub.

In order to create your project based on the template, one has to `install <https://cookiecutter.readthedocs.io/en/latest/installation.html>`_ and then run 
`cookicutter <https://cookiecutter.readthedocs.io/en/latest/>`_ tool as follows::

    $ cookiecutter https://github.com/indigo-dc/cookiecutter-data-science
    
You will be asked several questions, e.g.:

* User account at github.com, e.g. 'indigo-dc'
* Project name
* Repository name for the new project
* (main) author name
* Email of the author (or contact person)
* Short description of the project
* Application version
* Choose open source license
* Version of python interpreter
* Docker Hub account name
* Base image for Dockerfile

.. note::  These parameters are defined in cookiecutter.json in the `cookiecutter-data-science <https://github.com/indigo-dc/cookiecutter-data-science>`_ source.

When these questions are answered, following two repositories will be created locally and immediately linked to your github.com account::

	~/DEEP-OC-your_project
	~/your_project




------------------

.. [*] A more general `cockiecutter-data-science <http://drivendata.github.io/cookiecutter-data-science/>`_ template was adapted for the purpose of DEEP.
