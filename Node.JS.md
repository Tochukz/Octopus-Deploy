# Octopus Deploy Tutorial  

## Deploy a Node.js application to NGINX using Octopus and Jenkins

### Software Stack  
* Node.js
* Nginx

### Tools  
* Git
* Jenkins
* Octopus Deploy  
* Nginx  

### Web Application Server
* NGINX

### Build  
The build process will handled by __Jenkins__, with the following steps:  
1. The latest code is pulled from the Git repository.  
2. All dependencies are installed using `npm install`.
3. Unit tests are run if available.
4. The software files and dependencies are bundled into a `ZIP` file.  
5. The packaged `zip` file is pushed to Octopus (built-in) package repository.

The `ZIP` file generated is prepended with the software ID and version number to make them unique for example _MyApp.1.0.1.zip_ where is software ID is _MyApp_ and the version number is _1.0.1_.  
If we deploy a bad release, we can go and find the older version of the artifact and re-deploy it.

### Deployment  
The deployment process is handled by __Octupus__, with the following steps:  
1. Configurations files are transformed and settings that may differ for different environment is injected, such as database connection string or API keys.   
2. Database migration may be done here. The application may be taken offline temporarily

### Prerequisites    
* Git
* SQL Server
* Jenkins Server
* Octopus Deploy Server
* Octopus CLI
* Node.js

### Installations and Setup  
__Install Octopus CLI__  
Download Octopus CLI form [octopus.com/downloads](https://octopus.com/downloads). Extract the zip file, move it to you C drive and add the path to your system environment variable.

__Install SQL Server__  
Octopus deploy server requires SQL Server to host its database. You can download and install SQL server from [microsoft.com/en-gb/sql-server/sql-server-downloads](https://www.microsoft.com/en-gb/sql-server/sql-server-downloads)

__Install Octopus Deploy Server__  
Download and install the Octopus Deploy Server from [https://octopus.com/downloads/server](https://octopus.com/downloads/server) and lunch the _Octopus Manager_ when installation finishes.  

__Install Jenkins__  
Go to [jenkins.io/download/](http://jenkins.io/download/) to download and install Jenkins.

__Install NGINX__  
Todo: Write how to
You can change the port on whihc NGINX  listen by updateing its config file: `/etc/nginx/sites-enabled/default`  
```
listen 80 default_server;
```
NGINX must be restarted to take effect after a change to its config file.

### Octopus Setup and Operation   
__Create the Octopus deployment project__  
You can click on the project menu and _Add Project_ button to create a new project.

__NB:__ The [JSON Configuration variables feature](https://octopus.com/docs/deployment-process/configuration-features/json-configuration-variables-feature) enables us to defined variables whose values can be used to replace values with the same `Key` in the JSON `*.config` files

__Generate Octopus API Key__  
Jenkins will communicate with Octopus using an  _Octupus API key_. An _Octopus API Key_ can be generate using the Octopus web portal of your locally installed instance of Octopus deploy server:  
* Start Octopus manager
* Click on the link under `Octupus Web Port` and login to the web interface
* Click on your login name on the top left > `Profile`
* Click on `My API Key` on the left navigation bar.  
* Click `NEW API KEY` button to generate an API key.  
* Copy and save your API key in a safe place.  


### Jenkins Setup and Operations  
__Configure Jenkins credentials__  
Here we add the Octupus API key we generated earlier to _Jenkins Global credentials_. The key must be save as a secret so that it is not displayed in the Jenkins logs.  
* Click on `Manage Jekins`, and then `Configure Credential`
* Click on `Credentail` > `System` from the left navigation.  
* Click on the `Global credentials(unrestricted)` link on the content area.  
* Click `Add Credentials` from the left navigation.
* Select `Secret Text` for the `Kind` field selection
* Enter the Octopus API Key in the `Secret` input box.
* Enter a memorable name for the ID input field  e.g 'OctopusAPIKey'
* Click `Ok`

#### The Jenkins Project   
Create a free style project with the following stages and steps   


| Stage                                      | Steps1                                | Step2 | Step3 |
|--------------------------------------------|:-------------------------------------:|:-----:|------:|
| Description                                | A Demo Node.js App for Octopus Deploy |       |       |
| Source Code Management                     | https://github.com/Tochukz/AuthApp    |       |       |
| Build Triggers (Poll SCM)                  | H/5 * * * *                           |       |       |
| Build Environment (Use Secret text)        | Variable: OctopusAPIKey               |       |       |
| Build (Exe. Windows Batch command)         | npm install && npm test               | Octo pack -id AuthApp --version 1.0.%BUILD_NUMBER% --include "%WORKSPACE%\**" --outFolder "%WORKSPACE%" --format Zip | Octo push --server http://localhost:8082 --package "%WORKSPACE%\AuthApp.1.0.%BUILD_NUMBER%.zip" --apiKey %OctopusAPIKey% |                     

Save the setup and manually build the Jenkins project by clicking the `Build Now` button. You can take a look at the _Console Output_ to see the build process in real time.

The `zip` file must have been sent to the built-in octopus repository by now. Mine was found in `C:\Octopus\Packages\Spaces-1\feeds-builtin\AuthApp`.  

### Octopus Deployment
__Create Octopus environment__  
* Go to the Web platform and click on `Infrastructure` menu
* Click on the `Environments` link on the left navigation bar
* Click the `ADD ENVIRONMENT` button in the content area
* Enter the environment name `Dev` and click `Save`
* Repeat for `Test` and `Prod` environments.  

__Create the Octopus deployment project__  
* Click on the `Projects` menu and then `ADD PROJECT` button at the top right
* Enter you Project name and click `SAVE`
* Click `Variables` link on the navigation bar
* Add as many variables as needed. A variable should be some parameter that varies for each environment such as database connection string.
  * You can use `#{Octopus.Environment.Name}` as value of a variable to dynamically reference the environment name for each environment. e.g `EnvName=#{Octopus.Environment.Name}`
  * You can define a single variable and scope different values of that variable to different environments. e.g IIS Port as a variable name and different port number for different environments.  
* Click on link `Deployments`  > `Overview` > `DEFINE YOUR DEPLOYMENT PROCESS`
* Click the `ADD STEP` button
* Select `Package` and click `Deploy to NGINX`
* Enter a descriptive name for your selected step.
* Select a web role that marches one registered by your Tentacle. That is `web` in our case.
* Select the package ID e.g `MyApp`.
  * This is ID of package that was pushed to Octopus repository by Jenkins  
* Enter your website details and click `SAVE` button.

__Configure the Deployment Targets__  
Remember that your _Tantacle Manager_ which was installed as a prerequisite sits on each and every of your target machine.   
* Start the _Tantacle Manager_ from your start menu
* Add a New Tentacle and click on the `GET STARTED` button  
* For _Communication Style_ use `Polling Tentacle` and click `NEXT`.
* For storage live the default. It may be something line this `C:\Octopus\TantacleName` and `C:\Octopus\Applications\TantacleName` for logs and application installs respectively.
* For proxy support [learn more](https://octopus.com/docs/infrastructure/deployment-targets/proxy-support)
* Add your Octopus server URL credential and click `NEXT`. (You may use Octopus API KEY or User/Password)
* For _Machine Type_ use `Deployment Target`  
* Your _Display name_ will be the name that will be used to identify your target at the Octopus deploy server.  
* Select the environment you want you Tentacle to poll from or listen to.
* Choose or create a roll say `web` and click `NEXT`
* Leave `Tenants` and `Tenant tags` blank
* Finally, click `Install` to install your newly configured Tentacle.  

__Create an Octopus Release__
* Click on the `Release` link on the left navigation bar
* Click `CREATE RELEASE` on the top right
  * You can select the package version to release. By default it will be the latest version
  * You may also enter a release note.
* Click `SAVE` button
* On the release screen you can select the environment you want to deploy to. You can also configure the lifecycle for progression of deployment through environments.
* Click on the `DEPLOY` or `DEPLOY TO DEV` button to do a manually deployment to `Dev` environment.  

__Continuous deployments__
