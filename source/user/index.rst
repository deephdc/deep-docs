User documentation
==================

.. admonition:: Useful project links
   :class: important

    | :fa:`home` `Homepage <https://deep-hybrid-datacloud.eu/>`__
    |   A high level overview of the project.
    | :fa:`book` `Documentation <https://docs.deep-hybrid-datacloud.eu/en/latest/>`__
    |   The main source of knowledge on how to use the project. Refer always to here in case of doubt.
    | :fa:`rotate` `Marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`__
    |   Where users will typically search for modules developed by the community, and find the relevant pointers to use them.
    | :fa:`sliders` `Dashboard <https://train.deep-hybrid-datacloud.eu/>`__
    |   Deploy virtual machines on specific hardware (eg. gpus) to train a module. Access is restricted to authenticated users.
    | :fa:`id-badge` `DEEP IAM <https://iam.deep-hybrid-datacloud.eu/>`__
    |   The authentication manager of the project, where you should register to get access to the Dashboard for example.
    | :fa:`database` `NextCloud <https://data-deep.a.incd.pt/>`__
    |   The service that allows to store your data remotely and access them from inside your deployment.
    | :fa:`github` `Github <https://github.com/deephdc>`__
    |   The code of all the modules and services behind the project is stored.
    | :fa:`docker` `DockerHub <https://hub.docker.com/u/deephdc/>`__
    |   Where the Docker images of the modules are stored.
    | :fa:`temperature-half` `Status of services <https://status.deep-hybrid-datacloud.eu/>`__
    |   Check if a specific DEEP service might be down for some reason.

    You can also watch this `tutorial <https://www.youtube.com/watch?v=cRMIviobF_c>`__
    for a quick overview of the project from the user's point of view (`attached slides <https://cdn.jsdelivr.net/gh/deephdc/deep-docs@master/source/_static/overview.pdf>`__).


New to the project? How about a quick dive?

.. toctree::
   :maxdepth: 2

   Quickstart <quickstart>

Overview
--------

A more in depth documentation, with detailed description on the architecture and
components is provided in the following sections.

.. toctree::
   :maxdepth: 2

   DEEP architecture <overview/architecture>
   User roles and workflows <overview/user-roles>
   DEEP Modules Template <overview/cookiecutter-template>
   DEEPaaS API <overview/api>
   DEEP Dashboard <overview/dashboard>

How-to's
--------

Use a model (basic user)
^^^^^^^^^^^^^^^^^^^^^^^^

.. toctree::
   :maxdepth: 2

   Perform inference locally <howto/inference-locally>

..
  // comment because right now deepaas serverless is not working correctly,
  // thus this guide can be potentially confusing.
  // TODO: uncomment when ready
  Perform inference remotely <howto/inference-remotely>

Train a model (intermediate user)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. toctree::
   :maxdepth: 2

   Train a model locally <howto/train-model-locally>
   Train a model remotely <howto/train-model-remotely>
   Use rclone <howto/rclone>

Develop a model (advanced user)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. toctree::
   :maxdepth: 2

   Develop a model <howto/develop-model>

Others
^^^^^^

.. toctree::
   :maxdepth: 1

   Video demos <howto/video-demos>

More
----

.. toctree::
   :maxdepth: 1

   Modules <modules/index>
