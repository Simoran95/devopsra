# DevOps-Assignment

## Task 1


### docker-compose.yml
This is the main build for the 3 web services. It takes precedence over any DockerFile/Image specific for each web service.

This is made up of 3 containers:
1. The Python API - Raapi.
2. The React frontend - Raweb.
3. The Nginx proxy - nginx

Starting with the raapi, one can see that the docker container uses port 5000 in the container, this is exposed to the machine port 5020. It is important for the nginx container that the network for the raapi is taken note of, therfore with the use of the networks option we stored the IP for raapi for future use.

The raweb more or less has the same structure in the docker-compose.yml as the raapi. It is built in directory ./sys-stats/; Uses a container port 3000 which is exposed to the machine in order to test connection to gui.

We use the dockerhub image for the nginx. This is then overlayed with the nginx.conf found in the ./data/nginx/nginx.conf in order to upload our required configuration. The nginx is operational on port 80, though we exposed port 3000 and port 5020 in order to comunicate with the raapi and raweb containers.

In order to run this file just use docker-compose up -d in order to set it as daemon. The --build option can be used to build the image, although this is done by default when running for the first time.

###The ./api/ directory

The api directory is made up of the python file app.py, the Dockerfile for the image of raapi, and the requirements.txt that is needed to run.

Issue encountered: I had trouble with the requirements.txt as the one provided had local files saved that were not available to me. On reflection, this could have been done by installing the concourse container and these files may have been available. As a work around, I created a new reqirements.txt file that worked well to deploy the raapi.

The Dockerfile uses the python:3.7-alpine image as a base image for the raapi. The WORKDIR sets the current directory of the container to /code. We then proceed to point the FLASK APP toward the app.py in order to run in host 172.19.0.3. The requirements.txt is copied from the control machine into the current directory of the docker container (i.e. /code) and is parsed by pip to install all the necessary applications to run the raapi. Finally the rest of the /api/ directory is copied into the docker container and the app.py file is run, thus deploying the container on localhost:5000 for the container. This is exposed to the control machine via port 5020, therefore to connect to the api one must use localhost:5020.

###The ./sys-stats/ directory

The raweb uses the sys-stats directory in order to deploy the React frontend. The node:14 image is used as base image for the raweb, while also setting the WORKDIR to /code on the container. The packge-lock.json and package.json files are then copied onto the container and an npm install is run. The npm start command is then applied in otder to deploy the development-server and reach it using localhost:3000.

Possible improvements: Additon of a .dockerignore file would have allowed the build to skip the node modules and would make the container faster to deploy and more lightweight.

###The ./data/ directory

The nginx uses the ./data/nginx/ directory as a vlume to replace the current nginx.conf, as explained earlier in the docker-compose.yml. The nginx.conf is used in order to connect the frontend and backend to nginx and possibly establish data sharing between them. 

Issue encountered: With nginx I succeeded in deploying both the raapi and raweb, though I struggled in order to pass the CPU usage and RAM usage to the frontend. A clearer investigation on nginx must be made.

A directory in ./data/site-content was used in order to test that nginx is operational with the use of an example index.html.


##Task 2

###The tar files

One can see the raapi.tar and raweb.tar files in the parent directory. These were the images for the raapi and raweb that were configured in task 1. These are needed in order to deploy the containers in AWS efficiently.

### The ./ansible/ directory

The ansible directory is used in order to run the ansible-playbook to launch an ec2 instance on AWS and proceed to deploy the docker-containers on the instance. The playbook may be run with ansible-playbook awsami.yml

## References Used

1. Used the following to install docker & docker-engine properly: [https://docs.docker.com/engine/install/ubuntu/](https://docs.docker.com/engine/install/ubuntu/)
2. Used the following to install docker-compose: [https://docs.docker.com/compose/install/](https://docs.docker.com/compose/install/)
3. Getting started with docker-compose: [https://docs.docker.com/compose/gettingstarted/](https://docs.docker.com/compose/gettingstarted/)
4. Dockerfile for NodeJS: [https://nodejs.org/de/docs/guides/nodejs-docker-webapp/](https://nodejs.org/de/docs/guides/nodejs-docker-webapp/)
5. Example Architecture: [https://faun.pub/deploy-apache-web-server-in-docker-container-in-aws-ec2-instances-using-ansible-playbook-36edc9ee13e4](https://faun.pub/deploy-apache-web-server-in-docker-container-in-aws-ec2-instances-using-ansible-playbook-36edc9ee13e4)
6. AWS-Ansible Communication: [https://docs.ansible.com/ansible/latest/scenario_guides/guide_aws.html](https://docs.ansible.com/ansible/latest/scenario_guides/guide_aws.html)

