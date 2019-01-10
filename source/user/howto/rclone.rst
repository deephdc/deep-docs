..    include:: <isonum.txt>




How to use rclone
=================


Installation of rclone in Docker image (pro)
------------------------------------------
All the applications in the `DEEP Open Catalog <https://deephdc.github.io/>`_ are packed within a Docker containing rclone installed by default. If you want to create a Docker containing your own application, you should install rclone in the container to be able to access the data stored remotely. The following lines are an example of what has to be added in the Dockerfile when installation is based on Ubuntu. For other Linux flavors, please, refer to the `rclone official site  <https://rclone.org/downloads/>`_  ::

	# install rclone
		RUN wget https://downloads.rclone.org/rclone-current-linux-amd64.deb && \
   		dpkg -i rclone-current-linux-amd64.deb && \
   		apt install -f && \
   		rm rclone-current-linux-amd64.deb && \
    		apt-get clean && \
    		rm -rf /var/lib/apt/lists/* && \
    		rm -rf /root/.cache/pip/* && \
    		rm -rf /tmp/*

 
Nextcloud configuration for rclone
------------------------------------------
.. image:: ../../_static/nc-access.png

After login into `DEEP-Nextcloud  <https://nc.deep-hybrid-datacloud.eu/login>`_ with your DEEP-IAM credentials, go to (1) **Settings (top right corner)** |rarr|  (2) **Security**  |rarr|  (3) **Devices & sessions**. Set a name for you application and clik on **Create new app password**. That user and password is what one needs to include in the rclone config file (rclone.conf). 


Creating rclone.conf
-------------------------

You can install rclone at your host or run Docker image with rclone installed (see necessary installation of clone in Docker image above). In order to create the cofiguration file (rclone.conf) for rclone::
    
    	$ rclone config
   	 choose "n"  for "New remote"
   	 choose name for DEEP-Nextcloud, e.g. deep-nextcloud
   	 choose "Type of Storage" \u2192 "Webdav" (24)
   	 provide DEEP-Nextcloud URL for webdav access: https://nc.deep-hybrid-datacloud.eu/remote.php/webdav/
   	 choose Vendor, Nextcloud (1)
   	 specify "user" (see "Nextcloud configuration for rclone" above). Your username starts with "DEEP-IAM-..."
   	 specify password (see "Nextcloud configuration for rclone" above).
   	 by default rclone.conf is created in your $HOME/.config/rclone/rclone.conf

If you create rclone.conf from your running Docker container, ensure to copy it to your host, i.e. outside of container. **DO NOT STORE IT IN THE CONTAINER** (e.g. if you use uDocker, it will be stored in your filesystem, even being in the container).

Then one may have two options:

If your know under what user your run your application in the container, e.g. if docker or nvidia-docker is used, most probably this is 'root', you can mount your located at your host rclone.conf into the container as::
    
        $ docker run -ti -v $HOSTDIR_WITH_RCLONE_CONF/rclone.conf:/root/.config/rclone/rclone.conf <your-docker-image>

i.e. you mount rclone.conf file itself directly as a volume.

One can also mount rclone directory with the rclone.conf file::

  	$ docker run -ti -v $HOSTDIR_WITH_RCLONE_CONF:/root/.config/rclone <your-docker-image>

A more reliable way can be to mount either rclone directory or directly rclone.conf file into a pre-defined location and not (container) user-depended place::

    	$ docker run -ti -v $HOSTDIR_WITH_RCLONE_CONF:/rclone <your-docker-image>

One has, however, later call rclone with "--config" option to point to the rclone.conf file, e.g::

        $ rclone --config /rclone/rclone.conf ls deep-nextcloud:/Datasets/dogs_breed/models
