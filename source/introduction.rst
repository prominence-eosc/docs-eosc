.. toctree::

Introduction
============

Features include:

* **Flexible submission**: Submit jobs using a simple batch system style command line interface which can be installed anywhere. Or submit jobs from any Jupyter notebook. Alternatively, interact programmatically using a RESTful API.
* **Reliability and reproducibility**: All jobs are run in containers to ensure they will reliably run anywhere and are reproducible.
* **MPI ready**: Run multi-node Open MPI, Intel MPI or MPICH jobs in addition to single-node HTC jobs.
* **Workflows**: Construct complex workflows by specifying the dependencies between different jobs.
* **Multi-cloud native**: Go beyond a single cloud and leverage the resources and features available across many clouds.
* **Invisible infrastructure**: All infrastructure provisioning is handled completely automatically and is totally transparent to the user.

Workflows, jobs and tasks
-------------------------

A job in PROMINENCE consists of the following:

* Name
* Labels
* Input files
* Output files
* Required resources (e.g. CPU cores, memory, disk)
* One or more task definitions
* Policies (e.g. how many times should failing tasks should be retried)

Tasks execute sequentially within a job, and consist of the following:

* Container image
* Container runtime
* Command to run and optionally any arguments
* Environment variables
* Current working directory

A workflow consists of one or more jobs and optionally any dependencies between them. Jobs within a workflow can be executed sequentially, in parallel or combinations of both.

An example workflow, including how it is made up of jobs and tasks, is shown below:

.. image:: /_static/prominence-tasks-jobs-workflows.png

Job lifecycle
-------------

List of all possible job states:

* **idle**: the job is not yet running
* **deploying**: the infrastructure to run the job is being deployed.
* **running**: the job is runing.
* **deleted**: the job has been deleted by the user.
* **killed**: the job has been forcefully terminated, for example it had been running for too long.
* **completed**: the job has completed, however note that the user application's exit status may or may not be 0.
* **failed**: the job failed, for example the infrastructure could not be deployed successfully or the container image could not be pulled.

Note that jobs can transition from the deploying or idle states directly to the failed state in the event of problems.


