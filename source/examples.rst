.. toctree::

Examples
========

Jobs
----

LAMMPS on a single node
^^^^^^^^^^^^^^^^^^^^^^^

Here we run one of the `LAMMPS <https://lammps.sandia.gov/>`_ benchmark problems using Intel’s Singularity image. In this case we run an MPI job on a single node.

.. code-block:: console

   prominence create --cpus 2 \
                     --memory 2 \
                     --artifact https://lammps.sandia.gov/inputs/in.lj.txt \
                     --runtime singularity \
                     shub://intel/HPC-containers-from-Intel:lammps \
                     mpirun -np \$PROMINENCE_CPUS "/lammps/lmp_intel_cpu_intelmpi -in in.lj.txt"

This illustrates using `--artifact`` to download a file before executing the job. Rather than using a hardwired number of CPUs (e.g. 2 in this case) the environment variable `PROMINENCE_CPUS` is used instead. Resource requests (e.g. CPU and memory) refer to a minimum required, and as such it is possible that users may be given resources with more than what is requested. In this example, using the environment variable `PROMINENCE_CPUS` to specify how many MPI processes to run ensures that all CPUs are used.

Workflows
---------

1D parameter sweep
^^^^^^^^^^^^^^^^^^

.. code-block:: console

   {
     "name": "ps-workflow",
     "jobs": [
       {
         "resources": {
           "nodes": 1,
           "cpus": 1,
           "memory": 1,
           "disk": 10
         },
         "tasks": [
           {
             "image": "busybox",
             "runtime": "singularity",
             "cmd": "echo $frame"
           }
         ],
         "name": "render"
       }
     ],
     "factory": {
       "type": "parametricSweep",
       "parameters":[
         {
           "name": "frame",
           "start": 1,
           "end": 4,
           "step": 1
         }
      ]
     }
   }

Multiple steps
^^^^^^^^^^^^^^

Here we consider a simple workflow consisting of multiple sequential steps, e.g.

.. image:: /_static/multi-step-workflow.png

In this example ``job_A`` will run first, followed by ``job_B``, finally followed by ``job_C``. A basic JSON description is shown below:

.. code-block:: console

   {
     "name": "multi-step-workflow",
     "jobs": [
       {
         "resources": {
           "nodes": 1,
           "cpus": 1,
           "memory": 1,
           "disk": 10
         },
         "tasks": [
           {
             "image": "busybox",
             "runtime": "singularity",
             "cmd": "date"
           }
         ],
         "name": "job_A"
       },
       {
         "resources": {
           "nodes": 1,
           "cpus": 1,
           "memory": 1,
           "disk": 10
         },
         "tasks": [
           {
             "image": "busybox",
             "runtime": "singularity",
             "cmd": "date"
           }
         ],
         "name": "job_B"
       },
       {
         "resources": {
           "nodes": 1,
           "cpus": 1,
           "memory": 1,
           "disk": 10
         },
         "tasks": [
           {
             "image": "busybox",
             "runtime": "singularity",
             "cmd": "date"
           }
         ],
         "name": "job_C"
       }
     ],
     "dependencies": {
       "job_A": ["job_B"],
       "job_B": ["job_C"]
     }
   }

Scatter-gather
^^^^^^^^^^^^^^

He we consider the common type of workflow where a number of jobs can run in parallel. Once these jobs have completed another job will run. Typically this final step will take output generated from all the previous jobs. For example:

.. image:: /_static/scatter-gather-workflow.png

A basic JSON description is shown below:

.. code-block:: console

   {
     "name": "scatter-gather-workflow",
     "jobs": [
       {
         "resources": {
           "nodes": 1,
           "cpus": 1,
           "memory": 1,
           "disk": 10
         },
         "tasks": [
           {
             "image": "busybox",
             "runtime": "singularity",
             "cmd": "date"
           }
         ],
         "name": "job_A1"
       },
       {
         "resources": {
           "nodes": 1,
           "cpus": 1,
           "memory": 1,
           "disk": 10
         },
         "tasks": [
           {
             "image": "busybox",
             "runtime": "singularity",
             "cmd": "date"
           }
         ],
         "name": "job_A2"
       },
       {
         "resources": {
           "nodes": 1,
           "cpus": 1,
           "memory": 1,
           "disk": 10
         },
         "tasks": [
           {
             "image": "busybox",
             "runtime": "singularity",
             "cmd": "date"
           }
         ],
         "name": "job_A3"
       },
       {
         "resources": {
           "nodes": 1,
           "cpus": 1,
           "memory": 1,
           "disk": 10
         },
         "tasks": [
           {
             "image": "busybox",
             "runtime": "singularity",
             "cmd": "date"
           }
         ],
         "name": "job_B"
       }
     ],
     "dependencies": {
       "job_A1": ["job_B"],
       "job_A2": ["job_B"],
       "job_A3": ["job_B"]
     }
   }

