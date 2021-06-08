# Lifecycles  
Lifecycle is defined by phases which controls the order of promotion of a release.
A phase can have one or more environments.  

__Create lifecycle__
* Click on the `Library` menu
* Click on the `Lifecycle` link on the left navigation bar.  
* Click on the `Add Lifecycle` button.
* Define your `Retention Policy`
* Click on `Add Phase` to explicitly define the phases for the lifecycle.
*  
* Give the lifecycle a name, description and save.  

If you have a project setup with [Automatic Release Creation](octopus.com/docs/projects/project-triggers/automatic-release-creation) and set your first phase and environment to automatically deploy, pushing a package to the internal library will trigger both a release, and a deployment to that environment.
