User documentation
==================

.. admonition:: Useful project links
   :class: important

    * `Project's Homepage <https://deep-hybrid-datacloud.eu/>`__
    * `Marketplace <https://marketplace.deep-hybrid-datacloud.eu/>`__
    * `Dashboard <https://train.deep-hybrid-datacloud.eu/>`__
    * `Github <https://github.com/deephdc>`__
    * `DockerHub <https://hub.docker.com/u/deephdc/>`__
    * `Docs <https://docs.deep-hybrid-datacloud.eu/en/latest/>`__
    * `NextCloud <https://nc.deep-hybrid-datacloud.eu/>`__
    * `DEEP IAM <https://iam.deep-hybrid-datacloud.eu/>`_


    You can also check these `slides <https://cdn.jsdelivr.net/gh/deephdc/deep-docs@master/source/_static/overview.pdf>`_
    along with this `video <https://www.youtube.com/watch?v=cRMIviobF_c>`_  for a quick overview of the project
    from the user's point of view.


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
  // commment because right now deepaas serverless is not working correctly,
  // thus this can be potentially confusing
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
