Architecture overview
---------------------


There are several different components in the DEEP-HDC project that are relevant for the users. Later on you will see
how each :doc:`different type of user <user-roles>` can take advantage of the different components.

The marketplace
===============

This is one of the most important components. It is a catalogue of modules that every user can have access to. We can
divide the modules in two different categories:

* **Models** are modules (eg. an image classifier) that an user can train on their own data to create a new tool.

* **Tools** are models that have been trained for a specific task (eg. an plant classifier).

For more information have a look at the marketplace.

.. todo:: Add final links to marketplace, image classifier and plant classifier


The API
=======

The DEEPaaS API is a key component for making the modules accessible to everybody. It is a graphical interface that the
user can take advantage of to interact with the model. It is available for both inference and training. Advanced user
that want to create new models can make them compatible with the API to make them available to the whole community.

For more information take a look on the full :doc:`API guide <api>`.


The storage resources
=====================

Storage is essential for user that want to create new tools by training models on their custom data. For the moment we
support saving data in NextCloud.

.. todo:: Check rclone integration with Dropbox and Google Drive