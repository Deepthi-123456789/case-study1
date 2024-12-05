CI/CD Pipeline Documentation
This document provides a comprehensive overview of the CI/CD pipeline script designed for automating the process of building, publishing, and deploying a web application. The pipeline is implemented using Jenkins and is configurable based on user input parameters.

Understand the Project
----------------------
Assume we are deploying a sample web application.

Infrastructure Setup:
--------------------
>Click Launch Instances.
>Name and tags: Enter a name for your instance.(web)
>Application and OS Images (AMI): Choose Ubuntu Server 
>Instance type: Select t3.medium.
>change the inbound rule for securitygroup
Key pair:
----------
Select Proceed without a key pair. and launch the innstance
>git
>Java and Jenkins installed.
>docker
>nexus
> Kubernetes cluster (e.g., Minikube, EKS) 
>install zip(to create artifacts)
Install minikube,kubectl and helm


Jenkins
--------
open browser and URL Format: http://<server-ip>:8080
Replace <server-ip>
When accessing Jenkins for the first time, it asks for an initial admin password.
Locate the password:
      sudo cat /var/lib/jenkins/secrets/initialAdminPassword
Use your Jenkins credentials (username and password or token).

Jenkins:

Installed with required plugins:(managejenkins->plugins)
Git Plugin
Pipeline Plugin
Docker Pipeline
Kubernetes CLI Plugin
nexus artifactuploader
Restart jenkins
Configured with credentials for Docker registry (managejenkins->creditials) 

Nexus
-------
login to Nexus  http://<server-ip>:8081
Default credentials:
Username: admin
Password: Found in the admin.password file:
File location: nexus-data/admin.password.
Log in and change the default password on the first login.

>after login go to settings->reposetries->create reposetrie->select maven2(hosted)->create a nexus repo(eg:web)


Create a Jenkins Pipeline
------------------------
Step 1: Create a New Pipeline Job
Log in to Jenkins.
Click on "New Item".
Enter a name for the pipeline (e.g., case-study).
Select "Pipeline" as the project type and click OK.
Step 2: Configure the Pipeline Job
Under the General tab, check GitHub project and provide the repository URL.
In the Build Triggers section:

Check GitHub hook trigger for GITScm polling to allow webhook-based builds.
In the Pipeline section:
Choose Pipeline script from SCM.
Select Git as the SCM.
Provide the repository URL. Example: https://github.com/Deepthi-123456789/case-study1.git
Enter the branch to build (e.g., main or master).
Specify the script path if your pipeline file is named differently or not in the root directory (e.g., Jenkinsfile).
Save the configuration.

In Github
-----------
>Open Your GitHub Repository
>settings->webhook
>Enter your Jenkins URL with /github-webhook/ appended to it.
Example: http://your-jenkins-url/github-webhook/.
>Content type: Select application/json
>Under Which events would you like to trigger this webhook?:
 Select Just the push event.
 Click Add webhook

 >now we can build the pipeline
  
  
