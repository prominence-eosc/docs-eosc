.. toctree::

Using the API
=============

PROMINENCE uses a RESTful API using data formatted in JSON. A HTTP POST request is used to submit jobs while HTTP GET requests are used to check the status of jobs or retrieve information about jobs. An access token must be provided with each request in the **Authorization** header:

.. code-block:: console

   Authorization: Bearer <token>

cURL
----

The **curl** command line tool can be used to submit jobs and check their status. Firstly, define an environment variable containing a valid token, e.g.

.. code-block:: console

   export ACCESS_TOKEN=<token>

Here ``<token>`` should be replaced with the actual access token obtained from either https://aai.egi.eu/fedcloud directly or from the refresh token obtained from that page.

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
   Date: Mon, 22 Jun 2020 11:43:46 GMT
   Content-Type: application/json
   Content-Length: 12
   Connection: keep-alive

   {"id":1099}

We see heere that the job id is **1099**. We can check the status of this job:

.. code-block:: console

   $ curl -i -H "Authorization: Bearer $ACCESS_TOKEN" https://prominence.fedcloud-tf.fedcloud.eu/api/v1/jobs/1099
   HTTP/1.1 200 OK
   Server: nginx/1.10.3 (Ubuntu)
   Date: Mon, 22 Jun 2020 11:44:45 GMT
   Content-Type: application/json
   Content-Length: 212
   Connection: keep-alive

   [{"events":{"createTime":1592826226},"id":1099,"name":"calculate-pi","resources":{"cpus":1,"disk":10,"memory":1,"nodes":1},"status":"pending","tasks":[{"image":"eoscprominence/testpi","runtime":"singularity"}]}]


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

