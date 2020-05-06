## Atlas Windows 2019 Server Standalone Installation

### Install Docker
1. Install Docker and Restart Server

```
Install-Module DockerMsftProvider -Force
Install-Package Docker -ProviderName DockerMsftProvider -Force
(Install-WindowsFeature Containers).RestartNeeded
Restart-Computer
```

2. Test Docker

```
docker run hello-world:nanoserver
```
NOTE: Docker is not compatible with Symantec Endpoint Protection (SEP).  SEP should be disabled or specific policies implemented to allow Docker to function properly:

Set the following exceptions in SEP ADC (at a minimum):

* lsass.exe
* svchost.exe
* cexecsvc.exe
* oobe\windeploy.exe

![dockertest](screenshots/hello_world.png) 

### Create Location for Uploaded Files
ATLAS utilizes persistent storage to store any uploaded files. The files are stored on disk encrypted.
1. Create folder `C:\atlas\files` **OR**
2. If you are using another location, such as a mapped drive, make note of the location. We recommend the structure of `\atlas\files`.

### Setup Atlas Environment Variables File
1. Create folder `C:\atlas-install`
2. Copy the `atlas.env` file to `C:\atlas-install`
3. Update `atlas.env` with your JWT Token, Encryption Key, and Database Connection String
```
JWTSecretKey=<YourSuperSecretJWTSecretToken>
EncryptionKey=<YourSuperSecretEncryptionKey>
SQLConn=Server=tcp:<YourDatabaseURL>,1433;Initial Catalog=ATLAS;Persist Security Info=False;User ID=<YourDBUsername>;Password=<YourDBPassword;MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
```
**Notes on Variables**

1. Avoid unallowed special characters in the SQL Server password, [see MSDN article](https://docs.microsoft.com/en-us/sql/relational-databases/security/strong-passwords?view=sql-server-ver15).  In addition, avoid the use of the '$' and '=' characters as they can cause issues when being read in by Docker.
2. Set SQL Server to use static port 1433, [see MSDN article](https://kb.variphy.com/knowledge-base/how-to-set-static-tcp-port-1433-in-microsoft-sql-server-express/).  Dynamic ports are not currently supported by ATLAS.
3. With PowerShell, run the container using ```ContainerAdministrator```.  Otherwise, there will be issues with the certificate using HTTPS, [see GitHub Issue](https://github.com/dotnet/dotnet-docker/issues/915)
4. Set container to restart automatically with ``` --restart unless-stopped``` or ``` --restart always```, [see Docker article](https://docs.docker.com/config/containers/start-containers-automatically/)
5. For help creating the keys, you can use the following [generator](https://www.allkeysgenerator.com/Random/Security-Encryption-Key-Generator.aspx)
6. All installation work should be done as an Administator in Powershell.  You **cannot use Powershell ISE** as it is not supported for running Docker, [see GitHub Issue](https://github.com/docker/for-win/issues/223)

### Setup HTTPS
If you want to run without https enabled, comment out the ASPNETCORE environment variables in the `atlas.env` file and skip the following steps

1. Create and copy your .pfx file to the `C:\https` folder

2. Update the `atlas.env` file with your cert name and password

```
ASPNETCORE_Kestrel__Certificates__Default__Password=<YourSuperSecretCertPassword>
ASPNETCORE_Kestrel__Certificates__Default__Path=C:\https\<YourCertName.pfx>
```

### Start Atlas
1. Start the Atlas container using the following Docker Run command
  - **NOTE** If you moved the location of `C:\atlas\files` or mounted things to alternate drives, substitute that value in the `source` statement in the run command.

```
docker run -d --name atlas -p 443:443 --env-file atlas.env --mount type=bind,source="D:\Atlas\files",target=C:\atlas\files --mount type=bind,source="D:\HTTPS_Cert",target=C:\https,readonly --user ContainerAdministrator c2labs/atlas-c2internal:0.5.0-beta-win
```