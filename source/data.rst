.. toctree::

Data
====

It is possible for jobs to access data from external storage systems in a POSIX-like way, like a standard filesystem. Users can specify the mount point to be used inside the container. `B2DROP <https://b2drop.eudat.eu>`_, `OneData <https://onedata.org>`_ (including EGI's `DataHub <https://datahub.egi.eu>`_) and WebDAV are supported.

Container images can be obtained from this storage if necessary. In this case the image should be in the form of a Singularity image or Docker archive.

B2DROP
------

The following JSON needs to be included in every job description where access to B2DROP is required:

.. code-block:: console

   "storage":{
     "type":"b2drop",
     "mountpoint":"/data",
     "b2drop":{
       "app-username":"***",
       "app-password":"***"
     }
   }

where the app username and password should be set as appropriate. The mount point ``/data`` here is just an example and can be replaced with something else.

Note that the app username and password are not the same as the username and password used to access B2DROP. To create an app username and password, login to https://b2drop.eudat.eu/, then select **Settings** then **Security** and click **Create new app password**.

Using the PROMINENCE CLI to create jobs, the ``--storage`` option can be used to specify the name of a JSON file containing the above content. For example:

.. code-block:: console

   prominence create --storage my-b2drop.json ...

OneData
-------

In order to mount your OneData storage in jobs firstly an access token needs to be created using the **Access tokens** menu in the OneData web interface.

The following JSON needs to be included in every job description where access to OneData is required:

.. code-block:: console

   "storage":{
     "type":"onedata",
     "mountpoint":"/data",
     "onedata":{
       "provider":"***",
       "token":"***"
     }
   }

where the provider hostname and access token should be set as appropriate. The mount point ``/data`` here is just an example and can be replaced with something else.

WebDAV
------

The following JSON needs to be included in every job description where access to a storage system providing WebDAV is required:

.. code-block:: console

   "storage":{
     "type":"webdav",
     "mountpoint":"/data",
     "webdav":{
       "url":"***",
       "username":"***",
       "password":"***"
     }
   }

In this case a URL is required in addition to a username and password.
