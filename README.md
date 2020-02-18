# atlas-deploy
Installation files and instructions for deploying C2 Labs ATLAS platform

## Table of Contents
[Overview](#overview)<br>
[System Requirements](#system_reqs)<br>
[Database](#database)<br>
[Kubernetes](#kubernetes)<br>
[Local Docker](#local_docker)<br>
[Docker Swarm](#docker_swarm)<br>


<a name="overview"/>

## Overview
ATLAS is a Docker image, built using the <a href="https://www.opencontainers.org/">Open Containers Intiative (OCI)</a>. It is designed to be run in any environment that can run Docker containers. Configuration is done at runtime using environment variables. Below, we will detail how to deploy this in a Kubernetes environment, locally with Docker Desktop or standalone Docker, and with Docker Swarm (coming soon). The files necessary for these deployments are included in this GitHub repo. Some can be used as is, and some will need minor edits for your specific environment, which we will detail below. Please reach out if you have any questions.

<a name="system_reqs"/>

## System Requirements
ATLAS can be scaled up or down to meet your needs. The **_MINIMUM_** requirements to run it are as follows:
- Single Docker container
- SQL Database with 2 GB of storage
    - 5 ms (read), 10 ms (write)
    - 5 DTUs in running in Azure
    - Basic Tier if running in Azure
- 1 GB of disk capacity for persistent file storage

**_RECOMMENDED_** Requirements for Production:
- 3 pods/containers
- SQL Database with 2 GB of storage
    - 5 ms (read), 10 ms (write)
    - 5 DTUs in running in Azure
    - Aligned with SLAs for Production
- 10 GB of disk capacity for persistent file storage that can be expanded as needed
<a name="database"/>

## Database
A SQL database is required to run ATLAS. In order to connect to the database, you will need an ADO.NET (SQL Authentication) connection string, similar to the following:

`Server=tcp:{yourdatabase}.database.windows.net,1433;Initial Catalog={YOURDB};Persist Security Info=False;User ID={your_username};Password={your_password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;`

This is stored as an environment variable within the container. You can apply this using Secrets or some other mechanism, which we will detail below.

<a name="kubernetes"/>

## Kubernetes
If you are using Kubernetes, you first need to configure your database, as detailed above. The files referenced below are all in the `k8s` directory of this repo. Your next steps are the following:

1. Create a namespace for ATLAS. You could deploy in your `default` namespace, but we recommend a namespace dedicated for ATLAS. In the examples and files, we will use `atlas-test`.
    - To create a namespace, run the command: `kubectl create namespace atlas-test`
    - You can verify it is created by running the command: `kubectl get namespace` and ensuring `atlas-test` is present
2. Configure persistent storage that can be presented to the container. If simply deploying a single replica/pod, this is straightforward and can be done in the following ways:
    - Azure: Azure Files, Azure Disks, or NFS mount
    - AWS: Elastic Block Store (EBS) or NFS mount using Elastic Files System (EFS)
    - Local: NFS mount to the container
3. This storage should be expandable, so you can add space, as necessary. We will walk through a couple examples below 
    - **STORAGE COMMANDS**
4. After you have the storage, configured you are ready to configure your ConfigMap. The ConfigMap has all the configurable attributes for ATLAS, which we will detail below. You need to **EDIT THIS** for your environment, as detailed below.
    - Overall Config
        - namespace: This is the namespace configured above
            - Default value: `atlas-test`
        - Domain: This is the URL of your deployment. Leave this as it is until your service is created, and then you will need to edit this and re-deploy
            - Default value: `'http://atlas.yourdomain.com'`
    - File Configuration
        - StoredFilesPath: This is the location where the persistent storage will be mounted. You should not need to change this value, unless you change the deployment
            - Default value: `'/atlas/files'`
        - PermittedFileExtensions: These are the file types that are allowed to be uploaded through the platform
            - Default value: `'.doc,.docx,.xls,.xlsx,.ppt,.pptx,.pdf,.avi,.mp4,.mov,.wmv,.msg,.txt,.rtf,.csv,.m4v,.png,.jpg,.gif,.jpeg,.bmp,.zip,.gz,.json,.html'`
    - FileSizeLimit: The file size limit per file in **bytes**. Please note the overall limit is 120 MB, even if you set this larger than that.
        - Default value: `"104857600"`
    - Email Configuration
        - Email: From/Reply-To address used when sending emails
            - Default value: `atlas_noreply@yourdomain.com`
        - EmailAddress: Email address used to login to the SMTP Server defined
            - Default value: `emailbot@yourdomain`
        - AdminEmail: System administrator email address
            - Default value: `admin@yourdomain.com`
        - EmailPort: Email port for **sending** email
            - Default value: `"587"`
        - SmtpServer: Address for SMTP server used for **sending** email
            - Default value: `smtp.yourdomain.com` or for Office365, use `smtp.office365.com`
5. Now deploy the ConfigMap:
    - `kubectl apply -f atlas-env.yaml`
6. There is a similar configuration for Secrets, where passwords and other items are stored
    - JWTSecretKey: This is your JWT Secret Key. This is generated from **TRAVIS TO ANSWER**
    - SQLConn: This is the SQL Connection string from above.
    - EmailPassword: This is the password to login to your SMTP server using the user defined by `EmailAddress` in the ConfigMap
7. Now deploy the Secret:
    - `kubectl apply -f atlas-secrets.yaml`

<a name="local_docker"/>

## Local Docker

<a name="docker_swarm"/>

## Docker Swarm