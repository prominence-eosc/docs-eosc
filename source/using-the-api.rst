.. toctree::

Using the API
=============

PROMINENCE uses a RESTful API using data formatted in JSON. A HTTP POST request is used to submit jobs while HTTP GET requests are used to check the status of jobs or retrieve information about jobs. A HTTP DELETE request is used to delete jobs.

An access token must be provided with each request in the **Authorization** header:

.. code-block:: console

   Authorization: Bearer <token>

where ``<token>`` should be replaced with the actual access token obtained from either https://aai.egi.eu/fedcloud directly or from the refresh token obtained from that page.

The URLs to use are as follows:

* For jobs: https://prominence.fedcloud-tf.fedcloud.eu/api/v1/jobs
* For workflows: https://prominence.fedcloud-tf.fedcloud.eu/api/v1/workflows

cURL
----

The **curl** command line tool can be used to submit jobs and check their status. Firstly, for simplicity we define an environment variable containing a valid token, e.g.

.. code-block:: console

   export ACCESS_TOKEN=<token>

where ``<token>`` should be replaced with the access token as mentioned previously.

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
        https://prominence.fedcloud-tf.fedcloud.eu/api/v1/jobs

If the submission was successful, this should return something like the following:

.. code-block:: console

   HTTP/1.1 201 CREATED
   Server: nginx/1.10.3 (Ubuntu)
   Date: Fri, 05 Feb 2021 17:39:59 GMT
   Content-Type: application/json
   Content-Length: 12
   Connection: keep-alive

   {"id":1699}

We see here that the job id is **1699**. We can check the status of this job with a simple GET request, here using **jq** to display the JSON in a pretty way:

.. code-block:: console

   $ curl -s -H "Authorization: Bearer $ACCESS_TOKEN" https://prominence.fedcloud-tf.fedcloud.eu/api/v1/jobs/1169 | jq .
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

Alternatively we can list all currently active jobs, i.e. jobs which have not yet completed:

.. code-block:: console

   $ curl -s -H "Authorization: Bearer $ACCESS_TOKEN" https://prominence.fedcloud-tf.fedcloud.eu/api/v1/jobs | jq .
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

Jobs can easily be deleted using the REST API, for example:

.. code-block:: console

   $ curl -i -H "Authorization: Bearer $ACCESS_TOKEN" -X DELETE https://prominence.fedcloud-tf.fedcloud.eu/api/v1/jobs/1169
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
   response = requests.post('https://prominence.fedcloud-tf.fedcloud.eu/api/v1/jobs', json=job, headers=headers)

   # Check if the submission was successful and get the job id
   if response.status_code == 201:
       if 'id' in response.json():
           print('Job submitted with id %d' % response.json()['id'])
   else:
       print('Job submission failed with http status code %d and error: %s' % (response.status_code, response.text))

