# atlas-deploy
Installation files and instructions for deploying C2 Labs ATLAS platform

## Table of Contents
[Overview](#overview)<br>
[System Requirements](#system_reqs)<br>
[Database](#database)<br>
[Kubernetes](#kubernetes)<br>
[Local Docker](#local_docker)<br>
<!-- [Docker Swarm](#docker_swarm)<br> -->


<a name="overview"/>

## Overview
ATLAS is a Docker image, built using the <a href="https://www.opencontainers.org/">Open Containers Intiative (OCI)</a>. It is designed to be run in any environment that can run Docker containers. Configuration is done at runtime using environment variables. Below, we will detail how to deploy this in a Kubernetes environment and locally with Docker Desktop or standalone Docker. The files necessary for these deployments are included in this GitHub repo. Some can be used as is, and some will need minor edits for your specific environment, which we will detail below. Please reach out if you have any questions.

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
    - Standard Tier or higher, aligned with SLAs for Production
- 10 GB of disk capacity for persistent file storage that can be expanded as needed
<a name="database"/>

## Database
A SQL database is required to run ATLAS. This database needs to be named ATLAS. In order to connect to the database, you will need an ADO.NET (SQL Authentication) connection string, similar to the following:

`Server=tcp:{yourdatabase}.database.windows.net,1433;Initial Catalog=ATLAS;Persist Security Info=False;User ID={your_username};Password={your_password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;`

This is stored as an environment variable within the container. You can apply this using Secrets or some other mechanism, which we will detail below.

<a name="kubernetes"/>

## Kubernetes
If you are using Kubernetes, you first need to configure your database, as detailed above. The files referenced below are all in the `k8s` directory of this repo. Your next steps are the following:

1. Create a namespace for ATLAS. You could deploy in your `default` namespace, but we recommend a namespace dedicated for ATLAS. In the examples and files, we will use `atlas`.
    - To create a namespace, run the command: `kubectl create namespace atlas`
    - You can verify it is created by running the command: `kubectl get namespace` and ensuring `atlas` is present
2. Configure persistent storage that can be presented to the container. If simply deploying a single replica/pod, this is straightforward and can be done in the following ways:
    - Azure: Azure Files, Azure Disks, or NFS mount
    - AWS: Elastic Block Store (EBS) or NFS mount using Elastic Files System (EFS)
    - Local: NFS mount to the container
3. This storage should be expandable, so you can add space, as necessary. We will walk through a couple examples below
    - These commands are very dependent on your environment. I will walk through scenarios for AWS, Azure, and with local NFS.
    - **AZURE**
        - Azure for Kubernetes provides two separate provisioners as part of the K8S StorageClass. Azure Files does not support access to the storage for multiple containers at a time, so it is **_HIGHLY RECOMMENDED_** that you use **Azure Disks**
        - To use Azure Disks, you can simply apply the StorageClass in the file `azure-files-sc.yaml`
            - `kubectl apply -f azure-files-sc.yaml`
            - As you look at this file, there are a couple things to note:
                - `provisioner: kubernetes.io/azure-file` - This specifies to use Azure Files, which is what we want
                - `allowVolumeExpansion: true` - This specifies that the volumes using this StorageClass can later be expanded, which we want to allow for additional storage later, if necessary.
                - `skuName: Standard_LRS` - This is the least expensive option from Azure and is sufficient for most customer's needs.
                    - More details here: https://docs.microsoft.com/en-us/azure/aks/azure-files-dynamic-pv
                    - Standard_LRS - standard locally redundant storage (LRS)
                    - Standard_GRS - standard geo-redundant storage (GRS)
                    - Standard_RAGRS - standard read-access geo-redundant storage (RA-GRS)
                    - Premium_LRS - premium locally redundant storage (LRS)
        - After the StorageClass is created, you simply need to create your Persistent Volume Claim
            - `kubectl apply -f atlas-azure-pvc.yaml`
            - As you look at this file, there are a couple things to note:
                - `ReadWriteMany` - This allows multiple pods to write to the same PVC.
                - `storageClassName: azure-file` - This is the name of the StorageClass configured above. You do not need to edit this, unless you changed it in the above steps.
                - `storage: 1Gi` - This is the initial amount of storage for your file store
            - Ensure the PVC has been created:
                - `kubectl get pvc -n atlas`
                - As a note, this will _automatically_ create the Persistent Volume.
    - **AWS**
        - Unfortunately AWS does not have a built-in storage class for Kubernetes for storage that allows multiple pods to Read/Write to the same storage. If you have a single pod, you can use Elastic Block Storage. For multiple pods, we need to use Elastic File Storage (EFS).
        - Create your EFS storage in the same VPC as your Kubernetes cluster. Only static provisioning of EFS is currently supported, so you must manually provision the storage prior to the next steps
            - When creating EFS filesystem, make sure it is accessible from Kubernetes cluster. This can be achieved by creating the filesystem inside the same VPC as Kubernetes cluster or using VPC peering. We would recommend having it in the same VPC as K8s.
            - Permissions and settings are detailed here: https://docs.aws.amazon.com/eks/latest/userguide/efs-csi.html
        - There are two ways to utilize the storage:
            - The Amazon Supported CSI driver to leverage a StorageClass
                - Amazon Announcement: https://aws.amazon.com/about-aws/whats-new/2019/09/amazon-eks-announces-beta-release-of-amazon-efs-csi-driver/
                - GitHub Repo: https://github.com/kubernetes-sigs/aws-efs-csi-driver
                - Deploy the CSI Driver
                    - `kubectl apply -k "github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/?ref=master"`
                - Deploy the StorageClass
                    - `kubectl apply -f aws-efs-csi-sc.yaml`
                - Deploy the Persistent Volume (PV)
                    - Edit `atlas-aws-csi-pv.yaml` and insert your `FileSystemID`.
                    - `kubectl apply -f atlas-aws-csi-pv.yaml`
                - Deploy the Persistent Volume Claim (PVC)
                    - `kubectl apply -f atlas-aws-csi-pvc.yaml`
            - Direct NFS
                - Deploy the NFS Persistent Volume (PV)
                    - Edit `atlas-aws-nfs-pv.yaml` and insert your `FileSystemID` and `Region` (copy from AWS).
                    - `kubectl apply -f atlas-aws-nfs-pv.yaml`
                - Deploy the Persistent Volume Claim (PVC)
                    - `kubectl apply -f atlas-aws-nfs-pvc.yaml`
    - **NFS**
        - TBD on local NFS, but it should be very similar to the Direct NFS commands for AWS, changing the server name in `atlas-aws-nfs-pv.yaml` to your local server
4. After you have the storage, configured you are ready to configure your ConfigMap. The ConfigMap has all the configurable attributes for ATLAS, which we will detail below. You need to **EDIT THIS** for your environment, as detailed below.
    - Overall Config
        - namespace: This is the namespace configured above
            - Default value: `atlas`
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
    - JWTSecretKey: This is your JWT Secret Key. This can be any random value, Base 64 encoded. If you have Node.js installed you can generate this with the following command:
        - `node -e "console.log(require('crypto').randomBytes(256).toString('base64'));"`
        - You can use other mechanisms to create this as well.
    - SQLConn: This is the SQL Connection string from above.
    - EmailPassword: This is the password to login to your SMTP server using the user defined by `EmailAddress` in the ConfigMap
7. Now deploy the Secret:
    - `kubectl apply -f atlas-secrets.yaml`
8. Now we are ready to deploy the ATLAS container:
    - `kubectl apply -f atlas-deploy.yaml`
    - In this file, you can scale the number of replicas for ATLAS. For testing, one is fine, but we recommend at least 3 for production environments.
        - `replicas: 1` OR
        - `replicas: 3`
    - Ensure the pod or pods are running with the command:
        - `kubectl get pods -n atlas`
9. Now that the pods are running, we need to expose the service:
    - `kubectl apply -f atlas-svc.yaml`
    - Ensure the service is running and the IP/URL is exposed:
        - `kubectl get svc -n atlas`
        - Copy the IP/URL from the `EXTERNAL-IP` column
10. Update the ATLAS deploy file with the IP/URL from _Step 9_.
    - In production, you should point a standard DNS record to this address
    - For testing, we can simply paste this value into the `Domain` value in the `atlas-deploy.yaml` file.
    - Reapply the deployment
        - `kubectl apply -f atlas-deploy.yaml`
11. ATLAS should now be running. Point your Web Browser to the IP/URL from _Step 9_ or the DNS entry created in _Step 10_


*** ADD INFO FOR HTTPS ***
*** ADD DNS INFO FOR AZURE ***
*** ADD DNS INFO FOR AWS ***

<a name="local_docker"/>

## Local Docker

<!-- <a name="docker_swarm"/>

## Docker Swarm -->