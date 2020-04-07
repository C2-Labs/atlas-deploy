# ATLAS Deployment for Local Docker

This document provides installation files and instructions for deploying C2 Labs ATLAS in a Local Docker Environment (laptop, desktop, or Virtual Machine (VM)).

## Local Docker

C2 Labs created a convenient and straightforward way to run ATLAS locally, even without connecting to a remote database. This guide will allow a customer to spin up a local SQL container, connect to the database, initialize the database, and run ATLAS; all within their stand-alone environment (laptop, desktop, or VM).  This approach allows our customers to quickly and easily test and evaluate the product without external dependencies on infrastructure (which can take a long time to provision in Enterprise environments) and with no up-front costs for the evaluation.

In order to setup the test and evaluation environment, the customer should take the following steps: 

## PREREQUISITES

1. You need to have Docker installed; the free version of Docker Desktop is sufficient
2. You need to have access to Docker Hub
3. During Beta period, your Docker Hub ID needs access to the repository, `c2labs/atlas-c2internal`
4. Perform a `docker login` with that User ID (this can also force a download to make sure you have the right image)
    - If you want to test, you can manually pull the image:

        ```
        docker pull c2labs/atlas-c2internal:0.5.0-beta
        ```

### Prepare Configurations

1. Download all the files from the `docker_standalone` directory; or git clone the entire repository
2. Edit `db.env`
    - Set `SA_PASSWORD` to a secure value of your choosing. Avoid special characters, as these seem to cause some issues.  NOTE: this password is stored locally and is not available or retrievable by C2 Labs.
    - **NOTE** This password needs to meet the built-in complexity requirements of SQL Server:
        - At least 8 characters
        - 3 of the 4 following sets:
            - Uppercase letters
            - Lowercase letters
            - Base 10 digits
            - Symbols (avoid for now)
3. Edit `atlas.env`
    - StoredFilesPath: This is the location where the persistent storage will be mounted. You should not need to change this value, unless you change the mount point in your `docker-compose` file
        - Default value: `/atlas/files`
    - FileSizeLimit: The file size limit per file in **bytes**. Please note the overall limit is 120 MB, even if you set this variable larger than that. This also can later be set in the Admin Panel, but needs to be set initially here.
        - Default value: `104857600`
    - DB_SERVER: Name of the database server. **DO NOT CHANGE IF USING docker-compose**
        - Default value: `atlas-db`
    - DB_PORT: Port for the database. **DO NOT CHANGE IF USING docker-compose**
        - Default value: `1433`
    - JWTSecretKey: This is your JWT Secret Key. This can be any random value, Base 64 encoded. If you have Node.js installed you can generate the key with the following command:
        - `node -e "console.log(require('crypto').randomBytes(256).toString('base64'));"`
        - You can use any other mechanism to create this key.  However, it should be long and secure key as it will be changed infrequently (or never) as it will invalidate all open sessions after being deployed.
        - Default value: `Your&SuperSecret+JWTSecretToken+123442534234`
    - EncryptionKey: This is your Encryption Key for encrypted files and data in the database. This can be any random value, Base 64 encoded. If you have Node.js installed you can generate the key with the following command:
        - `node -e "console.log(require('crypto').randomBytes(256).toString('base64'));"`
        - You can use any other mechanism to create this key.  However, it should be long and secure key as it will be changed infrequently (or never) as it will invalidate all files and encrypted data in the database if you change it.
        - Default value: `Your$SuperSecret+Encryption!Key&122345q8`
    - SQLConn: This is the SQL Connection string from above.
        - If using container's DB, simply configure the password to match the one configured in db.env.  All other components should stay the same.  NOTE: If you wish to connect to an external database, you can provide a full connection string and avoid provisioning an in-memory database locally.  Also note that any data saved will be lost if the container running the database is stopped.

### Run ATLAS

1. If you want to stand up the database container and ATLAS, navigate into the directory where the `docker-compose.yml` file is located
    - Type:

        ```
        docker-compose up
        ```

        - This command will start the `atlas-db` container
        - Once the database is running, it will start the `atlas` container
        - The `atlas` container will wait for the database container to start and be listening on port 1433
    - To run this command in the background you can **_alternately_** run:

        ```
        docker-compose up -d
        ```

2. If you have a database you want to point to, edit the `atlas.env` with your database connection string and ensure your DB server is listening on port 1433
    - Simply run:

        ```
        docker run --env-file atlas.env -v atlasvolume:/atlas/files -p 81:80 c2labs.azurecr.io/atlas:dev
        ```

3. Following steps 5 or 6, ATLAS should now be running locally on a single container.
    - Point your browser to http://localhost:81
4. Login with the default credentials and **CHANGE THEM**
    - Username: `admin`
    - Password: `51mpl3Compliance$`
    - ATLAS will force you to change this upon first login
5. All configuration settings are now set within the application:
    - As an admin, click your Username->Setup
6. When you are done, you can clean up the containers with:

    ```
    docker-compose down
    ```

    - This will still leave the data in the database and on the volumes

### Remove Volumes
Please note, the Docker volumes are created and will remain, so your data will remain. If you want to **_REMOVE_** all the data or start fresh, run the following commands

1. Verify your volume names:

    ```
    docker volume ls
    ```

2. You should see output similar to(or exactly like) the following:

    ```
    DRIVER              VOLUME NAME
    local               docker_standalone_atlasvolume
    local               docker_standalone_sqlvolume
    ```

3. Remove both of the volumes, using the names from the command above:

    ```
    docker volume rm docker_standalone_atlasvolume docker_standalone_sqlvolume
    ```
