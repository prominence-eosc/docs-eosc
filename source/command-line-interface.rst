.. toctree::

Command line interface
======================

Installation
------------

The PROMINENCE CLI can be installed from PyPI, or if preferred, it can be run using containers. It is possible for normal users to install the PROMINENCE CLI without having to request any assistance from their system administrators.

Using pip
^^^^^^^^^

With sudo or root access
::::::::::::::::::::::::

The PROMINENCE CLI can be installed on a host by typing the following:

.. code-block:: console

   $ sudo pip install prominence-cli

As a normal user without using virtualenv
:::::::::::::::::::::::::::::::::::::::::

The PROMINENCE CLI can be installed in a user’s home directory by running:

.. code-block:: console

   $ pip install --user prominence-cli

The directory ``.local/bin`` will need to be added to the ``PATH``.

As a normal user using virtualenv
:::::::::::::::::::::::::::::::::

The PROMINENCE CLI can be installed in a new virtual environment, e.g.

.. code-block:: console

   $ virtualenv ~/.virtualenvs/prominence
   $ source ~/.virtualenvs/prominence/bin/activate
   $ pip install prominence-cli

If ``virtualenv`` is not available it can be installed in a user’s home directory by typing the following before running the above:

.. code-block:: console

   $ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
   $ python3 get-pip.py --user
   $ pip install --user virtualenv

Using Singularity
^^^^^^^^^^^^^^^^^

An alternative is to use Singularity, if available, and create an alias for the command ``prominence``. Firstly pull the Docker image:

.. code-block:: console

   $ singularity pull docker://eoscprominence/cli

This will create a file ``cli_latest.sif``. An alias can be created by putting the following in your ``~/.bashrc`` file:

.. code-block:: console

   alias prominence="singularity run <path>/cli_latest.sif"

where the full path to the container image should be specified.

Using udocker
^^^^^^^^^^^^^

Similarly to Singularity, another alternative is to use udocker. Because udocker can be installed as an unprivileged user, this method can be used to allow the PROMINENCE CLI to be used on a UI or login node which does not have Singularity installed.

Firstly install udocker if necessary:

.. code-block:: console

   $ curl https://raw.githubusercontent.com/indigo-dc/udocker/master/udocker.py > udocker
   $ chmod u+rx ./udocker
   $ ./udocker install

and move the file ``udocker`` to somewhere in your ``PATH``. See `here <https://github.com/indigo-dc/udocker/blob/master/doc/installation_manual.md>`_ for more information.

Once udocker is installed, pull the image and create a container:

.. code-block:: console

   $ udocker pull eoscprominence/cli
   $ udocker create --name=prominence eoscprominence/cli:latest

An alias for the prominence command can be created by putting the following in your ``~/.bashrc`` file:

.. code-block:: console

   alias prominence="udocker run --bindhome --hostenv prominence"

Checking what version is installed
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The version of the PROMINENCE CLI installed can be checked by running:

.. code-block:: console

   $ prominence --version

Setup
-----

The URL of the PROMINENCE server needs to be defined in an environment variable. Add the following line to your ``~/.bashrc`` file:

.. code-block:: console

    export PROMINENCE_URL=https://prominence.fedcloud-tf.fedcloud.eu/api/v1

Create a configuration directory for PROMINENCE:

.. code-block:: console

   $ mkdir ~/.prominence

Help
----

The main help page gives a list of all the available commands:

.. code-block:: console

   $ prominence --help
   usage: prominence [-h] [--version]
              {login,create,run,list,describe,delete,upload,download,stdout,stderr}
              ...

   Prominence - run jobs in containers across clouds
 
   positional arguments:
     {register,login,run,create,list,describe,delete,upload,download,stdout,stderr}
                           sub-command help
       register            Register as a client with the OIDC server
       login               Get a token from the OIDC server
       create              Create a job or workflow from a JSON file
       run                 Run a job
       list                List jobs or workflows
       describe            Describe a job or workflow
       delete              Delete a job or workflow
       upload              Upload a file to transient storage
       download            Download output files from a completed job or workflow
       stdout              Get standard output from a running or completed job
       stderr              Get standard error from a running or completed job

   optional arguments:
     -h, --help            show this help message and exit
     --version             show the version number and exit

Jobs and workflows
------------------

By default all PROMINENCE CLI commands refer to jobs. However, a number of commands include the ability to specify a resource, which is either a job or a workflow.

Listing workflows:

.. code-block:: console

   $ prominence list workflows

Describing a workflow:

.. code-block:: console

   $ prominence describe workflow <workflow id>

Deleting a workflow:

.. code-block:: console

   $ prominence delete workflow <workflow id>

The standard output and error from a job which is part of a workflow can be viewed by specifying both the workflow id and the name of the job, i.e.

.. code-block:: console

   $ prominence stdout <id> <job name>

Getting an access token
-----------------------

In order to interact with the PROMINENCE service an access token is required. Go to the `EGI FedCloud Check-in client <https://aai.egi.eu/fedcloud>`_ and click on **Authorise** to log in with your Check-in credentials to obtain:

* a client id
* a client secret
* a refresh token

The refresh token allows you to generate access tokens without having to login every time.

The FedCloud Check-in client also provides the exact command to run to generate an access token. The PROMINENCE CLI requires the output of this command to be stored in the file ``~/.prominence/token``. The command to run will be of the form:

.. code-block:: console

   $ curl -X POST -u '<client id>':'<client secret>' -d 'client_id=<client id>&client_secret=<client secrer>&grant_type=refresh_token&refresh_token=<refresh token>&scope=openid%20email%20profile' 'https://aai.egi.eu/oidc/token' > ~/.prominence/token

.. note::
   Since PROMINENCE uses a REST API every request needs to be authenticated and hence requires an access token. The access token is not stored in any way by the server.
