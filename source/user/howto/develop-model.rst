.. highlight:: console

Develop a model
===============

Setting the framework
---------------------

Run :doc:`the DEEP Modules Template <../overview/cookiecutter-template>`.
This creates two project directories:
::

    ~/DEEP-OC-your_project
    ~/your_project

Go to ``github.com/your_account`` and create corresponding repositories: ``DEEP-OC-your_project`` and ``your_project``
Do ``git push origin --all`` in both created directories. This puts your initial code in Github.


Editing ``your_project`` code
-----------------------------

Install your project as a Python module in `editable` mode (so that the changes you make to the codebase are picked by Python).
::

    cd your_project
    pip install -e .

Now you can start writing your code. To be able to interface with DEEPaaS
:ref:`you have to define <user/overview/api:1. Define the API methods for your model>` in ``api.py`` the
functions you want to make accessible to the user.
Once this is done check that DEEPaaS is interfacing correctly by running:
::

    deepaas-run --listen-ip 0.0.0.0
    # --> your module should be visible in http://0.0.0.0:5000/ui

If you don't see your module, you probably messed the ``api.py`` file.
Try running it with python so you get a more detailed debug message.
::

    python api.py

Once you are fine with the state of your module, push the changes to Github.


Editing ``DEEP-OC-your_project`` code
-------------------------------------

This is the repo in charge of creating a single docker image that integrates
your application, along with deepaas and any other dependency.

You need to modify the following files according to your needs:

* ``Dockerfile``: check the installation steps are fine. If your module needs additional
  Linux packages add them to the Dockerfile.
  Check your Dockerfile works correctly by building it locally and running it:
  ::

    docker build --no-cache -t your_project .
    docker run -ti -p 5000:5000 -p 6006:6006 -p 8888:8888 your_project
    # --> your module should be visible in http://0.0.0.0:5000/ui

* ``metadata.json``: this is the information that will be displayed in the Marketplace.
  Update and add the information you need.
  Check you didn't mess up the JSON formatting by running:
  ::

    pip install git+https://github.com/deephdc/schema4apps
    deep-app-schema-validator metadata.json

Once you are fine with the state of your module, push the changes to Github.


Integrating the module in the Marketplace
-----------------------------------------

Once your repos are set, it's time to make a PR to add your model to the marketplace!

For this you have to fork the code of the DEEP catalog repo (`deephdc/deep-oc <https://github.com/deephdc/deep-oc>`_)
and add your Docker repo name at the end of the ``MODULES.yml``.
You can do this directly `online on GitHub <https://github.com/deephdc/deep-oc/edit/master/MODULES.yml>`_ or via the command line:

.. code-block:: console

    git clone https://github.com/[my-github-fork]
    cd [my-github-fork]
    echo '- module: https://github.com/[my-account-name]/DEEP-OC-[my-app-name]' >> MODULES.yml
    git commit -a -m "adding new module to the catalogue"
    git push

Once the changes are done, make a PR of your fork to the original repo and wait for approval.
Check the `GitHub Standard Fork & Pull Request Workflow <https://gist.github.com/Chaser324/ce0505fbed06b947d962>`_ in case of doubt.

When your module gets approved, you may need to commit and push a change to ``metadata.json``
in ``DEEP-OC-your_project`` (`ref <https://github.com/deephdc/DEEP-OC-demo_app/blob/726e068d54a05839abe8aef741b3ace8a078ae6f/Jenkinsfile#L104>`__)
so that the Pipeline is run for the first time, and your module gets rendered in the marketplace.
