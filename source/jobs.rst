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

* CPU cores

* Memory (in GB)

* Disk (in GB)

* Maximum runtime (in mins)

CPU cores and memory can be specified using the ``--cpus`` and ``--memory`` options. A disk size can also be specified using ``--disk``.

Here is an example running an MPI job on 4 nodes where each node has 2 CPUs and 8 GB memory, there is a shared 20 GB disk accessible by all 4 nodes, and the maximum runtime is 1000 minutes:

.. code-block:: console

   $ prominence create --openmpi --nodes 4 --cpus 2 --memory 8 --disk 20 --runtime 1000 alahiff/geant4mpi:1.3a3

By default a 10 GB disk is available to jobs, which is located on separate block storage. For MPI jobs the disk is available across all nodes running the job. The default maximum runtime is 720 minutes.

Working directory
^^^^^^^^^^^^^^^^^

By default the current working directory is scratch space made available inside the container. The path to this directory is also specified by the environment variables HOME, TEMP and TMP.

To specify a different working directory use ``--workdir``.

.. note::
   Remember that you should not try to write inside the container’s filesystem as this may be prevented by the container runtime or result in permission problems.

Environment variables
^^^^^^^^^^^^^^^^^^^^^

It is a common technique to use environment variables to pass information, such as configuration options, into a container. The option ``--env`` can be used to specify an environment variable in the form of a key-value pair separated by “=”. This option can be specified multiple times to set multiple environment variables. For example:

.. code-block:: console

   $ prominence create --env LOWER=4.5 --env UPPER=6.7 test/container

Multiple tasks in a single job
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By default a job will run a single command inside a single container. However, it is possible to instead run multiple sequential tasks within a single job. Each task will have access to the same temporary storage, so transient files generated by one task are accessible by other tasks.

Standard output and error
-------------------------

The standard output and error from a job can be seen using the stdout and stderr commands. For example, to get the standard output for the job with id 299:

.. code-block:: console

   $ prominence stdout 299
    _______
   < hello >
    -------
       \
        \
         \
                       ##        .
                 ## ## ##       ==
              ## ## ## ##      ===
          /""""""""""""""""___/ ===
     ~~~ {~~ ~~~~ ~~~ ~~~~ ~~ ~ /  ===- ~~~
          \______ o          __/
           \    \        __/
             \____\______/


.. note::
   The standard output and error can be seen while jobs are running as well as once they have completed, allowing users to check the status of long-running jobs. This is not quite realtime, so there can be short delays in the standard output and error being updated.

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


