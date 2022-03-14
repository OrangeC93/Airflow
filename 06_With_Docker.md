## Introduction
What's Docker

How to start Airflow in 5 min with Docker

How to run Airflow with the Local Executor on Docker

How to run Airflow with the Celery Executor on Docker?

## Quick Remember About Docker
Dukat allows you to install and run software regardless of the installed dependencies and the operating system used, which means with Dakkar, you are able to run your application on any operating system without worrying about the dependencies.

- Docker file: you're going to put all the instructions needed to install and run your application
- Read Docker image: think it as the application compiled
- Docker Container: Docker ru based on th image to obtai  docker container where your application is running inside that container

Dokcer container is like a VM running inside your operating system but without the operating system. So it's much lightweight than a VM.

Docker compose: define and run multi-container Docker applications. For example in Airflow, you have metadata, web server, and the schedule. So you have three differnt Docker containers that you have to orchestrate in a way in order to run. As you don't want to run everything into a single container, that doesn't make sense. Why? Because if the website ever fails, then you will have to restart not only the Web server, but also the scheduler and the database. That's why it is better to separate your containers according to the components you want to run.

And the good thing with Doker is that you don't have to execute a run for each service. Just with a simple command. You will be able to run all the containers corresponding to your application.

And one thing to remember is that all of those containers will share the same network. They will run inside the same network, and so each container will be able to communicate with the other.

## Running Airflow on Docker with the Celery Executor
Set up and run airflow locally with Docker
```console
mkdir airflow-local
cd airflow 
wget https://airflow.apache.org/docs/apache-airflow/stable/docker-compose.yaml
code .
```

Open docker-compose.yaml in the airflow-local
- Docker compose file describes your application with all the services needed to run it

Run airflow
```console
docker-compse -f docker-compose.yaml up -d
docker ps
```

localhost:8080

## Running Airflow on Docker with Local Executor
Modify configuration: in docker-compose.yaml
- AIRFLOW__CORE_EXECUTOR:LocalExecutor
- comment out AIRFLOW_CELERY__RESULT_BACKED
- comment out AIRFLOW__CELERY__BROKER_URL
- commont out redis (as wel as depens_on)
- comment our airflow-worker
- comment flower

then run
```console
docker-compose -f docker-compose.yaml up -d
docker ps
```

Verify in UI
- run bash operator task se if it works


## Recap
- Docker allows to run a Docker container which is an instance of a Docker image.
- A Docker image is defined from a Dockerfile specifying how your application should be installed
and run.
- Docker-compose is used to run a multi-containers application very easily and create a network
shared by all the containers within this application.
- You can modify the configuration file of Airflow by using environment variables. For example, if we
want to change the value of the parameter dags_folder under the core section, we just need to
create the environment variable AIRFLOW CORE DAGS FOLDER and set the value
- A FERNET_KEY is related to the crypto package from python in order to crypt the connections
saved into the metadatabase increasing the security
