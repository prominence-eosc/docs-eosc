.. toctree::

Jobs
====

Running jobs
------------

Single node jobs
^^^^^^^^^^^^^^^^

In order to run an instance of a container, running the command defined in the image’s entrypoint, all you need to do is to specify the Docker Hub image name:

.. code-block:: console

   $ prominence create eoscprominence/testpi
   Job created with id 3101

When a job has been successfully submitted an (integer) ID will be returned. Alternatively, a command (and arguments) can be specified. For example:

.. code-block:: console

   prominence create centos:7 "/bin/sleep 100"

The command of course should exist within the container. If arguments need to be specified you should put the command and any arguments inside a single set of double quotes, as in the example above.

To run multiple commands inside the same container, use ``/bin/bash -c`` with the commands enclosed in quotes and seperated by semicolons, for example:

.. code-block:: console

   prominence create centos:7 "/bin/bash -c \"date; sleep 10; date\""

This is of course assuming ``/bin/bash`` exists inside the container image.

MPI jobs
^^^^^^^^

To run an MPI job, you need to specify either ``--openmpi`` for Open MPI, ``--intelmpi`` for Intel MPI and ``--mpich`` for MPICH. For multi-node jobs the number of nodes required should also be specified. For example:

.. code-block:: console

   prominence create --openmpi --nodes 4 alahiff/openmpi-hello-world:latest /mpi_hello_world

The number of processes to run per node is assumed to be the same as the number of cores available per node. If the number of cores available per node is more than the requested number of cores all cores will be used. This behaviour can be changed by using ``--procs-per-node`` to define the number of processes per node to use.

Hybrid MPI-OpenMP jobs
^^^^^^^^^^^^^^^^^^^^^^

Resources
^^^^^^^^^

Working directory
^^^^^^^^^^^^^^^^^

Environment variables
^^^^^^^^^^^^^^^^^^^^^

Multiple tasks in a single job
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Standard output and error
-------------------------

Checking job status
-------------------

Listing jobs
^^^^^^^^^^^^

Describing a job
^^^^^^^^^^^^^^^^

Completed jobs
^^^^^^^^^^^^^^

Deleting a job
--------------


