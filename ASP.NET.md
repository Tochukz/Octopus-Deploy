# Octopus Deploy Tutorial  

## Deploy an ASP.NET application to IIS using Octopus and Jenkins
For more details see [using-octopus-onprem-jenkins-builtin](https://octopus.com/docs/guides/deploy-aspnet-app/to-iis/using-octopus-onprem-jenkins-builtin)
### Software Stack
* ASP.NET
* IIS

### Tools  
* Git
* Jenkins
* Octopus Deploy
* IIS

### Web Application Server
* IIS

### Build
The build process is handled by __Jenkins__, with the following steps:  
1. The latest code is pulled from Git repository.
2. All dependencies are installed using `nuget restore`.  
3. The code is compiled using   `msbuild`.  
4. Unit tests are run if available.
5. The software files and dependencies are bundled into a _NuGet_ package (`.nupkg`).
6. The package is pushed to the Octopus (built-in) package repository.  

The `NuGet` package generated is prepended with the software ID and a build version number to make each build unique, for  example _MyApp.1.0.1.nupkg_, where the software ID is _MyApp_ and the version number is _1.0.1_.  
If we deploy a bad release, we can always go back and find the older version of the artifact and re-deploy it.

### Deployment  
The deployment process is handled by __Octupus__, with the following steps:  
1. Configurations files are transformed and settings that may differ for different environment in injected such as database connection string or API keys.   
2. The software code is uploaded to IIS.
3. Database migration may be done here. The application may be taken offline temporarily.   

### Prerequisites  
* Git
* SQL Server
* Jenkins server
* Octopus deploy server
* Octopus CLI
* Octopus Tentacle
* NuGet  
* .NET Framework Dev Pack [dotnet.microsoft.com/download](https://dotnet.microsoft.com/download)
* MSBuild
* IIS server


### Installations and Setup   
__Install Chocolatey__  
Chocolaty is a software management tool that is also a package manager. _choco_ is to Windows as _apt-get_ is to Linux's Ubuntu, but _choco_ is more. Go to [chocolatey.org](https://chocolatey.org) to install chocolatey.  

__Install Octopus CLI  using Chocolatey__   
Octopus CLI (_octo_), a tool that:
* creates and deploys releases,
* pushes packages, and
* manage environments with Octopus.
Install Octopus CLI using Chocoletey:
```
> choco install octupustools
```
To explore Octopus CLI help document   
```
> octo --help
```
You may also download Octopus CLI manually at [octopus.com/downloads](https://octopus.com/downloads). Extract the zip file, move it to you C drive and add the path to your system environment variable.

__Install nuget CLI using Chocolatey__   
Using Chocoletey:
```
>  choco install nuget.commandline
```
You may also download `nuget.exe` from [nuget.org/downloads](https://www.nuget.org/downloads) and install it manually.    
See [NuGet CLI](https://docs.microsoft.com/en-us/nuget/consume-packages/install-use-packages-nuget-cli) for basic use of the _NuGet CLI_.

__Install .NET Framework Dev Pack__  
Go to [https://dotnet.microsoft.com/download](https://dotnet.microsoft.com/download) to download the .NET Framework Dev Pack.

__Install MSBuild tool__  
A copy of the _MSBuild tool_ comes installed when we install Visual Studio. The _Web development build tools_ workload must be selected from _visual studio installer_ interface during installation.  
For an installed instance of visual studio version 2019, _MSBuild.exe_ may be found at `C:\Program Files (x86)\Microsoft Visual Studio\2019\Professional\MSBuild\Current\Bin`

__Install SQL Server__  
Octopus deploy server requires SQL Server to host its database. You can download and install SQL server from [microsoft.com/en-gb/sql-server/sql-server-downloads](https://www.microsoft.com/en-gb/sql-server/sql-server-downloads)

__Install Octopus Deploy Server__  
Download and install the Octopus Deploy Server from [https://octopus.com/downloads/server](https://octopus.com/downloads/server) and lunch the _Octopus Manager_ when installation finishes.  

__Install Jenkins__  
Go to [jenkins.io/download/](http://jenkins.io/download/) to download and install Jenkins.

__Install IIS__  
IIS is a feature in many Windows installation. To enable it you simple  _turn it on_ if it has not already been enabled.   
Go to `Control Panel` > `Programs` > `Programs and Features` and `Turn Windows features on or off` to turn on IIS.   

__Install Octopus Tentacles__
A _Tentacle_ is an Octopus deployment agent. The _Tentacle Manager_ is the windows application that configures your Tentacle. Download the Tentacle Windows installer [here](
https://octopus.com/downloads/latest/WindowsX64/OctopusTentacle) and install it on your target machine. Once installed you can access it from your start menu.

### Octopus Setup and Operation   
__Create the Octopus deployment project__  
You can click on the project menu and _Add Project_ button to create a new project.  

__NB:__ The [Configuration Variables feature](https://octopus.com/docs/deployment-process/configuration-features/xml-configuration-variables-feature) enables us to defined variables whose values can be used to replace values with the same `Key` in the `*.config` files under the `appSettings`, `connectionStrings` and `applicationSettings` element. We can define a given variable multiple time and scope it to different environment such as `dev`, `staging` and `production`.    
For JSON configuration files see [JSON Configuration variables feature](https://octopus.com/docs/deployment-process/configuration-features/json-configuration-variables-feature).

__Generate Octopus API Key__  
Jenkins will communicate with Octopus using an  _Octupus API key_. An _Octopus API Key_ can be generate using the Octopus web portal of your locally installed instance of Octopus deploy server:  
* Start Octopus manager
* Click on the link under `Octupus Web Port` and login to the web interface
* Click on you login name on the top left > `Profile`
* Click on `My API Key` on the left navigation bar.  
* Click `NEW API KEY` button to generate an API key.  
* Copy and save your API key in a safe place.  


### Jenkins Setup and Operations
__Install Jenkins MSBuild Plugin__  
* Login to Jenkins
* Go `Manage Jenkins` and then `Manage Plugins`
* Search for `MSBuild` and the click `Install without restart` to install the `MSBuild` plugin.  

__Configure the installed Jenkins MSBuild plugin__  
Now, we add `MSBuild` to the `Global Tool Configuration`
* Click on `Manage Jenkins`, then `Global Tool Configuration`
* Click on `Add MSBuild`
* Enter the string `MSBuild` for the _Name_ field input and a path to the installed `MSBUild.exe` for the _Path to MSBuild_ field input. The path to my `MSBuild.exe` is `C:\Program Files (x86)\Microsoft Visual Studio\2019\Professional\MSBuild\Current\Bin`.
* Click the `Save` button.  

__Configure Jenkins credentials__  
Here we add the Octupus API key we generated earlier to _Jenkins Global credentials_. The key must be save as a secret so that it is not displayed in the Jenkins logs.  
* Click on `Manage Jekins`, and then `Configure Credential`
* Click on `Credentail` > `System` from the left navigation.  
* Click on the `Global credentials(unrestricted)` link on the content area.  
* Click `Add Credentials` from the left navigation.
* Select `Secret Text` for the `Kind` field selection
* Enter the Octopus API Key for the `Secret` input box.
* Enter a memorable name for the ID input field  e.g 'OctopusAPIKey'
* Click `Ok`

#### The Jenkins Project  
__Create a Jenkins project__   
* Create a Jenkins project using the `Freestyle project` option.  
* Under the _Build Environment_ section, you must check the `Use Secret text(s) or files(s)` checkbox.  
* Under the _Bindings_ section, that appears after you check the checkbox, Click the `Add` selection drop down and select `Secret text`
* Enter `OctopusAPIKey` for the `Variable` input box and check the `Specific credentials` radio button.
* Select the `Octopus API Key` we configure earlier on for Jenkins credential.
The `OctopusAPIKey` variable will be used in a later step with the _Octopus CLI_

__Configure the Build Trigger__
For the build trigger section you can tick the _Poll SCM_ checkbox and use `H/60 * * * *` to poll for changes to you git repository every 60mins.

__Add Build Environment__   
Remember that we added Octopus API key as a secret to _Jenkins Global credentials_.
Now we must add a variable that reference that _Octopus API Key_  in the build environment.  

__Jenkins Build Steps__  
The following Windows Batch Commands are executed as build steps during the build stage.  
__Step 1:__ To install project dependencies use the `Execute Windows batch command` build step option and enter the command:
```
C:\ProgramData\chocolatey\bin\nuget.exe restore
```

__Step 2:__ To build the project use the `Build a Visual Studio project or solution using MSBuild` option with inputs as shown in the table below

| Input Label            |   Value      |
|------------------------|--------------|
| Build Version          | `MSBUild`    |
|MSBuild Build File      | `MyApp.sln`  |
| Command Line Arguments | `/p:RunOctoPack=true /p:OctoPackPackageVersion=1.0.$BUILD_NUMBER /p:OctoPackEnforceAddingFiles=true` |

__NB:__ The _OctoPack NuGet Package_ must be installed in main project to be deployed not the test or supporting project.  
_OctoPack_ adds a custom MSBuild target that hooks into the build process of the solution and package the project when MSBuild runs.  OctoPack works by calling `nuget.exe pack` to build the NuGet package, and `nuget.exe push` to publish the package (if so desired).   
OctoPack is not compatible with ASP.NET Core applications. If you want to package APS.NET Core applications see [create packages with the Octopus CLI](https://octopus.com/docs/packaging-applications/create-packages/octopus-cli).  

__Step 3:__ To run NUnit test use the `Execute Windows batch command` build step option and enter the command:
```
.\packages\NUnit.ConsoleRunner.3.10.0\tools\nunit3-console.exe .\MyApp.Tests\bin\Debug\MyApp.Tests.dll
```
The nunit3-console.exe executable is downloaded as part of a NuGet dependency included in the test project.  

__Step 4:__ To push the compiled package to your octopus server built-in feed, use the `Execute Windows batch command` build step option and enter the command:
```
Octo.exe push --server http://localhost:8082 --apiKey %OctopusAPIKey% --package MyApp/obj/octopacked/MyApp.1.0.%BUILD_NUMBER%.nupkg
```  
My octopus server is running at http://localhost:8082.  

__Step 5:__ To tell Octopus release the build to it's _Dev_ environment, use the `Execute Windows batch command` build step option and enter the command:  
```
Octo.exe create-release --server http://localhost:8082 --apiKey %OctopusAPIKey% --project "My App" --progress --deployto Dev
```

### Octopus Deployment
__Create Octopus environment__  
* Go to the Web platform and click on `Infrastructure` menu
* Click on the `Environments` link on the left navigation bar
* Click the `ADD ENVIRONMENT` button in the content area
* Enter the environment name `Dev` and click `Save`
* Repeat for `Test` and `Prod` environments.  

__Configure the Deployment Targets__  
Remember that you _Tantacle Manager_ which was installed as a prerequisite sits on each and every of your target machine.   
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

__Create the Octopus deployment project__  
* Click on the `Projects` menu and then `ADD PROJECT` button at the top right
* Enter you Project name and click `SAVE`
* Click `Variables` link on the navigation bar
* Add as many variables as needed. A variable should be some parameter that varies for each environment such as database connection string.
  * Use can use `#{Octopus.Environment.Name}` as value of a variable to dynamically reference the environment name for each environment. e.g `EnvName=#{Octopus.Environment.Name}`
  * You can define a single variable an scope different values of that variable to different environments. e.g IIS Port as a variable name and different port number for different environments.  
* Click on link `Deployments`  > `Overview` > `DEFINE YOUR DEPLOYMENT PROCESS`
* Click the `ADD STEP` button
* Select `Windows Server` and click `Deploy to IIS`
* Enter a descriptive name for your selected step.
* Select a web role that marches one registered by your Tentacle. That is `web` in our case.
* Select the package ID e.g `MyApp`.
  * This is ID of package that was pushed to Octopus repository by Jenkins  
* Enter your website details and click `SAVE` button

__Create an Octopus Release__
* Click on the `Release` link on the left navigation bar
* Click `CREATE RELEASE` on the top right
  * You can select the package version to release. By default it will be the latest version
  * You may also enter a release note.
* Click `SAVE` button
* On the release screen you can select the environment you want to deploy to. You can also configure the lifecycle for progression of deployment through environments.
* Click on the `DEPLOY` or `DEPLOY TO DEV` button to do a manually deployment to `Dev` environment.
* Deploy to `Dev` will be automated on the next Jenkins build.  
