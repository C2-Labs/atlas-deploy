# ATLAS Deployment Instructions
Installation files and instructions for deploying the C2 Labs ATLAS platform

## Table of Contents
[Overview](#overview)<br>
[System Requirements](#system_reqs)<br>
[Database](#database)<br>
[Deployment Scenarios](#scenarios)<br>
[Kubernetes](k8s/README.md)<br>
[DNS, SSL, and Ingress](k8s/DNS-SSL-Ingress.md)<br>
[Local Docker/Docker Standalone](docker_standalone/README.md)<br>
<!-- [Docker Swarm](#docker_swarm)<br> -->


<a name="overview"/>

## Overview
ATLAS is deployed via a Docker image, built using the <a href="https://www.opencontainers.org/">Open Containers Intiative (OCI) Standard</a>. It is designed to be hosted in any environment that can run Docker containers. Configuration is done at runtime using environment variables. Below are instructions on how to deploy ATLAS in a Kubernetes environment and/or locally with Docker Desktop in a stand-alone configuration. The files necessary for these deployments are included in this GitHub repo. Some can be used as is while others will require minor edits for your specific environment, which are detailed below. Please [Contact Us](https://www.c2labs.com/contact-us) if you have any questions.

<a name="system_reqs"/>

## System Requirements
ATLAS can be dynamically scaled up or down to meet customer needs; from a small stand alone deployment on a single laptop to a multi-node Kubernetes install supporting thousands of users.  In addition, ATLAS can be deployed anywhere; to include on-premise, in a hyper-scale cloud provider (AWS or Azure in either commercial or government environments), or Software as a Service (SaaS) (Coming Soon).

The **_MINIMUM_** requirements to run ATLAS  are as follows:

- Single Docker container
- Microsoft SQL Server Database with 2 GB of storage
    - 5 ms (read), 10 ms (write)
    - 5 DTUs if running in Azure
    - Basic Tier if running in Azure
    - SQL Server Express can also be used for a local installation or low cost evaluations (it should not be used for Production deployments)
- 1 GB of disk capacity for persistent file storage

**_RECOMMENDED_** Requirements for Production:

- Multi-node Kubernetes cluster (NOTE: Docker Swarm with Docker Enterprise Edition (EE) is also supported but may require some initial professional services to configure)
- 3 pods/containers for high availability
- Microsoft SQL Server Database with 2 GB of storage
    - 5 ms (read), 10 ms (write)
    - 5 DTUs if running in Azure
    - Standard Tier or higher if running in Azure, aligned with customer Service Level Agreements (SLAs) for Production
- Daily database and storage backups 
- 10 GB of disk capacity for persistent file storage that can be expanded as needed

<a name="database"/>

## Database
A Microsoft SQL Server Database (or MS SQL Server Express) is required to run ATLAS. This database should be named ATLAS to minimize any confusion later if needing C2 Labs support. In order to connect to the database, you will need an ADO.NET (SQL Authentication) connection string, similar to the following:

`Server=tcp:{yourdatabase}.database.windows.net,1433;Initial Catalog=ATLAS;Persist Security Info=False;User ID={your_username};Password={your_password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;`

The connection string is stored as an environment variable within the container. It is most commonly (and securely) applied using Kubernetes Secrets or some other mechanism which are detailed below.

<a name="scenarios"/>

## Common Deployment Scenarios

There are several common ways to deploy ATLAS (NOTE: Additional deployment options may be available to meet unique customer requirements). Customer must support a way to run containers in their environment. For many of our customers, this technology is new to them and can be overwhelming, so have provided a common set of deployment options with supporting detailed instructions:

- [ATLAS Production with Kubernetes](k8s/README.md)
    - Quick Start
        - Kubernetes (K8s) is an open-source orchestration software for deploying, managing, and scaling containers; developed by Google.
        - K8s is the market leader and the best available option for an Enterprise environment. Customers will gain portability, availability, scalability, and extensibility with the market leading orchestration software in industry today.
    - PROS
        - Very portable, scalable, and extensible
        - Free to use
        - Can run in all major clouds today, including Azure, AWS, and Google Cloud
        - Can be dployed on-premise and on air-gapped networks leveraging a customer's existing gold image for Linux
        - Easily scale nodes to grow the cluster with real-time upgrades with no down time
        - Easily apply configurations to manage all components of the cluster
        - Expertise can be extended to the customer's DevOps pipeline to spin up additional containers/nodes/deployments (one cluster can host many applications over time)
    - CONS
        - Can be complex to set up your own K8s environment (NOTE: Very easy to get started with K8s in the cloud)
        - Requires support from personnel familiar with command line driven interfaces (i.e. Linux backgrounds)
        - May require security certification of K8s prior to use if the existing customer environment does not have a K8s cluster deployed
        - Uses a lot of laptop battery if running kubelet on Docker Desktop
- [ATLAS Stand-Alone with Local Docker](docker_standalone/README.md)
    - Quick Start
        - **_This is the preferred method to initially test and evaluate ATLAS_**
        - This approach can be used for stand-alone on a laptop or on a virtual machine.  It just requires Docker to be installed.
    - PROS
        - Very quick to set up
        - Runs on local computer/laptop or a single Windows/Linux Virtual Machine (VM)
        - Can run on Mac, Windows, and Linux
        - Can have ATLAS up and running to test it out in minutes with no external database needed
        - No cost to get started and can use existing resources to quickly evaluate the product
    - CONS
        - Not an enterprise solution
        - Not recommended for Production deployments (with the potential exception of small businesses and single user deployments)
        - Limited to the local resources on your computer or VM (CPU, Memory, Storage)
        - Persistence of container storage is harder (NOTE: Local DB container runs in memory and storage is not currently peristed)
        - Only designed for initial testing and evaluation
        - Must have admin privileges to install and configure on the local device/VM

## Post-Deployment Steps
1.  Login with ATLAS Admin account (break glass account)
2.  Change default password to a secure password (NOTE: Please put the password in a safe place as it cannot be retrieved if lost.)
3.  Configure the Tenant (coming soon)
4.  Provide user roles (coming soon)
5.  [Load Catalogues](catalogues\README.md)

<!-- <a name="docker_swarm"/>

## Docker Swarm -->