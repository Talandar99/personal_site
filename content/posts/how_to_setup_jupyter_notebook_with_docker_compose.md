+++
title = 'Jupyter + Spark as Docker Compose'
date = 2025-05-18
draft = false
+++
## Prerequisite
Make sure you have docker compose installed on your machine. 
I assume you have docker compose V2 but V1 should also do the job.
We both know you are just as lazy as I am. So i will start with most important stuff:
## How to run 
### 1. Create compose.yml file and add content above
```bash
vim compose.yml
```
compose.yml :
```yaml
services:
  spark-scala-notebook:
    build:
      context: .
      dockerfile: Dockerfile
      args:
        NB_UID: 1000
        NB_GID: 1000
    image: spark-scala-notebook:latest
    ports:
      - "8888:8888"
    environment:
      - JUPYTER_TOKEN=
    volumes:
      - ./work:/home/jovyan/work
    user: root
    command: >
      bash -c "
        chown -R jovyan:users /home/jovyan/work &&
        start-notebook.sh --NotebookApp.token=''
      "
    networks:
      - spark_network

networks:
  spark_network:
    driver: bridge
```
### 2. Compose it
execute the following command in the same file you created your compose.yml file
```bash
docker compose up
```
### 3. Start using Jupyter
notebook will be available at localhost:8888
[link](http://localhost:8888/)

---
## About compose file
### Good to know:
After work you can compose down this container with this command:
```bash
docker compose down 
```
All your notebooks are stored in `work` directory. This directory is created when you run docker compose for the first time
### Explaining weird stuff in compose.yml
`NB_UID: 1000` — user ID for the notebook user.\
`NB_GID: 1000` — group ID for the notebook user.\
`command:` keyword overrides the container’s default command with a shell script:
```yaml
    command: >
      bash -c "
        chown -R jovyan:users /home/jovyan/work &&
        start-notebook.sh --NotebookApp.token=''
      "
```
First, it ensures that the `work` directory is owned by the jovyan user (used inside the notebook). Then, it starts the Jupyter notebook server with no token (no auth).



