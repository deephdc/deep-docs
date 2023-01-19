.. highlight:: console

Develop a model from scratch
============================

This tutorial explains how to develop a DEEP module from scratch on your local machine.

You could also use the **DEEP Development environment** from the :doc:`Dashboard<../overview/dashboard>`
if you want to develop your code in a ready made environment based on some predefined Docker container
(eg. official Tensorflow or Pytorch containers). The tutorial still applies the same.
You only need to go to the Dashboard, select the **DEEP Development environment** and
configure the Docker image and resources you want to use
(see `video demo <https://www.youtube.com/watch?v=J_l_xWiBGNA&list=PLJ9x9Zk1O-J_UZfNO2uWp2pFMmbwLvzXa&index=3>`__).

.. admonition:: Requirements

    * If you plan to use the **DEEP Development environment**, you need  a `DEEP-IAM <https://iam.deep-hybrid-datacloud.eu/>`__ account to be able to access the Dashboard.
    * For **Step 7** we recommend having `docker <https://docs.docker.com/install/#supported-platforms>`__ installed (though it's not strictly mandatory).

1. Setting the framework
------------------------

First start by running :doc:`the DEEP Modules Template <../overview/cookiecutter-template>`:

.. code-block::

    $ pip install cookiecutter
    $ cookiecutter https://github.com/deephdc/cookiecutter-deep --checkout master

After answering the configuration questions, this will create two project directories:

.. code-block:: console

    ~/DEEP-OC-<project-name>
    ~/<project-name>

Go to Github and create the corresponding repositories:

* ``https://github.com/<github-user>/<project-name>``
* ``https://github.com/<github-user>/DEEP-OC-<project-name>``

Do a ``git push origin --all`` in both created directories to put your initial code in Github.


2. Editing ``<project-name>`` code
----------------------------------

Install your project as a Python module in **editable** mode (so that the changes you make to the codebase are picked by Python).

.. code-block:: console

    $ cd <project-name>
    $ pip install -e .

Now you can start writing your code.

To be able to interface with DEEPaaS :ref:`you have to define <user/overview/api:1. Define the API methods for your model>`
in ``api.py`` the functions you want to make accessible to the user.
For this tutorial we are going to head to our `official demo module <https://github.com/deephdc/demo_app/blob/master/demo_app/api.py>`__
and copy-paste its ``api.py`` file.

Once this is done, check that DEEPaaS is interfacing correctly by running:

.. code-block:: console

    $ deep-start --deepaas

Your module should be visible in http://0.0.0.0:5000/ui .
If you don't see your module, you probably messed the ``api.py`` file.
Try running it with python so you get a more detailed debug message.

.. code-block:: console

    $ python api.py

Remember to leave untouched the ``get_metadata()`` function that comes predefined with your module,
as all modules should have proper metadata.

In order to improve the readability of the code and the overall maintainability of the project,
we enforce proper Python coding styles (``pep8``) to all modules added to the Marketplace.
Modules that fail to pass style tests won't be able to build docker images.
If you want to check if your module pass the tests, go to your project folder and type:

.. code-block:: console

    $ flake8

There you should see a detailed report of the offending lines (if any).
You can always `turn off flake8 testing <https://stackoverflow.com/a/64431741>`__
in some parts of the code if long lines are really needed.

.. tip::

    If your project has many offending lines, it's recommended using a code formatter tool like
    `Black <https://black.readthedocs.io>`__. It also helps for having a consistent code style
    and minimizing git diffs. Black formatted code will always be compliant with flake8.

    Once `installed <https://black.readthedocs.io/en/stable/getting_started.html#installation>`__,
    you can check how Black would have reformatted your code:

    .. code-block:: console

        $ black <code-folder> --diff

    You can always `turn off Black formatting <https://black.readthedocs.io/en/stable/the_black_code_style/current_style.html?highlight=fmt#code-style>`__
    if you want to keep some sections of your code untouched.

    If you are happy with the changes, you can make them permanent using:

    .. code-block:: console

        $ black <code-folder>

    Remember to have a backup before reformatting, just in case!

Once you are fine with the state of ``<project-name>`` folder, push the changes to Github.


3. Editing ``DEEP-OC-<project-name>`` code
------------------------------------------

This is the repo in charge of creating a single docker image that integrates
your application, along with deepaas and any other dependency.

You need to modify the following files according to your needs:

* ``Dockerfile``: check the installation steps are fine. If your module needs additional
  Linux packages add them to the Dockerfile.
  Check your Dockerfile works correctly by building it locally and running it:

.. code-block:: console

    $ docker build --no-cache -t your_project .
    $ docker run -ti -p 5000:5000 -p 6006:6006 -p 8888:8888 your_project  #

Your module should be visible in http://0.0.0.0:5000/ui .
You can make a POST request to the ``predict`` method to check everything is working as intended.

* ``metadata.json``: this is the information that will be displayed in the Marketplace.
  Update and add the information you need.
  Check you didn't mess up the JSON formatting by running:

.. code-block:: console

    $ pip install git+https://github.com/deephdc/schema4apps
    $ deep-app-schema-validator metadata.json

Once you are fine with the state of ``DEEP-OC-<project-name>``, push the changes to Github.


4. Integrating the module in the Marketplace
--------------------------------------------

Once your repo is set, it's time to make a PR to add your model to the marketplace!

For this you have to fork the code of the DEEP catalog repo (`deephdc/deep-oc <https://github.com/deephdc/deep-oc>`__)
and add your Docker repo name at the end of the ``MODULES.yml``.

.. code-block:: yaml

    - module: https://github.com/deephdc/UC-<github-user>-DEEP-OC-<project-name>

You can do this directly `online on GitHub <https://github.com/deephdc/deep-oc/edit/master/MODULES.yml>`__ or via the command line:

.. code-block:: console

    $ git clone https://github.com/[my-github-fork]
    $ cd [my-github-fork]
    $ echo '- module: https://github.com/deephdc/UC-<github-user>-DEEP-OC-<project-name>' >> MODULES.yml
    $ git commit -a -m "adding new module to the catalogue"
    $ git push

Once the changes are done, make a PR of your fork to the original repo and wait for approval.
Check the `GitHub Standard Fork & Pull Request Workflow <https://gist.github.com/Chaser324/ce0505fbed06b947d962>`__ in case of doubt.

When your module gets approved, you may need to commit and push a change to ``metadata.json``
in your ``https://github.com/<github-user>/DEEP-OC-<project-name>`` so that
`the Pipeline <https://github.com/deephdc/DEEP-OC-demo_app/blob/726e068d54a05839abe8aef741b3ace8a078ae6f/Jenkinsfile#L104>`__
is run for the first time, and your module gets rendered in the marketplace.
