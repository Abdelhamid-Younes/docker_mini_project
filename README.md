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
![](https://github.com/Abdelhamid-Younes/docker_mini_project/blob/main/images/Capture%20d'%C3%A9cran%202024-05-20%20123246.png?raw=true)

Now, it's time to run the container and test it:
![](https://github.com/Abdelhamid-Younes/docker_mini_project/blob/main/images/Capture%20d'%C3%A9cran%202024-05-20%20125807.png?raw=true)
To verify the API and make sure the it is correctly responding we can run the following curl command:
  ```bash
  curl -u toto:python -X GET http://172.17.0.2:5000/pozos/api/v1.0/get_student_ages
  ```  
  ![](https://github.com/Abdelhamid-Younes/docker_mini_project/blob/main/images/curl_result.png?raw=true)


## Infrastructure As Code

After testing the API image, we need now to put all together and deploy it, using docker-compose.

The docker-compose.yml file defines two services for deploying the POZOS "student_list" application:
1. **php_app**: This service represents the web application:
   - *depends_on*: Ensures the API service starts before the web app.
   - *ports*: Exposes port 80 to the host.
   - *networks*: Connects to a custom network student_list_network.
   - *volumes*: Binds the local website directory to /var/www/html in the container to serve the custom website.
   - *environment*: Sets environment variables for USERNAME and PASSWORD to enable the web app to authenticate with the API.
   - *image*: Uses the php:apache image.
2. **student_list_api**: This service represents the API:
   - *networks*: Connects to the student_list_network.
   - *volumes*: Mounts the local simple_api directory to /data/ in the container to provide the API with the necessary data.
   - *image*: Uses the custom-built student_list_api_img image.

Before runnig and creating the services described in docker-compose.yml file, we have to remove the api container created previously :
```bash
docker stop student_list_api
docker rm student_list_api
```
Now we launch the two services by running this command:
```bash
docker-compose up
```
![](https://github.com/Abdelhamid-Younes/docker_mini_project/blob/main/images/docker-compose%20up.png?raw=true)
Once the services are running, we open a web browser and try to access the web application on this URL: 
*http://localhost:80* (don't forget tu update the ip address if the docker container is running in a remote machine).
![](https://github.com/Abdelhamid-Younes/docker_mini_project/blob/main/images/via%20browser.png?raw=true)

## Docker Registry
To deploy a private Docker registry and store our built images, we can create a docker-compose file with two services, registry service and the UI service.
his approach simplifies the process by allowing us to define and manage both services together:
- **Services**:
  - `registry`: This service runs the Docker registry on port 5000 and allows for image deletion and necessary HTTP headers.
  - `registry_ui`: This service provides a web UI for managing the Docker registry, accessible on port 8090.

- **Networks**:
  - `pozos_network`: A custom bridge network allowing communication between the registry and its UI using container names.

- We used the following command to start the Docker compose stack:
  ```bash
docker-compose -f docker-compose.registry.yml up -d

  ```
- Then we have tagged our student_api image with the registry URL, as follows:
   ```bash
   docker tag student_list_api.img localhost:5000/student_list_api.img:v2
   ```
- Finnaly, we've pushed the new image to the private registry, like this:
   ```bash
   docker push localhost:5000/student_list_api.img:v2
    ```
To access to the registry interface, we open the web browser and type http://localhost:8090. (Update the ip address if the docker container is running in a remote machine).
![](https://github.com/Abdelhamid-Younes/docker_mini_project/blob/main/images/pozos_registry.png?raw=true)

## Conclusion
This project showed how Docker can improve the deployment, scalability, and availability of POZOS's "student_list" application. By using Docker containers and setting up a private registry, we made the infrastructure more efficient and easier to manage. 
