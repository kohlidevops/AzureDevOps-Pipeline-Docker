# How to Build and Push the Docker Image to Docker Hub using Azure Pipeline

![image](https://github.com/kohlidevops/AzureDevOps-Pipeline-Docker/assets/100069489/ee5d0fb5-a094-433a-a15c-78147bfe45a8)

**Please refer below link to start this pipeline**

```
https://github.com/kohlidevops/AzureDevOps-CICD-Pipeline/blob/main/README.md
```

**Where is the source code repository?**

I can use below source code to Build and Push the Docker Image to Docker Hub Repository.

```
https://github.com/kohlidevops/DevopsBasics.git
```

By default, any build tools will see the Dockerfile in the repository and build the Docker image.

However, In Azure Cloud Pipeline, the Pipeline will see the AzureDockerfile in the source code repository.

**To create a repository in Docker Hub**

Login into Docker Hub and create a new repository called azuredevops

![image](https://github.com/kohlidevops/AzureDevOps-Pipeline-Docker/assets/100069489/5d2ce10b-be23-4b2d-b23b-1fa9929008eb)


**To setup a service connection in Azure**

Navigate to Azure devops portal and select the Project settings - Service Connection

![image](https://github.com/kohlidevops/AzureDevOps-Pipeline-Docker/assets/100069489/faceb487-55f5-4265-8bee-758dc9b063ac)

Choose New Service Connection - Select - Docker Registry - Next - Registry type - Docker Hub

```
Docker Registry - https://index.docker.io/v1/
Docker ID - latchudevops
Docker Password - ***********
Service connection name - dock2
```

Verify and save this service connection.

![image](https://github.com/kohlidevops/AzureDevOps-Pipeline-Docker/assets/100069489/9718cba8-92bc-49eb-8ad7-5f9b61b5beb0)

**To create a Pipeline to build the Artifacts**

Navigate to Azure devops portal and create a new Pipeline with source code as a Github repo

![image](https://github.com/kohlidevops/AzureDevOps-Pipeline-Docker/assets/100069489/421b7708-87cd-43ad-90cc-be074bdf7421)

My project is DevopsBasics

```
https://github.com/kohlidevops/DevopsBasics.git
```

I'm going to select the Maven tool to build the Artifacts before build and push the docker image.

![image](https://github.com/kohlidevops/AzureDevOps-Pipeline-Docker/assets/100069489/fc1a5738-5026-4c42-bc52-5bf999d5b5aa)

Choose - show Assistant - search and select the Docker from listed

```
Container Registry - dock2  //This is the service connection name which we created just before
command - login
```

Then add the Task. Your task would be 

![image](https://github.com/kohlidevops/AzureDevOps-Pipeline-Docker/assets/100069489/075683ee-d2e5-4a81-892f-4112ab8a0830)

Then add - Docker task again

```
Container registry - dock2
Container repository - latchudevops/azuredevops  //my docker hub username/repository name
Command - buildAndPush
Dockerfile - AzureDockerfile   //Azure blindly looks the name as AzureDockerfile instead of Dockerfile
Build context - **    //** to specify the directory that contains the Dockerfile
Tags - $(Build.BuildId)
```

Then add this task. Now my complete yaml file looks like below

```
# Maven
# Build your Java project and run tests with Apache Maven.
# Add steps that analyze code, save build artifacts, deploy, and more:
# https://docs.microsoft.com/azure/devops/pipelines/languages/java

trigger:
- master

pool:
  vmImage: ubuntu-latest

steps:
- task: Maven@3
  inputs:
    mavenPomFile: 'pom.xml'
    mavenOptions: '-Xmx3072m'
    javaHomeOption: 'JDKVersion'
    jdkVersionOption: '1.8'
    jdkArchitectureOption: 'x64'
    publishJUnitResults: true
    testResultsFiles: '**/surefire-reports/TEST-*.xml'
    goals: 'package'
- task: Docker@2
  inputs:
    containerRegistry: 'dock2'
    command: 'login'
- task: Docker@2
  inputs:
    containerRegistry: 'dock2'
    repository: 'latchudevops/azuredevops'
    command: 'buildAndPush'
    Dockerfile: '**/AzureDockerfile'
```

Then save and run this Azure Pipeline

![image](https://github.com/kohlidevops/AzureDevOps-Pipeline-Docker/assets/100069489/a2f585fd-f25e-4488-98a5-45b17ff91848)

I can see the azure-pipelines-4.yml available in my source code repository. Because this file has been updated or committed when i save my pipeline.

![image](https://github.com/kohlidevops/AzureDevOps-Pipeline-Docker/assets/100069489/2bf14428-dd82-4bab-965f-da0f2d0e71df)

Let's run my pipeline and see what happen

![image](https://github.com/kohlidevops/AzureDevOps-Pipeline-Docker/assets/100069489/8b27864a-7e60-433e-880e-10e35a2d23e4)

My Pipeline job is completed. 

![image](https://github.com/kohlidevops/AzureDevOps-Pipeline-Docker/assets/100069489/129e0355-2258-43f5-9d93-2cf5673dbe31)

If I check with my Docker Hub repo

![image](https://github.com/kohlidevops/AzureDevOps-Pipeline-Docker/assets/100069489/681eb8a1-e7f7-472a-bb6a-cad7e40b9415)

Yup! I could see the image which was pushed just now.

Does this Pipeline will Continuous Integrate? means - When source code changes, then will Azure Pipeline trigger and build/push the image to Docker Hub?

Definetly yes! Because we are having a parameter in a yaml file.

```
trigger
- master
```

![image](https://github.com/kohlidevops/AzureDevOps-Pipeline-Docker/assets/100069489/ae689f0e-de7c-4b43-a7d9-e826dd591803)

