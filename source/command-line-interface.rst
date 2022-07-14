.. toctree::

Command line interface
======================

Installation
------------

The PROMINENCE CLI can be installed from PyPI, or if preferred, it can be run using containers. It is possible for normal users to install the PROMINENCE CLI without having to request any assistance from their system administrators.

With sudo or root access
::::::::::::::::::::::::

The PROMINENCE CLI can be installed on a host by typing the following:

.. code-block:: console

   $ sudo pip install prominence-cli

As a normal user without using a virtual environment
::::::::::::::::::::::::::::::::::::::::::::::::::::

The PROMINENCE CLI can be installed in a user’s home directory by running:

.. code-block:: console

   $ pip install --user prominence-cli

The directory ``.local/bin`` will need to be added to the ``PATH``.

As a normal user using a virtual environment
::::::::::::::::::::::::::::::::::::::::::::

The PROMINENCE CLI can be installed in a new virtual environment, e.g.

.. code-block:: console

   $ mkdir ~/.virtualenvs
   $ python3 -m venv ~/.virtualenvs/prominence
   $ source ~/.virtualenvs/prominence/bin/activate
   $ pip install prominence-cli

If ``pip`` is not available for some reason it can be installed in a user’s home directory by typing the following before running the above:

.. code-block:: console

   $ curl https://bootstrap.pypa.io/get-pip.py -o get-pip.py
   $ python3 get-pip.py --user
   $ pip install --user virtualenv

Checking what version is installed
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The version of the PROMINENCE CLI installed can be checked by running:

.. code-block:: console

   $ prominence --version

Setup
-----

Create a configuration directory for PROMINENCE:

.. code-block:: console

   $ mkdir ~/.prominence

Help
----

The main help page gives a list of all the available commands:

.. code-block:: console

   $ prominence --help
   usage: prominence [-h] [--version]
                     {run,create,list,describe,delete,stdout,stderr} ...

   Prominence - run jobs in containers across clouds
 
   positional arguments:
     {run,create,list,describe,delete,stdout,stderr}
                           sub-command help
       create              Create a job
       run                 Create a job or workflow from JSON or YAML in a file or URL
       list                List jobs or workflows
       describe            Describe a job or workflow
       delete              Delete a job or workflow
       stdout              Get standard output from a running or completed job
       stderr              Get standard error from a running or completed job

   optional arguments:
     -h, --help            show this help message and exit
     --version             show the version number and exit

Prerequisites
-------------

In order to use the EGI PROMINENCE service you must be a member of one of the following VOs and have the **vm_operator** role:

* vo.access.egi.eu
* fusion

Support for additional VOs can be added upon request.


Getting an access token
-----------------------

In order to interact with the PROMINENCE service via the CLI an access token is required. Go to the `<https://eosc.prominence.cloud>`_ and click on **Login** to log in with your Check-in credentials to obtain an access token.


The FedCloud Check-in client also provides the exact command to run to generate an access token. The PROMINENCE CLI requires the output of this command to be stored in the file ``~/.prominence/token``. The command to run will be of the form:

.. code-block:: console

   $ curl -X POST -u '<client id>':'<client secret>' -d 'client_id=<client id>&client_secret=<client secrer>&grant_type=refresh_token&refresh_token=<refresh token>&scope=openid%20email%20profile' 'https://aai.egi.eu/oidc/token' > ~/.prominence/token

.. note::
   Since PROMINENCE uses a REST API every request needs to be authenticated and hence requires an access token. The access token is not stored in any way by the server.

Your first job
--------------

Run the following command in order to submit a simple test job by typing ``prominence create eoscprominence/testpi``, i.e.

.. code-block:: console

   $ prominence create eoscprominence/testpi
   Job created with id 7375

Here ``eoscprominence/testpi`` is the name of the container image on Docker Hub. This command will submit a job which runs a container from the ``eoscprominence/testpi`` image using the default entrypoint specified in the image. In this case it is a simple Python script which calculates pi in three different ways.

To check the status of the job, run ``prominence list`` to list all currently active jobs:

.. code-block:: console

   $ prominence list
   ID     NAME   CREATED               STATUS    ELAPSED      IMAGE                   CMD
   7375          2019-10-14 12:33:11   pending                eoscprominence/testpi 

Eventually the status will change to ``runnning``, ``completed`` and then disappear. The ``list`` command can be given the argument ``--completed`` to show completed jobs. For example, to see the most recently completed job:

.. code-block:: console

    $ prominence list --completed
    ID     NAME   CREATED               STATUS      ELAPSED      IMAGE                   CMD
    7375          2019-10-14 12:33:11   completed   0+00:00:15   eoscprominence/testpi

Once the test job has finished running, ``prominence stdout`` can be used to view the standard output, e.g.

.. code-block:: console

   $ prominence stdout 7375
                            Plouff                  Bellard                         Chudnovsky
   Iteration number  1   3.133333333333333333333333   3.141765873015873015873017   3.141592653589734207668453
   Iteration number  2   3.141422466422466422466422   3.141592571868390306374053   3.141592653589793238462642
   Iteration number  3   3.141587390346581523052111   3.141592653642050769944284   3.141592653589793238462642
   Iteration number  4   3.141592457567435381837004   3.141592653589755368080514   3.141592653589793238462642
   Iteration number  5   3.141592645460336319557021   3.141592653589793267843377   3.141592653589793238462642
   Iteration number  6   3.141592653228087534734378   3.141592653589793238438852   3.141592653589793238462642
   Iteration number  7   3.141592653572880827785241   3.141592653589793238462664   3.141592653589793238462642
   Iteration number  8   3.141592653588972704940778   3.141592653589793238462644   3.141592653589793238462642
   Iteration number  9   3.141592653589752275236178   3.141592653589793238462644   3.141592653589793238462642
   Iteration number  10   3.141592653589791146388777   3.141592653589793238462644   3.141592653589793238462642
   Iteration number  11   3.141592653589793129614171   3.141592653589793238462644   3.141592653589793238462642
   Iteration number  12   3.141592653589793232711293   3.141592653589793238462644   3.141592653589793238462642
   Iteration number  13   3.141592653589793238154767   3.141592653589793238462644   3.141592653589793238462642
   Iteration number  14   3.141592653589793238445978   3.141592653589793238462644   3.141592653589793238462642
   Iteration number  15   3.141592653589793238461733   3.141592653589793238462644   3.141592653589793238462642
   Iteration number  16   3.141592653589793238462594   3.141592653589793238462644   3.141592653589793238462642
   Iteration number  17   3.141592653589793238462641   3.141592653589793238462644   3.141592653589793238462642
   Iteration number  18   3.141592653589793238462644   3.141592653589793238462644   3.141592653589793238462642
   Iteration number  19   3.141592653589793238462644   3.141592653589793238462644   3.141592653589793238462642

The next sections of the documentation describe in more detail how to run more complex jobs and workflows.

Jobs and workflows
------------------

By default all PROMINENCE CLI commands refer to jobs. However, a number of commands include the ability to specify a resource, which is either a ``job`` or a ``workflow``.

Listing workflows:

.. code-block:: console

   prominence list workflows

Describing a specific workflow:

.. code-block:: console

   prominence describe workflow <id>

Deleting a workflow:

.. code-block:: console

   prominence delete workflow <id>

The standard output and error from a job which is part of a workflow can be viewed by specifying both the workflow id and the name of the job, i.e.

.. code-block:: console

   prominence stdout <id> <job name>

To list all the individual jobs from workflow <id>:

.. code-block:: console

   prominence list jobs <id>


