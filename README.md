# student-list 
This repo is a simple application to list student with a webserver (PHP) and API (Flask)

![project](https://user-images.githubusercontent.com/18481009/84582395-ba230b00-adeb-11ea-9453-22ed1be7e268.jpg)


------------


## Objectives

The objectives of this practice exam are to ensure that you are able to manage a docker infrastructure, so you will be evaluated about the following

### Themes:

- improve an existed application deployment process
- versioning your infrastructure release
- address best practice when implementing docker infrastructure
- Infrastructure As Code

## Context


*POZOS*  is an IT company located in France and develops software for High School.

The innovation department want to disrupt the existing infrastructure to ensure that

it can be scalable, easily deployed with a maximum of automation.

POZOS wants you to build a **POC** to show how docker can help you and how much this technology is efficient.

For this POC, POZOS will give you an application and want you to build a "decouple" infrastructure based on **Docker**.

Currently, the application is running on a single server with any scalability and any high availability.

When POZOS needs to deploy a new release, every time some goes wrong.

In conclusion, POZOS needs agility on its software farm.

## Infrastructure

For this POC, you will only use one single machine with a docker installed on it.

The build and the deployment will be made on this machine.

POZOS recommends you to use centos7.6 OS because it's the most used in the company.

Please also note that you are authorized to use a virtual machine base on Centos7.6 and not your physical machine.

The security is a very critical aspect of POZOS DSI so please do not disable the firewall or other security mechanisms otherwise please explain your reasons in your delivery.

## Application


The application that you will be working on is named "*student_list*", this application is very basic and enables POZOS to show the list of the student with their age.

student_list has two modules:

- the first module is a REST API (with basic authentication needed) who send the desire list of the student based on JSON file
- The second module is a web app written in HTML + PHP who enable end-user to get a list of students

Your work is to build one container for each module an make them interact with each other

Application source code can be found [here](https://github.com/diranetafen/student-list.git "here")

The files that you must provide (in your delivery) are ***Dockerfile*** and ***docker-compose.yml***  (currently both are empty)

Now it is time to explain you each file's role:

- docker-compose.yml: to launch the application (API and web app)
- Dockerfile: the file that will be used to build the API image (details will be given)
- student_age.json: contain student name with age on JSON format
- student_age.py: contains the source code of the API in python
- index.php: PHP  page where end-user will be connected to interact with the service to - list students with age. You need to update the following line before running the website container to make ***api_ip_or_name*** and ***port*** fit your deployment
   ` $url = 'http://<api_ip_or_name:port>/pozos/api/v1.0/get_student_ages';`



## Build and test :  First task answer
- this work was achieved in virtual machine docker (predefined in this bootcamps)
- after cloning the project , I proceed to write the dockerfile : [My dockerfile](https://github.com/mintoug/projet1_docker/blob/master/simple_api/Dockerfile"My dockerfile")
to build the image I use the command line
`docker build -t api-student:v1 .`
next I deploy the first container `docker run -d -v ${pwd}/student_age.json:/data/student_age.json -p 4000:5000 --name test-list api-student:v1`
to test I use `curl -u toto:python -X GET http://127.0.0.1:3000/pozos/api/v1.0/get_student_ages`
the result is ✨✨
![](https://github.com/mintoug/projet-1....jpg)


# Infrastructure As Code : Second task answer

Using docker-compose.
first I delete my previous created container

As asked The ***docker-compose.yml*** file will deploy two services :

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

my docker-compose.yml is 
`version: '2'
services:
  website:
    image: php:apache
    depends_on:
      - api
    ports:
      - "3001:80"
    volumes:
      - ./website:/var/www/html
    environment:
      - USER_NAME=toto
      - PASSWORD=python
    networks:
      - api_pozos
  api:
    image: api-student:v1
    ports:
      - "3000:5000"
    volumes:
      - ./simple_api/student_age.json:/data/student_age.json
    networks:
      - api_pozos
networks:
  api_pozos:`   

I Rrun your docker-compose.yml `docker-compose up `



## Docker Registry (4 points)

POZOS need me to deploy a private registry and store the built images

So you need to deploy :

- a docker [registry](https://docs.docker.com/registry/ "registry")
- a web [interface](https://hub.docker.com/r/joxit/docker-registry-ui/ "interface") to see the pushed image as a container
   ### to do that 
for the docker registry : i Run 
`docker run -d -p 5000:5000 --name registry-pozos --network student-list_api_pozos registry:2`

next the tag  `docker image tag api-student:v1 localhost:5000/api-student:v1`

next pushing the image to the container registry-pozos `docker push localhost:5000/api-student:v1`

for the UI I use 
`docker run -d --name registry_anissa_ui --network student-list_api_pozos -p 4002:80 -e REGISTRY_TITLE="registry pozos" -e REGISTRY_URL="http://registry-pozos:5000" joxit/docker-registry-ui:static`


