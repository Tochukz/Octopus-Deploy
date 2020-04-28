# Octopus Deploy Tutorial  

## Deploy an ASP.NET application to IIS using Octopus and Jenkins

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

The `NuGet` package generated is prepended with the software ID and version number to make them unique for example _MyApp.1.0.1.nupkg_ where is software ID is _MyApp_ and the version number is _1.0.1_.  
If we deploy a bad release, we can go and find the older version of the artifact and re-deploy it.

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
* NuGet  
* .NET Framework Dev Pack [dotnet.microsoft.com/download](https://dotnet.microsoft.com/download)
* MSBuild
* IIS server


### Setup and Installations  
__Install Chocolatey__  
Chocolaty is a software management tool that is also a package manager. _choco_ is to Windows as _apt-get_ is to Linux's Ubuntu, but _choco_ is more. Go to [https://chocolatey.org](https://chocolatey.org) to install chocolatey.  

__Install Octopus CLI  using Chocolatey__   
Octopus CLI (_octo_), a tool to create and deploy releases, create and push packages, and manage environments with Octopus.
```
> choco install octupustools
```
Explore _octo_:    
```
> octo --help
```

__Install nuget CLI using Chocolatey__   
```
>  choco install nuget.commandline
```
See [NuGet CLI](https://docs.microsoft.com/en-us/nuget/consume-packages/install-use-packages-nuget-cli) for basic use of the _NuGet CLI_.

__Install .NET Framework Dev Pack__  
Go to [https://dotnet.microsoft.com/download](https://dotnet.microsoft.com/download) to download the .NET Framework Dev Pack.

__Install MSBuild tool__  
A copy of the _MSBuild tool_ comes installed when we install Visual Studio. The _Web development build tools_ workload must be selected from _visual studio installer_ interface during installation.  
For an installed instance of visual studio version 2019, _MSBuild.exe_ may be found at `C:\Program Files (x86)\Microsoft Visual Studio\2019\Professional\MSBuild\Current\Bin`

__Install SQL Server__  
oCtopus deploy server requires SQL Server to host its database. You can download and install SQL server from [microsoft.com/en-gb/sql-server/sql-server-downloads](https://www.microsoft.com/en-gb/sql-server/sql-server-downloads)

__Install Octopus Deploy Server__  
Download and install the Octopus Deploy Server from [https://octopus.com/downloads/server](https://octopus.com/downloads/server) and install it and lunch the _Octopus Manager_ when installation finishes.  

__Install IIS__  
IIS come as a feature in many Windows installation. GO to `Control Panel` > `Programs` > `Programs and Features` and `Turn Windows features on or off` to turn on IIS  if you have not already done so.   
You can also turn on IIS by using the terminal command  
```
> Start /w pkgmgr /iu:IIS-WebServerRole;IIS-WebServer;IIS-CommonHttpFeatures;IIS-StaticContent;IIS-DefaultDocument;IIS-DirectoryBrowsing;IIS-HttpErrors;IIS-ApplicationDevelopment;IIS-ASPNET;IIS-NetFxExtensibility;IIS-ISAPIExtensions;IIS-ISAPIFilter;IIS-HealthAndDiagnostics;IIS-HttpLogging;IIS-LoggingLibraries;IIS-RequestMonitor;IIS-Security;IIS-RequestFiltering;IIS-HttpCompressionStatic;IIS-WebServerManagementTools;IIS-ManagementConsole;WAS-WindowsActivationService;WAS-ProcessModel;WAS-NetFxEnvironment;WAS-ConfigurationAPI;IIS-ASPNET45
```

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
Here we add the Octupus API key we generated earlier to _Jenkins Global credentials__. The key must be save as a secret so that it is not displayed in the Jenkins logs.  
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

__NB:__ The _OctoPack NuGet Package_ must be installed in main project to be deployed not the test or supporting project.  
_OctoPack_ adds a custom MSBuild target that hooks into the build process of the solution and package the project when MSBuild runs.  OctoPack works by calling `nuget.exe pack` to build the NuGet package, and `nuget.exe push` to publish the package (if so desired).   
OctoPack is not compatible with ASP.NET Core applications. If you want to package APS.NET Core applications see [create packages with the Octopus CLI](https://octopus.com/docs/packaging-applications/create-packages/octopus-cli).


__Install Octopus Tantacles__

### Octopus Setup and Operation   
__Create the Octopus deployment project__  
The [Configuration Variables feature](https://octopus.com/docs/deployment-process/configuration-features/xml-configuration-variables-feature) enables us to defined variables whose values can be used to replace values with the same `Key` in the `*.config` files under the `appSettings`, `connectionStrings` and `applicationSettings` element. We can define a given variable multiple time and scope it to different environment such as `dev`, `staging` and `production`.    
For JSON configuration files see [JSON Configuration variables feature](https://octopus.com/docs/deployment-process/configuration-features/json-configuration-variables-feature).

__Octupus API Key__  
Jenkins will communicate with Octopus using an  _Octupus API key_. An _Octopus API Key_ can be generate using the Octopus web portal of your locally installed instance of Octopus deploy server:  
* Start Octopus manager
* Click on the link under `Octupus Web Port` and login to the web interface
* Click on you login name on the top left > `Profile`
* Click on `My API Key` on the left navigation bar.  
* Click `NEW API KEY` button to generate an API key.  
* Copy and save your API key in a safe place.  
