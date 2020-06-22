.. toctree::

Using the API
=============

Python
------

With the requests module
************************

The standard `requests module <https://requests.readthedocs.io/en/master/>`_ can be used to interact with the PROMINENCE service. A HTTP POST request is used to submit jobs while HTTP GET requests are used to check the status of jobs or retrieve information about jobs.

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

   if response.status_code == 201:
       if 'id' in response.json():
           print('Job submitted with id %d' % response.json()['id'])
   else:
       print('Job submission failed with http status code %d and error: %s' % (response.status_code, response.text))

