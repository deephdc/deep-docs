.. include:: <isonum.txt>
.. highlight:: console

Rclone code examples
====================

Example code on usage of rclone from python
-------------------------------------------

Simple example
^^^^^^^^^^^^^^

A simple call of rclone from python is via ``subprocess.Popen()``

.. code-block:: python

    import subprocess

    # from "rshare" remote storage into the container
    command = (['rclone', 'copy', 'rshare:/Datasets/dogs_breed/data', '/srv/dogs_breed_det/data'])

    result = subprocess.Popen(command, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
    output, error = result.communicate()

.. important::
    When deploying a module on the DEEP Pilot testbed, you pass rclone parameters e.g. ``rclone_user`` and ``rclone_password`` during the deployment.
    If you use our `general template <https://github.com/indigo-dc/tosca-templates/blob/master/deep-oc/deep-oc-marathon-webdav.yml>`__ , the name of the remote storage has to be ``rshare`` as in the example above (``rshare:/Datasets/dogs_breed/data``). If you create your own TOSCA template, you need to pay attention on matching these names in your code and in the template (for example, see environment parameters in the `general template <https://github.com/indigo-dc/tosca-templates/blob/master/deep-oc/deep-oc-marathon-webdav.yml>`_ like RCLONE_CONFIG_RSHARE_USER etc).

Advanced examples
^^^^^^^^^^^^^^^^^

More advanced usage includes calling rclone with various options (ls, copy, check) in order to check file existence at
Source, check if after copying two versions match exactly.

* rclone_call

.. code-block:: python

    def rclone_call(src_path, dest_dir, cmd = 'copy', get_output=False):
        """ Function
           rclone calls
        """
         if cmd == 'copy':
            command = (['rclone', 'copy', '--progress', src_path, dest_dir])
         elif cmd == 'ls':
            command = (['rclone', 'ls', src_path])
         elif cmd == 'check':
            command = (['rclone', 'check', src_path, dest_dir])

         if get_output:
            result = subprocess.Popen(command, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
         else:
             result = subprocess.Popen(command, stderr=subprocess.PIPE)
         output, error = result.communicate()
         return output, error


* rclone_copy

.. code-block:: python

    def rclone_copy(src_path, dest_dir, src_type='file', verbose=False):
        """ Function for rclone call to copy data (sync?)
        :param src_path: full path to source (file or directory)
        :param dest_dir: full path to destination directory (not file!)
        :param src_type: if source is file (default) or directory
        :return: if destination was downloaded, and possible error
        """

        error_out = None

        if src_type == 'file':
            src_dir = os.path.dirname(src_path)
            dest_file = src_path.split('/')[-1]
            dest_path = os.path.join(dest_dir, dest_file)
        else:
            src_dir = src_path
            dest_path =  dest_dir

        # check first if we find src_path
        output, error = rclone_call(src_path, dest_dir, cmd='ls')
        if error:
            print('[ERROR, rclone_copy()] %s (src):\n%s' % (src_path, error))
            error_out = error
            dest_exist = False
        else:
            # if src_path exists, copy it
            output, error = rclone_call(src_path, dest_dir, cmd='copy')
            if not error:
                output, error = rclone_call(dest_path, dest_dir,
                                            cmd='ls', get_output=True)
                file_size = [ elem for elem in output.split(' ') if elem.isdigit() ][0]
                print('[INFO] Copied to %s %s bytes' % (dest_path, file_size))
                dest_exist = True
                if verbose:
                    # compare two directories, if copied file appears in output
                    # as not found or not matching -> Error
                    print('[INFO] File %s copied. Check if (src) and (dest) really match..' % (dest_file))
                    output, error = rclone_call(src_dir, dest_dir, cmd='check')
                    if 'ERROR : ' + dest_file in error:
                        print('[ERROR, rclone_copy()] %s (src) and %s (dest) do not match!'
                              % (src_path, dest_path))
                        error_out = 'Copy failed: ' + src_path + ' (src) and ' + \
                                     dest_path + ' (dest) do not match'
                        dest_exist = False
            else:
                print('[ERROR, rclone_copy()] %s (src):\n%s' % (dest_path, error))
                error_out = error
                dest_exist = False

        return dest_exist, error_out



.. code-block:: python

    import subprocess

    def sync_nextcloud(frompath, topath):
        """
        Mount a NextCloud folder in your local machine or viceversa.
        """
        command = (['rclone', 'copy', frompath, topath])
        result = subprocess.Popen(command, stdout=subprocess.PIPE, stderr=subprocess.PIPE)
        output, error = result.communicate()
        if error:
            warnings.warn("Error while mounting NextCloud: {}".format(error))
        return output, error

    sync_nextcloud('rshare:/your/dataset/folder', '/your/data/path/inside/the/container') # sync local with nextcloud
    sync_nextcloud('/your/data/path/inside/the/container', 'rshare:/your/dataset/folder') # sync nextcloud with local

