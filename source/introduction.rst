.. toctree::

Introduction
============

Getting access
--------------

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
* Working directory

A workflow consists of one or more jobs and optionally any dependencies between them. Jobs within a workflow can be executed sequentially, in parallel or combinations of both.

Job lifecycle
-------------

List of all possible job states:

* **idle**: the job is not yet running
* **deploying**: the infrastructure to run the job is being deployed.
* **running**: the job is runing.
* **deleted**: the job has been deleted by the user.
* **killed**: the job has been forcefully terminated, for example it had been running for too long.
* **completed**: the job has completed, however note that the exit status may or may not be 0.
* **failed**: the job failed, for example the infrastructure could not be deployed successfully or the container image could not be pulled.

Note that jobs can transition from the deploying or idle states directly to the failed state in the event of problems.

