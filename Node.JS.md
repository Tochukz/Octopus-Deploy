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

### Build  
The build process will handled by __Jenkins__, with the following steps:  
1. The latest code is pulled from the Git repository.  
2. All dependencies are installed using `npm install`.
3. Unit tests are run if available.
4. The software files and dependencies are bundled into a `ZIP` file.  
5. The packaged `zip` file is pushed to Octopus (built-in) package repository.

The `ZIP` file generated is prepended with the software ID and version number to make them unique for example _MyApp.1.0.1.zip_ where is software ID is _MyApp_ and the version number is _1.0.1_.  
If we deploy a bad release, we can go and find the older version of the artifact and re-deploy it.

### Prerequisites    
* Git
* SQL Server
* Jenkins Server
* Octopus Deploy Server
* Octopus CLI
* Node.js

### Jenkins Setup and Operations  
__Configure Jenkins credentials__
Same as for ASP.NET  

#### The Jenkins Project   
Create a free style project with the following stages and steps   


| Stage                                      | Steps1                                | Step2 | Step3 |
|--------------------------------------------|:-------------------------------------:|:-----:|------:|
| Description                                | A Demo Node.js App for Octopus Deploy |       |       |
| Source Code Management                     | https://github.com/Tochukz/AuthApp    |       |       |
| Build Triggers (Poll SCM)                  | H/5 * * * *                           |       |       |
| Build Environment (Use Secret text)        | Variable: OctopusAPIKey               |       |       |
| Build (Exe. Windows Batch command)         | npm install && npm test               | Octo pack -id AuthApp --version 1.0.%BUILD_NUMBER% --include "%WORKSPACE%\**" --outFolder "%WORKSPACE%" --format Zip | Octo push --server http://localhost:8082 --package "%WORKSPACE%\AuthApp.1.0.%BUILD_NUMBER%.zip" --apiKey %OctopusAPIKey% |                     

Save the setup and manually build the Jenkins project.  

The `zip` file must have been sent to the built-in octopus repository by now. Mine was found in `C:\Octopus\Packages\Spaces-1\feeds-builtin\AuthApp`.  

### Deployment with Octopus Deploy  
__Create environments__  
Login to Octopus Deploy server web, click on the `Infrastructure` menu on the top navigation bar, and click `Environments` on the left navigation and click the `Add Environment` button on the content area.

Create 3 environments - `Dev`, `Test` and `Prod`.  

#### The Octopus Project
Click to `Project` menu and `Add Project` button on the top right hand corner to create the project. 
