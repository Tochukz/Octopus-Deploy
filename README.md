# Octopus Deploy
A continuous integration/continuous deployment (CI/CD) tool. See [octopus.com/docs](https://octopus.com/docs)

Assuming you have installed Octopus Server, Jenkins Server, and their associated tools and dependencies, the general steps for building a CI/CD deployment pipeline with is as follows:  
1. Generate Octopus API Key
2. Install Jenkins Plugin if needed(for ASP.NET the _MSBuild_ plugin is required).
3. Add any installed Jenkin Plugin or `.exe` path to Jenkins _Global Tool Configuration_ (For ASP.NET, this is the path to `MSBuild.exe`).
4. Add Octopus API Key to Jenkins _Global Credential_ as a secret.
5. Create a Jenkins Project  
  * Remember to add the API key under the _Build Environment_ section as _Secret text_.
6. Create an Octopus Project

Here I outline and summarize the steps taken to build a CI/CD pipeline for various software stack including setup of tools and environments.
Here are the stacks covered.
* ASP.NET application on IIS using Octopus and Jenkins.
[See steps.](https://github.com/Tochukz/Octopus-Deploy/blob/master/ASP.NET.md)

* Node.js application on NGINX using Octopus and Jenkins. [See Steps.](https://github.com/Tochukz/Octopus-Deploy/blob/master/NODE.js.md)
