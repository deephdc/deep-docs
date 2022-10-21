.. highlight:: console

Configure oidc-agent
====================

`oidc-agent <https://indigo-dc.gitbook.io/oidc-agent/>`__ is a tool to manage OpenID Connect tokens
and make them easily usable from the command line.

.. admonition:: Requirements

    * having a `DEEP IAM <https://iam.deep-hybrid-datacloud.eu/>`__ account
    * having the oidc-agent installed (follow the `official installation guide <https://indigo-dc.gitbook.io/oidc-agent/install>`_).

Start oidc-agent::

	$ eval $(oidc-agent)
	$ oidc-gen

You will be asked for the name of the account to configure. Let's call it **deep-iam**.
After that you will be asked for the additional *client-name-identifier*, you should choose the option::

		[2] https://iam.deep-hybrid-datacloud.eu/

Then just click ``Enter`` to accept the default values for ``Space delimited list of scopes [openid profile offline_access]``.
After that, if everything has worked properly, you should see the following messages::

	Registering Client ...
	Generating account configuration ...
	accepted

At this point you will be given a URL. You should visit it in the browser of your choice in order to continue and
approve the registered client. For this you will have to login into your DEEP-IAM account and accept the permissions
you are asked for.

Once you have done this you will see the following message::

	The generated account config was successfully added to oidc-agent. You don't have to run oidc-add

Next time you want to start oidc-agent from scratch, you will only have to do::

	$ eval $(oidc-agent)
	oidc-add deep-iam
	Enter encryption password for account config deep-iam: ********
	success

You can print the token::

	$ oidc-token deep-iam


**Usage with orchent**

You should set OIDC_SOCK (this is not needed, if you did it before)::

	$ eval $(oidc-agent)
        $ oidc-add deep-iam

Set the agent account to be used with orchent and the ORCHENT_URL::

	$ export ORCHENT_AGENT_ACCOUNT=deep-iam
	$ export ORCHENT_URL="https://paas.cloud.cnaf.infn.it/orchestrator"
