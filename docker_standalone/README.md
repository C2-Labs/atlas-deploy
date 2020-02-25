# atlas-deploy for Local Docker
Installation files and instructions for deploying C2 Labs ATLAS in a Local Docker Environment


## Local Docker
We have created a straightforward way to run ATLAS locally, even without a database. It will spin up a local SQL container, connect to the database, initialize the database, and run ATLAS. In order to do this: 

**_PREREQUISITE: You need to have Docker installed; Docker Desktop works great_**

1. Download all the files from the `docker_standalone` directory
2. Edit `db.env`
    - Set `SA_PASSWORD` to a value of your choosing. Avoid special characters, as these seem to cause some issues.
3. Create a local directory to map as a persistent volume for ATLAS. This is where ATLAS will store and files
    - Edit `docker-compose.yml`
    - Change `/tmp` under `volumes` to the directory you created or you can leave it at `/tmp` if that exists on your host.
4. Edit `atlas.env`
    - Domain: This is the URL of your deployment. Leave this as it is until your service is created, and then you will need to edit this and re-deploy
        - Default value: `'http://atlas.yourdomain.com'`
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
    - DB_SERVER: Name of the database server. **DO NOT CHANGE IF USING docker-compose**
        - Default value: `atlas-db`
    - DB_PORT: Port for the database. **DO NOT CHANGE IF USING docker-compose**
        - Default value: `1433`
    - JWTSecretKey: This is your JWT Secret Key. This can be any random value, Base 64 encoded. If you have Node.js installed you can generate this with the following command:
        - `node -e "console.log(require('crypto').randomBytes(256).toString('base64'));"`
        - You can use other mechanisms to create this as well.
        - Default value: `Your&SuperSecret+JWTSecretToken+123442534234`
    - SQLConn: This is the SQL Connection string from above.
        - If using container's DB, simply pass the password match the one you configured in db.env
    - EmailPassword: This is the password to login to your SMTP server using the user defined by `EmailAddress` above
5. If you want to stand up the database container and ATLAS, simply go into the directory where the `docker-compose.yml` file is located
    - Type:
    ```
    docker-compose up
    ```
        - This will start the `atlas-db` container
        - Once that is running, it will start the `atlas` container
        - The `atlas` container will wait for the database container to start and be listening on port 1433
6. If you have a database you want to point to, edit the `atlas.env` with your information and ensure your DB server is listening on port 1433
    - Simply run:
    ```
    docker run --env-file atlas.env -v /tmp:/atlas/files -p 81:80 c2labs.azurecr.io/atlas:dev
    ```
7. Following steps 5 or 6, ATLAS should now be running.
    - Point your browser to http://localhost:81
8. Login with the default credentials and **CHANGE THEM**
    - Username: `admin`
    - Password: `51mpl3Compliance$`
9. When you are done, you can clean up the containers with:
    ```
    docker-compose down
    ```
