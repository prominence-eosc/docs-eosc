.. toctree::

Using the API
=============

PROMINENCE uses a RESTful API with data formatted in JSON. A POST request is used for job/workflow submission while GET requests are used to check the status of or retrieve information about jobs/workflows. A DELETE request is used to delete jobs/workflows.

An access token must be provided with each request in the ``Authorization`` header:

.. code-block:: console

   Authorization: Bearer <token>

where ``<token>`` should be replaced with the actual access token.

The base URL is https://eosc.prominence.cloud/api/v1, while the specific endpoints to use are as follows:

* For jobs: https://eosc.prominence.cloud/api/v1/jobs
* For workflows: https://eosc.prominence.cloud/api/v1/workflows

To begin with we will go through some basic examples using cURL in order to demonstrate common API requests and their responses, and then look at using the API with Python.

cURL
----

The ``curl`` command line tool can be used to submit jobs and check their status. Firstly, for simplicity we define an environment variable containing a valid token, e.g.

.. code-block:: console

   export ACCESS_TOKEN=<token>

where ``<token>`` should be replaced with the access token as mentioned previously.

Submitting a job
^^^^^^^^^^^^^^^^

Create a file containing the JSON description of a job. In this example we use a file ``testpi.json`` containing the following:

.. code-block:: json

   {
     "resources": {
       "memory": 1,
       "cpus": 1,
       "nodes": 1,
       "disk": 10
     },
     "name": "calculate-pi",
     "tasks": [
       {
         "image": "eoscprominence/testpi",
         "runtime": "singularity"
       }
     ]
   }

This job can be submitted by running the following command:

.. code-block:: console

   curl -i -X POST -H "Authorization: Bearer $ACCESS_TOKEN" \
        -H "Content-Type: application/json" \
        -d@testpi.json \
        https://eosc.prominence.cloud/api/v1/jobs

If the submission was successful, this should return a response like the following:

.. code-block:: console

   HTTP/1.1 201 CREATED
   Server: nginx/1.10.3 (Ubuntu)
   Date: Fri, 05 Feb 2021 17:39:59 GMT
   Content-Type: application/json
   Content-Length: 12
   Connection: keep-alive

   {"id":1699}

We see here that the job id is ``1699``.

Checking the satus of a job
^^^^^^^^^^^^^^^^^^^^^^^^^^^

We can check the status of this job with a simple GET request, here using ``jq`` to display the JSON in a pretty way:

.. code-block:: console

   curl -s -H "Authorization: Bearer $ACCESS_TOKEN" https://eosc.prominence.cloud/api/v1/jobs/1169 | jq .

will return:

.. code-block:: json

   [
     {
       "events": {
         "createTime": 1612546799
       },
       "id": 1169,
       "name": "calculate-pi",
       "resources": {
         "cpus": 1,
         "disk": 10,
         "memory": 1,
         "nodes": 1
       },
       "status": "pending",
       "tasks": [
         {
           "image": "eoscprominence/testpi",
           "runtime": "singularity"
         }
       ]
     }
   ]

This request returns all information about the specified job. A completed job will have more information in the JSON response, for example:

.. code-block:: console

   curl -s -H "Authorization: Bearer $ACCESS_TOKEN" "https://eosc.prominence.cloud/api/v1/jobs/1181" | jq .

will return:

.. code-block:: json

   [
     {   
       "events": {
         "createTime": 1612684304,
         "endTime": 1612684626,
         "startTime": 1612684595
       },
       "execution": {
         "maxMemoryUsageKB": 242814,
         "site": "OpenStack-UNIV-LILLE",
         "tasks": [
           {
             "cpuTimeUsage": 0.5480000000000012,
             "exitCode": 0,
             "imagePullStatus": "completed",
             "imagePullTime": 26.14915418624878,
             "maxResidentSetSizeKB": 63044,
             "retries": 0,
             "wallTimeUsage": 0.6737308502197266
           }
         ]
       },
       "id": 1181,
       "name": "calculate-pi",
       "resources": {
         "cpus": 1,
         "disk": 10,
         "memory": 1,
         "nodes": 1
       },
       "status": "completed",
       "tasks": [
         {
           "image": "eoscprominence/testpi",
           "runtime": "singularity"
         }
       ]
     }
   ]

Listing all jobs
^^^^^^^^^^^^^^^^

Alternatively we can list all currently active jobs, i.e. jobs which have not yet completed:

.. code-block:: console

   curl -s -H "Authorization: Bearer $ACCESS_TOKEN" https://eosc.prominence.cloud/api/v1/jobs | jq .

will return:

.. code-block:: json

   [
     {
       "events": {
         "createTime": 1612546799
       },
       "id": 1169,
       "name": "calculate-pi",
       "status": "pending",
       "tasks": [
         {
           "image": "eoscprominence/testpi",
           "runtime": "singularity"
         }
       ]
     }
   ]

Listing completed jobs
^^^^^^^^^^^^^^^^^^^^^^

In order to list completed jobs (e.g. finished successfully, deleted, failed, or killed) add the query parameter ``completed`` with value ``true``, for example:

.. code-block:: console

   curl -s -H "Authorization: Bearer $ACCESS_TOKEN" "https://eosc.prominence.cloud/api/v1/jobs?completed=true" | jq .

will return:

.. code-block:: json

   [
     {
       "events": {
         "createTime": 1612682844,
         "endTime": 1612683091
       },
       "id": 1179,
       "name": "calculate-pi",
       "status": "failed",
       "statusReason": "No matching resources currently available",
       "tasks": [
         {
           "image": "eoscprominence/testpi",
           "runtime": "singularity"
         }
       ]
     }
   ]

By default the last completed job will be shown. An additional query parameter ``num`` can be added specifying the number of jobs to display.

Getting the standard output or error from jobs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

The following example returns the standard output from a job, in this case with id ``1181``:

.. code-block:: console

   curl -H "Authorization: Bearer $ACCESS_TOKEN" https://eosc.prominence.cloud/api/v1/jobs/1181/stdout

To get the standard error replace ``stdout`` above with ``stderr``.

Note that the standard output and error can be obtained both once a job has completed and while it is running, so it is possible to watch what a job is doing in real time, no matter where in the world it is running.

Deleting jobs
^^^^^^^^^^^^^

Jobs can easily be deleted using the REST API, for example:

.. code-block:: console

   curl -i -H "Authorization: Bearer $ACCESS_TOKEN" -X DELETE https://eosc.prominence.cloud/api/v1/jobs/1169
   HTTP/1.1 200 OK
   Server: nginx/1.10.3 (Ubuntu)
   Date: Fri, 05 Feb 2021 18:01:28 GMT
   Content-Type: application/json
   Content-Length: 3
   Connection: keep-alive

   {}

Python
------

With the requests module
************************

The standard `requests module <https://requests.readthedocs.io/en/master/>`_ can be used to interact with the PROMINENCE service.

Below is a complete simple example which submits a basic job. A JSON description of the job is constructed and a HTTP POST request is used to submit the job to the PROMINENCE service. In order to authenticate with the PROMINENCE server the access token is read from a file (the same file used by the PROMINENCE CLI) and the appropriate header is constructed and included in the HTTP request.

.. code-block:: console

   import json
   import os
   import requests

   # Define a job
   job = {
       "resources": {
           "memory": 1,
           "cpus": 1,
           "nodes": 1,
           "disk": 10
       },
       "name": "calculate-pi",
       "tasks": [
           {
               "image": "eoscprominence/testpi",
               "runtime": "singularity"
           }
       ]
   }

   # Read the access token
   if os.path.isfile(os.path.expanduser('~/.prominence/token')):
       with open(os.path.expanduser('~/.prominence/token')) as json_data:
           data = json.load(json_data)

       if 'access_token' in data:
           token = data['access_token']
       else:
           print('The saved token file does not contain access_token')
           exit(1)

   # Create the header including the auth token
   headers = {'Authorization':'Bearer %s' % token}

   # Submit the job
   response = requests.post('https://eosc.prominence.cloud/api/v1/jobs', json=job, headers=headers)

   # Check if the submission was successful and get the job id
   if response.status_code == 201:
       if 'id' in response.json():
           print('Job submitted with id %d' % response.json()['id'])
   else:
       print('Job submission failed with http status code %d and error: %s' % (response.status_code, response.text))

