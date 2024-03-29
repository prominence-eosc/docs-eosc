.. toctree::

Containers
==========

In PROMINENCE all jobs are run in unprivileged containers using user-specified images. It is possible to use either the Singularity or udocker container runtimes, but of course standard Docker images can be used.

Why containers?
---------------

Every job in PROMINENCE needs to have a container image specified. Running jobs in containers ensure that the application will be able to run successfully without depending on any software on the host. This is important because PROMINENCE makes use of a variety of different cloud resources, managed by different people from different organizations, potentially around the world.

Accessing images
----------------

The image can be specified in the following ways:

* ``<user>/<repo>:<tag>`` (Docker Hub)
* ``<hostname>/<project-id>/<image>:<tag>`` (Google Container Registry)
* ``shub://<user>/<repo>:<tag>`` (Singularity Hub)
* URL of a tarball created by ``docker save``
* URL of a Singularity image
* path of a tarball created by ``docker save`` when remote storage is attached to jobs
* path of a Singularity image when remote storage is attached to jobs

Container registries other than Docker Hub may also work. It is possible to provide credentials to use for authenticating with a registry.

Under some conditions a container runtime will be selected automatically. This will only happen if there is only one runtime which will work for the specified image. For other cases, e.g. a Docker Hub image, Singularity is used as the default but optionally udocker can be forced by the user.

Images which will result in Singularity being selected:

* Singularity Hub image (begins with ``shub://``)
* URL for a Singularity image (filename ends in ``.sif`` or ``.simg``)

Images which will result in udocker being selected:

* URL for a Docker tarball (filename ends in ``.tar``)

Generally it is recommended to use Singularity. If it is essential to be able to write into the container's filesystem (e.g ``/opt`` or ``/var``) then udocker should be used.

Tips for creating containers
----------------------------
Some important tips for creating containers to be used with PROMINENCE:

* Do not put any software or required files in ``/root``, since containers are run as an unprivileged user.
* Do not put any software or required files in ``/home`` or ``/tmp``, as these directories in the container image will be replaced when the container is executed.
* Do not specify ``USER`` in your Dockerfile when creating the container image.
* The environment variable ``HOME`` will be set to a scratch directory (``/home/user``) accessible inside the container when the container is executed. For the case of multi-node MPI jobs this scratch directory is accessible across all nodes running the job.
* The environment variables ``TMP`` and ``TEMP`` are set to ``/tmp``. This directory is always local to the host, including for multi-node MPI jobs.
* Do not expect to be able to write inside the container's filesystem. Write any files into the default current working directory, or into the directories specified by the environment variables ``HOME``, ``TMP`` and ``TEMP``.
* The application should be able to be run from within any directory and access any required input or output files using relative paths.

MPI jobs
--------
Some are some additional requirements on the container images for MPI jobs:

* ``mpirun`` should be in available inside the container and in the ``PATH``
* The ``ssh`` command should be installed inside the container

There is no reason to set an entrypoint as it will not be used. A command (and any required arguments) must be specified.

A simple minimal starting point for a Dockerfile for a CentOS 7 container image for OpenMPI is:

.. code-block:: console

   FROM centos:7
   RUN yum -y install openssh-clients openmpi openmpi-devel

   ENV PATH            /usr/lib64/openmpi/bin:${PATH}
   ENV LD_LIBRARY_PATH /usr/lib64/openmpi/lib:${LD_LIBRARY_PATH}

and for MPICH:

.. code-block:: console

   FROM centos:7
   RUN yum -y install openssh-clients mpich mpich-devel

   ENV PATH            /usr/lib64/mpich/bin:${PATH}
   ENV LD_LIBRARY_PATH /usr/lib64/mpich/lib:${LD_LIBRARY_PATH}

To create a container using IntelMPI an Intel compiler licence is required to build the application. This application can then be copied into a container image with the Intel Parallel Studio Runtime installed. For example, see `here <https://software.intel.com/en-us/articles/installing-intel-parallel-studio-xe-runtime-2019-using-yum-repository>`_ for information on installing the free Intel runtime in a CentOS environment.

Registry authentication
-----------------------
If a registry requires authentication then ``imagePullCredential`` must be specified in the task, which defines a ``username`` and ``token``. For example:

.. code-block:: console

   {
     "resources": {
       "memory": 1,
       "cpus": 1,
       "nodes": 1,
       "disk": 10
     },
     "name": "gitlab-image",
     "tasks": [
       {
         "image": "registry.gitlab.com/mynamespace/myproject/image:rc1",
         "runtime": "singularity",
         "imagePullCredential": {
           "username": "username",
           "token": "VzIxo3sZ2yC6V5YeSBxR"
         }
       }
     ]
   }

The same method can be used for other private registries and Docker Hub. It is preferable that a token with read-only privileges is used rather than a password if at all possible.

