# student-list 
This repo is a simple application to list student with a webserver (PHP) and API (Flask)

![project](https://user-images.githubusercontent.com/18481009/84582395-ba230b00-adeb-11ea-9453-22ed1be7e268.jpg)


------------


## Introduction
This project aims to demonstrate how Docker can enhance the deployment process, improve scalability, and ensure high availability for POZOS, an IT company located in France that develops software for high schools.
The goal of this practice exam is to assess my ability to manage a Docker infrastructure, focusing on the following areas:
- improve an existed application deployment process
- versioning your infrastructure release
- address best practice when implementing docker infrastructure
- Infrastructure As Code

## Context
*POZOS* is an IT company in France that develops software for high schools. The innovation department aims to revamp the existing infrastructure to ensure scalability, easy deployment, and maximum automation. They want to build a Proof of Concept (**POC**) using **Docker** to demonstrate its efficiency. Currently, the application runs on a single server with no scalability or high availability, often encountering issues during new releases. POZOS needs a more agile software infrastructure.


## Tasks
Our tasks for this project were to:

- Build a separate Docker container for each module of the application.
- Configure the containers to interact seamlessly with each other.
- Set up a private Docker registry for storing and managing the built images.

## Application
The application, named "student_list," is designed to display a list of students with their ages. It has two modules:
- A REST API with basic authentication that serves the student list from a JSON file.
- A web app written in HTML and PHP that allows end-users to view the students list.
For this project, i need to provide a **Dockerfile** and a **docker-compose.yml** file.

Let's now go over the role of each file:
- docker-compose.yml: to launch the application (API and web app)
- Dockerfile: the file that will be used to build the API image (details will be given)
- requirements.txt: contains all the packages to be installed to run the application
- student_age.json: contain student name with age on JSON format
- student_age.py: contains the source code of the API in python
- index.php: PHP  page where end-user will be connected to interact with the service to list students with their age.

## Building and testing images
- According to the dockerfile of api image we have selected *python:3.8-buster* that uses the official Python 3.8 image based on Debian Buster.
- Then we have copied the *student_age.py* and *requirements.txt* files from the local machine into the root directory of the image "/".
- Next we have have updated and installed system and python dependencies.
```bash
apt-get update -y && apt-get install python-dev python3-dev libsasl2-dev python-dev libldap2-dev libssl-dev -y
pip install --upgrade pip && pip3 install -r /requirements.txt
```
- Then we indicated on which port the container will be listen at runtime (5000).
- Finally, we have defined the entry point for the container. When the container starts, it will run the command
```bash
python3 ./student_age.py
```
To build the Docker image from this Dockerfile, you would navigate to the directory containing the Dockerfile *simple_api* and run the following command:
```bash
docker build . -t student_list_api.img
docker images
```





Copy the requirements.txt file into the container in the root "/" directory to install the packages needed to start up our application

to launch the installation, use this command

```bash
pip3 install -r /requirements.txt
```
- Persistent data (volume)

Create data folder at the root "/" where data will be stored and declare it as a volume

You will use this folder to mount student list

- API Port

To interact with this API expose 5000 port

- CMD

When container start, it must run the student_age.py (copied at step 4), so it should be something like
```bash 
CMD [ "python3", "./student_age.py" ]
```

Build your image and try to run it (don't forget to mount *student_age.json* file at */data/student_age.json* in the container), check logs and verify that the container is listening and is  ready to answer

Run this command to make sure that the API correctly responding (take a screenshot for delivery purpose)
NB: Start your container using this specific port to reach it
Port: 5000
```bash 
curl -u toto:python -X GET http://<host IP>:<API exposed port>/pozos/api/v1.0/get_student_ages
```

**Congratulation! Now you are ready for the next step (docker-compose.yml)**

## Infrastructure As Code (5 points)

After testing your API image, you need to put all together and deploy it, using docker-compose.

The ***docker-compose.yml*** file will deploy two services :

- website: the end-user interface with the following characteristics
   - image: php:apache
   - environment: you will provide the USERNAME and PASSWORD to enable the web app to access the API through authentication
   - volumes: to avoid php:apache image run with the default website, we will bind the website given by POZOS to use. You must have something like
`./website:/var/www/html`
   - depend on: you need to make sure that the API will start first before the website
   - port: do not forget to expose the port
- API: the image builded before should be used with the following specification
   - image: the name of the image builded previously
   - volumes: You will mount student_age.json file in /data/student_age.json
   - port: don't forget to expose the port
   - networks: don't forget to add specific network for your project

Delete your previous created container

Run your docker-compose.yml

Finally, reach your website and click on the bouton "List Student"

**If the list of the student appears, you are successfully dockerizing the POZOS application! Congratulation (make a screenshot)**

## Docker Registry (4 points)

POZOS need you to deploy a private registry and store the built images

So you need to deploy :

- a docker [registry](https://docs.docker.com/registry/ "registry")
- a web [interface](https://hub.docker.com/r/joxit/docker-registry-ui/ "interface") to see the pushed image as a container

Or you can use [Portus](http://port.us.org/ "Portus") to run both

Don't forget to push your image on your private registry and show them in your delivery.

## Delivery (4 points)

Your delivery must be link of your repository with your name that contain:
- A README file with your screenshots and explanations.
- Configuration files used to realize the graded exercise (docker-compose.yml and Dockerfile).

Your delivery will be evaluated on:

- Explanations quality
- Screenshots quality (relevance, visibility)
- Presentation quality
- The structure of your github repository

Send your delivery at ***eazytrainingfr@gmail.com*** and we will provide you the link of the solution.

![good luck](https://user-images.githubusercontent.com/18481009/84582398-cad38100-adeb-11ea-95e3-2a9d4c0d5437.gif)
