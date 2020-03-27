.. toctree::

Examples
========

Jobs
----

Workflows
---------

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

