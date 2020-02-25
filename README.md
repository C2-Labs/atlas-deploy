# atlas-deploy
Installation files and instructions for deploying C2 Labs ATLAS platform

## Table of Contents
[Overview](#overview)<br>
[System Requirements](#system_reqs)<br>
[Database](#database)<br>
[Deployment Scenarios](#scenarios)<br>
[Kubernetes](k8s/README.md)<br>
[DNS, SSL, and Ingress](k8s/DNS-SSL-Ingress.md)<br>
[Local Docker](docker_standalone/README.md)<br>
<!-- [Docker Swarm](#docker_swarm)<br> -->


<a name="overview"/>

## Overview
ATLAS is a Docker image, built using the <a href="https://www.opencontainers.org/">Open Containers Intiative (OCI)</a>. It is designed to be run in any environment that can run Docker containers. Configuration is done at runtime using environment variables. Below, we will detail how to deploy this in a Kubernetes environment and locally with Docker Desktop or standalone Docker. The files necessary for these deployments are included in this GitHub repo. Some can be used as is, and some will need minor edits for your specific environment, which we will detail below. Please reach out if you have any questions.

<a name="system_reqs"/>

## System Requirements
ATLAS can be scaled up or down to meet your needs. The **_MINIMUM_** requirements to run it are as follows:
- Single Docker container
- Microsoft SQL Server Database with 2 GB of storage
    - 5 ms (read), 10 ms (write)
    - 5 DTUs in running in Azure
    - Basic Tier if running in Azure
    - SQL Server Express can also be used
- 1 GB of disk capacity for persistent file storage

**_RECOMMENDED_** Requirements for Production:
- 3 pods/containers
- Microsoft SQL Server Database with 2 GB of storage
    - 5 ms (read), 10 ms (write)
    - 5 DTUs in running in Azure
    - Standard Tier or higher, aligned with SLAs for Production
- 10 GB of disk capacity for persistent file storage that can be expanded as needed
<a name="database"/>

## Database
A Microsoft SQL Server Database (or MS SQL Server Express) is required to run ATLAS. This database needs to be named ATLAS. In order to connect to the database, you will need an ADO.NET (SQL Authentication) connection string, similar to the following:

`Server=tcp:{yourdatabase}.database.windows.net,1433;Initial Catalog=ATLAS;Persist Security Info=False;User ID={your_username};Password={your_password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;`

This is stored as an environment variable within the container. You can apply this using Secrets or some other mechanism, which we will detail below.

<a name="scenarios"/>

## Deployment Scenarios

There are several ways to deploy ATLAS. You need a way to run containers. We know all of this can be a bit overwhelming, so we will detail a couple of the options:
- [Kubernetes](k8s/README.md)
    - Quick Start
        - "Kubernetes is open-source orchestration software for deploying, managing, and scaling containers."
        - This is really the way to go for an Enterprise environment. You gain portability, scalability, and extensibility with the main orchestration software in industry today.
    - PROS
        - Very portable, scalable, and extensible
        - Can run in all major clouds today, including Azure, AWS, and Google Cloud
        - Easily scale nodes as you need to, as well as in place upgrades with no down time
        - Easily apply configurations to manage all pieces of the environment
        - Can easily be part of your DevOps pipeline to spin up additional containers/nodes/deployments
    - CONS
        - Can be complex to set up your own environment
        - Uses a lot of battery if running kubelet in Docker Desktop
- [Local Docker](docker_standalone/README.md)
    - Quick Start
        - This is the way to test and use containers for initial testing and trying things out
    - PROS
        - Very quick to set up
        - Runs on local computer/laptop
        - Can run on Mac, Windows, and Linux easily
        - Can have ATLAS up and running to test it out in minutes with no external database needed
    - CONS
        - Not an enterprise solution
        - Limited to the local resources on your computer (CPU, Memory, Storage)
        - Persistence of containers is harder
        - Only designed for initial testing and use



<!-- <a name="docker_swarm"/>

## Docker Swarm -->