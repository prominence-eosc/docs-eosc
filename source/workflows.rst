.. toctree::

Workflows
=========

Groups of jobs
--------------

In some situations it can be useful to be able to manage a group of jobs as a single entity (a workflow). In order to submit a group of indendepent jobs as a workflow the first step is to write a JSON description of the workflow. This is just a list of the definitions of the individual jobs, which can be created easily using ``prominence create --dry-run``. The basic structure is:

.. code-block:: console
  
   {
       "name": "test-workflow-1",
       "jobs": [
        {...},
        {...}
      ]
   }

Directed acyclic graphs 
-----------------------

In order to submit a workflow the first step is to write a JSON description of the workflow. This is just a list of the definitions of the individual jobs, each of which can be created easily using ``prominence create --dry-run``, along with the dependencies between them. Each dependency defines a parent and its children. The basic structure is:

.. code-block:: console

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

Each of the individual jobs must have defined names as these are used in order to define the dependencies. Dependencies between jobs are *abstract dependencies*, i.e. defined in terms of names of jobs.

It is important to note that the resources requirements for the individual jobs can be (and should be!) specified. This will mean that each step in a workflow will only use the resources it requires. Jobs within a single workflow can of course request very different resources, which makes it possible for workflows to have both HTC and HPC steps.

By default the number of retries is zero, which means that if a job fails the workflow will fail. Any jobs which depend on a failed job will not be attempted. If the number of retries is set to one or more, if an individual job fails (i.e. exit code is not 0) it will be retried up to the specified number of times. To set a maximum number of retries, include ``maximumRetries`` in the workflow definition, e.g.

.. code-block:: console

   "policies": {
     "maximumRetries": 2
   }

Job factories
-------------

The following types of job factories are available:
* **parametric sweep**: a set of jobs is created by sweeping one or more parameters through a range of values
* **zip**: a set of jobs is created from multiple lists, where the i-th job contains the i-th element from each list

In both cases a set of jobs is created by substituting a range of values into a template job. Substitutions can be made in the command to be executed or the values obtained using environment variables.

When a workflow using job factory is submitted to PROMINENCE individual jobs will automatically be created. The job names will be of the form ``<workflow name>/<job name>/<id>`` where ``<id>`` is an integer.

Parametric sweep
****************

Zip
***
