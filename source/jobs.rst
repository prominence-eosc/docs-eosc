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

   $ prominence create centos:7 "/bin/sleep 100"

The command of course should exist within the container. If arguments need to be specified you should put the command and any arguments inside a single set of double quotes, as in the example above.

To run multiple commands inside the same container, use ``/bin/bash -c`` with the commands enclosed in quotes and seperated by semicolons, for example:

.. code-block:: console

   $ prominence create centos:7 "/bin/bash -c \"date; sleep 10; date\""

This is of course assuming ``/bin/bash`` exists inside the container image.

MPI jobs
^^^^^^^^

To run an MPI job, you need to specify either ``--openmpi`` for Open MPI, ``--intelmpi`` for Intel MPI and ``--mpich`` for MPICH. For multi-node jobs the number of nodes required should also be specified. For example:

.. code-block:: console

   $ prominence create --openmpi --nodes 4 alahiff/openmpi-hello-world:latest /mpi_hello_world

The number of processes to run per node is assumed to be the same as the number of cores available per node. If the number of cores available per node is more than the requested number of cores all cores will be used. This behaviour can be changed by using ``--procs-per-node`` to define the number of processes per node to use.

.. warning::
   Currently ``--procs-per-node`` is not supported for MPICH jobs.

Unlike single-node jobs, a command to run (and optionally any arguments) must be specified. If an entrypoint is defined in the container image it will be ignored.

Hybrid MPI-OpenMP jobs
^^^^^^^^^^^^^^^^^^^^^^

In this situation the number of MPI processes to run per node must be specified using ``--procs-per-node`` and the environment variable OMP_NUM_THREADS should be set to the required number of OpenMP threads per MPI process.

In the following example we have 2 nodes with 4 CPUs each, and we run 2 MPI processes on each node, where each MPI process runs 2 OpenMP threads:

.. code-block:: console

   $ prominence create --cpus 4 \
                       --memory 4 \
                       --nodes 2 \
                       --procs-per-node 2 \
                       --openmpi \
                       --env OMP_NUM_THREADS=2 \
                       --artifact https://github.com/lammps/lammps/archive/stable_12Dec2018.tar.gz \
                       --workdir lammps-stable_12Dec2018/bench \
                       alahiff/lammps-openmpi-omp "lmp_mpi -sf omp -in in.lj"

Resources
^^^^^^^^^

By default a job will be run with 1 CPU and 1 GB memory but this can easily be changed. The following resources can be specified:

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


