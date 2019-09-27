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

   $ prominence create --openmpi \
                       --nodes 4 \
                       --cpus 2 \
                       --memory 8 \
                       --disk 20 \
                       --runtime 1000 \
                       alahiff/geant4mpi:1.3a3

By default a 10 GB disk is available to jobs, which is located on separate block storage. For MPI jobs the disk is available across all nodes running the job. The default maximum runtime is 720 minutes.

Working directory
^^^^^^^^^^^^^^^^^

By default the current working directory is scratch space made available inside the container. The path to this directory is also specified by the environment variables HOME, TEMP and TMP.

To specify a different working directory use ``--workdir``. For example, the following will run ``pwd`` inside the "/tmp" directory.

.. code-block:: console

   $ prominence create --workdir /tmp centos:7 pwd

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

The list command by default lists any active jobs, i.e. jobs which are idle or running:

.. code-block:: console

   $ prominence list
   ID     STATUS   IMAGE                       CMD     ARGS
   3101   idle     alahiff/testpi
   3103   idle     alahiff/cherab-jet:latest   python  batch_make_sensitivity_matrix.py 0 59
   3104   idle     ikester/blender:latest      blender -b classroom/classroom.blend -o frame_### -f 

It’s also possible to request a list of jobs using a constraint on the labels associated with each job. For example, if you submitted a group of jobs with a label name=run5, the following would list all such jobs:

.. code-block:: console

   $ prominence list --all --constraint name=run5

Here the ``--all`` option means that both active (i.e. idle or running) and completed jobs will be listed.

Describing a job
^^^^^^^^^^^^^^^^

To get more information about an individual job, use the describe command, for example:

.. code-block:: console

   $ prominence describe 345
   [
     {
       "id": 345,
       "status": "created",
       "resources": {
         "cpus": 1,
         "disk": 10,
         "memory": 1,
         "nodes": 1,
         "walltime": 720
       },
       "tasks": [
         {
           "image": "alahiff/testpi",
           "runtime": "singularity"
         }
       ],
       "events": {
         "createTime": "2019-06-18T10:16:36"
       }
     }
   ]

To show information about completed jobs, both the list and describe commands accept a ``--completed`` option. For example, to list the last 2 completed jobs:

.. code-block:: console

   $ prominence list --completed --last 2
   ID     STATUS      IMAGE                       CMD          ARGS
   2980   completed   alahiff/tensorflow:1.11.0   python       models-1.11/official/mnist/mnist.py --export_dir mnist_saved_model
   2982   completed   alahiff/tensorflow:1.11.0   python       models-1.11/official/mnist/mnist.py --export_dir mnist_saved_model

Note that jobs which are completed or have been removed for some reason may be visible briefly without using the ``--completed`` option.

Completed jobs
^^^^^^^^^^^^^^

The JSON descriptions of completed jobs contain additional information. This may include:

* **status**: current job status.
* **statusReason**: for jobs in a terminal state other than the completed state this may give a reason for the current status.
* **createTime**: date & time when the job was created by the user.
* **startTime**: date & time when the job started running.
* **endTime**: date & time when the job ended.
* **site**: the site where the job was executed.
* **maxMemoryUsageKB**: the maximum total memory usage of the job, summed over all processes (note this is not available for jobs running on remote HTC or HPC resources)

The following information is also provided for each task:

* **retries**: the number of retries attempted.
* **exitCode**: the exit code returned by the user’s job. This would usually would be 0 for success.
* **imagePullTime**: time taken to pull the container image. If a cached image from a previous task was used this will be -1.
* **wallTimeUsage**: wall time used by the task.
* **cpuTimeUsage**: CPU time usage by the task. For a task using multiple CPUs this will be larger than the wall time.
* **maxResidentSetSizeKB**: maximum resident size (in KB) of the largest process


Deleting a job
--------------

Jobs cannot be modified after they are created but they can be deleted. The delete command allows you to kill a single job:

.. code-block:: console

   $ prominence delete 164
   Success

