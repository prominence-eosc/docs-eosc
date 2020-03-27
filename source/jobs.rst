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
   The standard output and error can be seen while jobs are running as well as once they have completed, allowing users to check the status of long-running jobs.

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

It’s also possible to request a list of jobs using a constraint on the labels associated with each job. For example, if you submitted a group of jobs with a label ``name=run5``, the following would list all such jobs:

.. code-block:: console

   $ prominence list --all --constraint name=run5

Here the ``--all`` option means that both active (i.e. idle or running) and completed jobs will be listed.

Describing a job
^^^^^^^^^^^^^^^^

To get more information about an individual job, use the describe command, for example:

.. code-block:: console

   $ prominence describe 345
   {
     "id": , 
     "status": "idle", 
     "resources": {
       "nodes": 1, 
       "disk": 10, 
       "cpus": 1, 
       "memory": 1
     }, 
     "tasks": [
       {
         "image": "eoscprominence/testpi", 
         "runtime": "singularity"
       }
     ], 
     "events": {
       "createTime": "2019-10-04 18:07:40"
     }
   }

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

Labels
------

Arbitrary metadata in the form of key-value pairs can be associated with jobs.

Defining labels
^^^^^^^^^^^^^^^

Labels in the form of key-value pairs (separated by “=”) can be set using the ``--label`` option when creating a job. This option can be used multiple times to set multiple labels. For example:

.. code-block:: console

   $ prominence create --label experiment=MAST-U --label shot=12 test/container

Each key and value must be a string of less than 64 characters. Keys can only contain alphanumeric characters (``[a-z0-9A-Z]``) while values can also contain dashes (``-``), underscores (``_``), dots (``.``) and forward slashes (``/``).

Finding jobs with specific labels
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

It is possible to specify a constraint when using the list command. For example, to list all active jobs which have a label ``experiment`` set to ``MAST-U``:

.. code-block:: console

   $ prominence list --constraint experiment=MAST-U

To list all jobs, i.e. both active and completed jobs, with a specific label, add the `--all` option, e.g.

.. code-block:: console

   $ prominence list --constraint experiment=MAST-U --all


Generating JSON
---------------

When prominence create is run with the ``--dry-run`` option, the job will not be submitted but the JSON description of the job will be printed to standard output. For example:

.. code-block:: console
   
   $ prominence create --dry-run --name test1 --cpus 4 --memory 8 --disk 20 busybox
   {
     "resources": {
       "memory": 8,
       "cpus": 4,
       "nodes": 1,
       "disk": 20
     },
     "name": "test1",
     "tasks": [
       {
         "image": "busybox",
         "runtime": "singularity"
       }
     ]
   }

If the JSON output is saved in a file it be submitted to PROMINENCE using the ``run`` command, e.g.:

.. code-block:: console

   $ prominence run <filename.json>

The job description can also be a URL rather than a file, e.g.

.. code-block:: console

   $ prominence run <https://.../filename.json>


Multiple tasks in a single job
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

By default a job will run a single command inside a single container. However, it is possible to instead run multiple sequential tasks within a single job. Each task will have access to the same temporary storage, so transient files generated by one task are accessible by other tasks.

To run multiple tasks it is necessary to construct a JSON description of the job. For example, in this job there are two sequential tasks:

.. code-block:: console

   {
     "resources": {
       "memory": 1,
       "cpus": 1,
       "nodes": 1,
       "disk": 10
     },
     "name": "multiple-tasks",
     "tasks": [
       {
         "image": "centos:7",
         "runtime": "singularity",
         "cmd": "cat /etc/redhat-release"
       },
       {
         "image": "centos:8",
         "runtime": "singularity",
         "cmd": "cat /etc/redhat-release"
       }
     ]
   }

Use of JSON job descriptions is also necessary to run workflows, which we will come to next.

Policies
--------

The ``policies`` section of a job’s JSON description enables users to have more control of how jobs are managed and influence where they will be executed. The available options are:

* **maximumRetries**: maximum number of times a failing job will be retried
* **maximumTimeInQueue**: maximum time in minutes the job will remain in the queue
* **leaveInQueue**: when set to ``true`` (default is ``false``) completed, failed and deleted jobs will remain in the queue until the user specifies that they can be removed
* **placement**: allows users to specify requirements and preferences to influence where jobs will run

For example:

.. code-block:: console

   {
     "tasks": [
       {
         "image": "centos:7", 
         "runtime": "singularity",
         "cmd": date
       }
     ], 
     "name": "test", 
     "resources": {
       "nodes": 1, 
       "disk": 10, 
       "cpus": 1, 
       "memory": 1
     },
     "policies": {
       "maximumRetries": 4,
       "maximumTimeInQueue": 600,
       "leaveInQueue": true
     }
   }

Placement policies
^^^^^^^^^^^^^^^^^^

Job placement policies enable users to influence where jobs will be executed. This consists of ``requirements`` and ``preferences``.

To force a job to run at a particular site (``OpenStack-Alpha`` in this case):

.. code-block:: console

   "policies": {
     "placement": {
       "requirements": {
         "sites": [
           "OpenStack-Alpha"
         ]
       }
     }
   }

To force a job to run at any site in a particular region:

.. code-block:: console

   "policies": {
     "placement": {
       "requirements": {
         "regions": [
           "Alpha"
         ]
       }
     }
   }

