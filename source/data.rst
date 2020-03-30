.. toctree::

Data
====

It is possible for jobs to access data from external storage systems in a POSIX-like way, like a standard filesystem. Users can specify the mount point to be used inside the container. `B2DROP <https://b2drop.eudat.eu>`_, `OneData <https://onedata.org>`_ (including EGI's `DataHub <https://datahub.egi.eu>`_) and WebDAV are supported.

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

where the app username and password should be set as appropriate. Note that an application username and password are required, which are not the same as the username and password you use to access B2DROP. To create an application username and password, after logging in to https://b2drop.eudat.eu/ select **Settings** then **Security**. The mountpoint ``/data`` here is just an example and can be replaced with something else.

Using the PROMINENCE CLI to create jobs, the ``--storage`` option can be used to specify the name of a JSON file containing the above content. For example:

.. code-block:: console

   prominence create --storage my-onedata.json ...

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

where the provider hostname and access token should be set as appropriate. The mountpoint ``/data`` here is just an example and can be replaced with something else.

Using the PROMINENCE CLI to create jobs, the ``--storage`` option can be used to specify the name of a JSON file containing the above content. For example:

.. code-block:: console

   prominence create --storage my-onedata.json ...

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

