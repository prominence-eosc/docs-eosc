.. toctree::

Workflows
=========

Types of workflows
------------------

Groups of jobs
^^^^^^^^^^^^^^

In some situations it can be useful to be able to manage a group of jobs as a single entity. In order to submit a group of indendepent jobs as a workflow the first step is to write a JSON description of the workflow. This is just a list of the definitions of the individual jobs, which can be created easily using ``prominence create --dry-run``. The basic structure is:

.. code-block:: json
  
   {
       "name": "test-workflow-1",
       "jobs": [
        {...},
        {...}
      ]
   }

Directed acyclic graphs 
^^^^^^^^^^^^^^^^^^^^^^^

In order to submit a workflow the first step is to write a JSON description of the workflow. This is just a list of the definitions of the individual jobs, each of which can be created easily using ``prominence create --dry-run``, along with the dependencies between them. Each dependency defines a parent and its children. The basic structure is:

.. code-block:: json

   {
     "name": "test-workflow-1",
     "jobs": [
       {...},
       {...}
     ],
     "dependencies": {
       "parent_job": ["child_job_1", ...],
       ...
     }
   }

Each of the individual jobs must have defined names as these are used in order to define the dependencies. Dependencies between jobs are *abstract dependencies*, i.e. defined in terms of names of jobs. This is different to CWL, for example, where the dependencies arise due to input and output data or files required by each job.

It is important to note that the resources requirements for the individual jobs can be (and should be!) specified. This will mean that each step in a workflow will only use the resources it requires. Jobs within a single workflow can of course request very different resources, which makes it possible for workflows to have both HTC and HPC steps.

Job factories
^^^^^^^^^^^^^

Job factories allow many similar jobs to be created from a single template. The following types of job factories are available:

* **parametric sweep**: a set of jobs is created by sweeping one or more parameters through a range of values
* **zip**: a set of jobs is created from multiple lists, where the i-th job contains the i-th element from each list

In both cases a set of jobs is created by substituting a range of values into a template job. Substitutions can be made in the command to be executed or the values obtained using environment variables.

When a workflow using job factory is submitted to PROMINENCE individual jobs will automatically be created. The job names will be of the form ``<workflow name>/<job name>/<id>`` where ``<id>`` is an integer.

Parametric sweep
****************

In this case numeric values are generated from start and end points in addition to an increment provided by the user.

Here is an example fragment which would need to be included in a workflow description:

.. code-block:: json

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

Here we specify the factory to be of type ``parametericSweep``. The range of values used to create the jobs is defined in ``parameters``. The name of the parameter is given by ``name``. In this example the parameter frame is varied between the value ``start`` and at most ``end`` in increments of ``step``.

Jobs can obtain the value of the parameter through the use of substitutions or environment variables. If a job’s command was to include ``$frame`` or ``${frame}``, this would be substituted by the appropriate value. An environment variable ``PROMINENCE_PARAMETER_frame`` would also be available to the job containing this value.

Additional parameters can be included in order to carry out multi-dimensional parameter sweeps. 

If you wish to explicitly specify each value to be used, rather than specifying start and end values and a step, use a ``zip`` (described below) rather than a parametric sweep.

Zip
***

A set of jobs is created by substituting a range of values into a template job. The values to be used are specified in the form of lists. If multiple parameters are provided, the i-th job is provided with the i-th element from each list. The name comes from Python’s ``zip`` function.

Here’s an example fragment which would need to be included in a workflow description:

.. code-block:: json

   "factory": {
     "type": "zip",
     "parameters":[
       {
         "name": "start_value",
         "values": [
           0, 1, 2, 3
         ]
       },
       {
         "name": "end_value",
         "values": [
           8, 9, 10, 11
         ]
       }
     ]
   }

Here we specify the factory to be of type ``zip``. The range of values used to create the jobs is defined in ``parameters``. The name of each parameter is given by ``name`` and a list of values for each parameter is provided. In this example 4 jobs would be created, with:

* start_value = 0, end_value = 8
* start_value = 1, end_value = 9
* start_value = 2, end_value = 10
* start_value = 3, end_value = 11

Failures and retries
--------------------

By default the number of retries is zero, which means that if a job fails the workflow will fail. Any jobs which depend on a failed job will not be attempted. If the number of retries is set to one or more, if an individual job fails (i.e. exit code is not 0) it will be retried up to the specified number of times. To set a maximum number of retries, include ``maximumRetries`` in the workflow definition, e.g.

.. code-block:: json

   "policies": {
     "maximumRetries": 2
   }

If any jobs in a workflow fail, the workflow will stop running once there are no more jobs which can be run. Any dependencies of jobs which fail will not be executed.

The ``rerun`` command can be used to re-run any failed jobs in a workflow, for example:

.. code-block:: console

   prominence rerun <workflow id>

This will retry jobs which had previously failed and execute any dependencies which were not run previously due to the failed jobs. This command can be used multiple times if necessary.

