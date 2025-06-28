---
id: lab6
title: Lab 6 - Amazon Elastic Container Service (ECS)
sidebar_position: 6
description: Configure the Elastic Container Service (ECS)
---

# Lab 6 - ECS

# Under Development

## Overview
In this lab, you will learn how to build, run, and deploy a simple web application using Docker containers, and then use Amazon Elastic Container Service (ECS) to manage and orchestrate a containerized application in the cloud. 

Containers are lightweight, portable units that package an application and all its dependencies together, ensuring that the application runs consistently across different computing environments. Docker is a popular platform for creating and managing containers.

You will start by creating a local Docker container for a web application, then push your container image to docker hub (a container registry). Next, you will configure and deploy a web application using Amazon ECS, a fully managed service that makes it easy to run and scale containerized applications on AWS. By the end of this lab, you will understand the basics of containerization and how to leverage ECS for deploying and managing containers in a cloud environment.

## Investigation 1: Creating a local web application using Docker
In this investigation you will install the Docker Desktop software, create a simple web page and docker file. You will then use these to deploy your container locally.

### Installing the required local software
Install the following required software:
#### Windows Subsystem for Linux
- [Windows Subsystem for Linux](https://learn.microsoft.com/en-us/windows/wsl/install)

#### Docker
- [Windows](https://docs.docker.com/desktop/setup/install/windows-install/)
- [Mac](https://docs.docker.com/desktop/setup/install/mac-install/)
- [Linux](https://docs.docker.com/desktop/setup/install/linux/)

#### Visual Studio Code (VSCode)
- [VS Code](https://code.visualstudio.com/)

### Creating a Docker account
Browse to the [Docker website](https://app.docker.com/signup?state=hKFo2SB5X0ZIQVQxTVBFY0RzLXpWQ0VwZi14dC01SWVvbUhrQqFur3VuaXZlcnNhbC1sb2dpbqN0aWTZIFVPcVREalBsd19qSF9zSVRSMTJtZU5Ya2IwV1RYM2pto2NpZNkgRmkyQ0VleDZtMzhkS1lxdnZaWVljSms5SUN0cGc3ZzQ) and follow the steps to create an account. You can select **Personal** for type and use your personal email.

### Creating a web page
- Create a local directory called **lab6**.
- Open VS Code and create the file **index.html** with the following contents (replacing **your name** with your name). Save it in the **lab6** directory.
```html
<!DOCTYPE html>
<html>
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta name="author" content="Your Name">
        <title>My Web Page</title>
    </head>
    <body>
        <h1>Your name's web page</h1>
        <p>Hello World</p>
    </body>
</html>
```

- In the same directory create the file **Dockerfile** with the following contents. When VS Code prompts you to install the Docker extension, install it.
```bash
FROM httpd:latest
WORKDIR /usr/local/apache2/htdocs
COPY . /usr/local/apache2/htdocs
EXPOSE 80
```

### Building and running your application
- In VS Code click on **View** > **Terminal** and issue the following commands.

```bash
docker build -t my-apache-app .
docker run --name my-apache-app -d -p 80:80 -t my-apache-app
```

### Viewing your web page
- Open a browser and enter localhost in the URL bar
- You should see your webpage

### Pushing your code to dockerhub
#### Creating a dockerhub account and repository
- Create a dockerhub account: https://hub.docker.com
- Create a repository. Name it my-apache-app

#### Generating a gpg key locally
[gpg --generate-key](https://docs.docker.com/desktop/setup/sign-in/)


#### Generate a personal access token
- Generate a personal access token on dockerhub - save this.

- Generate GPG key
```bash
gpg --generate-key
```

- Initialize the key
```bash
pass init ID
```

- Login
```bash
docker login -u username
```
  - Paste the PAT

- Tag your local image
```bash
docker tag local-image:my-apache-app new-repo:my-apache-app
```

#### Pushing your local docker file to dockerhub
- Push your image to docker hub
```bash
docker push jmcarman/my-apache-app:latest
```

## Investigation 2: Amazon ECS (Elastic Container Service) Configuration
 
> [!NOTE]
> These instructions have been updated to use the new ECS UI. If you notice any bugs in the instructions, please let your professor know so that they can be fixed.

Now that we have a general idea of what ECS entails, we'll learn how it works by _manually_ setting-up an ECS Cluster, Service, and Task for our `my-apache-app` microservice.

1. Start the **AWS Academy Learner Lab** and **AWS Console**. Search for **Elastic Container Service**.
2. We'll need to define our **Container definition**, **Task definition**, **Service**, and **Cluster**, and we'll begin with the **Task Definition**.

### Task Definition

As we said above, the **Task Definition** defines how to run our container with ECS. Let's create one for our `my-apache-app` server:

1. In the **Amazon Elastic Container Service** console, choose **Task definitions** in the left-hand menu

#### Task definition configuration

1. Click the **Create new task definition** button and **Create new task definition**.
2. In the **Task definition configuration**, give your new task definition a **family name**: `my-apache-app`. This name will be the basis for the many individual task definition versions that we will create in the coming weeks.

#### Infrastructure requirements

As we know from our work with EC2, running a container on AWS requires compute resources (e.g., EC2 instances), and these need to be provisioned and set with sufficient resources. Previously we did this manually, but now we will use the managed Fargate service to handle our compute.

1. Under **Infrastructure requirements** we specify the infrastructure computing needs for our task. We will be using **AWS Fargate** serverless compute and **Linux/X86_64** for our **Operating system/Architecture**.
2. The **Task size**, defines how much **CPU** and **Memory** to allocate. Accept the defaults.
3. In terms of security, we need to define two different [IAM Roles](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles.html) within our AWS account. First, the **Task role** specifies the rights that the container running within the task will have (e.g., to be able to access other AWS resources in our account); second, the **Task execution role** specifies what rights that the ECS cluster and infrastructure will have (e.g., to access resources like our ECR repo). Set both of these roles to the pre-defined **LabRole** AWS Role, which will grant them access to any resources we own.

#### Container - 1

Next we define the container (or containers if running multiple), that will be run in our task. This step is a bit like defining the options to pass to `docker run` on the command line.

> [!IMPORTANT]
> In order to make the following changes work, your Docker image needs to use the `root` user. If you have previously set `USER node` or similar, you'll need to remove this and use the default `root` user instead. This is necessary because we are going to use a privileged port (i.e., port below 1024), which requires elevated rights. You should make that change if necessary then push new images to your Docker registries. If you do not do this, your server will crash with an `EACCESS: permission denied 0.0.0.0:80` error when it tries to listen on port 80.

1. For the **Container details**, use a **Name** of `my-apache-app`
1. For **Image URI**, use `public.ecr.aws/docker/library/httpd:latest`
1. For the **Port mappings**, we'll use the production HTTP port, `80`. This is the default configuration: **Container port** is `80`, uses the `TCP` **Protocol**, and that the **App protocol** is `HTTP`. You can leave the **Port name** empty.
1. For the **Resource allocation limits**, specify the amount of **CPU**, `1`, and **Memory**, `1` for the **Memory soft limit**. This means your container will reserve `512 MiB` of RAM when it starts (a **Hard limit** would define the _maximum_ memory allowed, which we won't set).
1. The default values for **Logging** are good. ECS will use the [Amazon CloudWatch](https://aws.amazon.com/cloudwatch/) service to collect our container logs into a [Log Group](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/CloudWatchLogsConcepts.html) named `/ecs/fragments-task`. You should write this down for later, when you need to see the logs for your server.
1. The default values for **Storage** are also good, since we won't be storing our data on disk (i.e., currently we use an in-memory database and later we'll use Amazon database services).

Everything else should be good using the default configuration. You can now click the **Create** button.

Your `my-apache-app` Task Definition is now created, and you can inspect or edit it at any time by coming back to **Amazon Elastic Container Service** > **Task definitions** > **my-apache-app**. When you do make changes, a new version will be created (i.e., this first version is **Revision 1**, `my-apache-app:1`).

## Investigation 3: Creating a Cluster
Next we need to create a cluster where we can run our task.

1. Click the **Create cluster** button
2. Give your cluster a name: `my-apache-app`
3. Under **Infrastructure**, choose **AWS Fargate (serverless)**
4. The default settings are good for all other options
5. Click the **Create** button

It will take a minute or two for the cluster to be fully created. Once it is, move on to create your **Service**.

### Creating a service

The final step is to create a **Service** within our cluster. A service is responsible for deploying, managing, and monitoring our tasks (and the containers they run).

Click on the newly created `my-apache-app` cluster, then under **Services**, click the **Create** button

For our service's **Environment** we will use our **Existing cluster**, `my-apache-app`. The **Compute configuration** should be set to `Launch Type` instead of the more advanced `Capacity provider strategy`. Our **Launch type** will be `FARGATE` and the **Platform version** will be `LATEST`. This will allow our service to directly launch instances via Fargate to run our tasks.

#### Service Details

1. For the **Task definition**, choose the **Family** we created in the previous steps, `my-apache-app`, and the only **Revision** we have, `1 (LATEST)`. Our service will use this task definition to create and manage our task for us.
1. Next, select the default **Service name**.

# Environment
1. For **Compute options** select **Launch Type**
1. For **Launch type** select **FARGATE**
1. For **Platform version** select **latest**

#### Deployment configuration
1. We will use a **Service type** of `Replica`, which allows us to run multiple, simultaneous versions of our task for high-availability. To start, we'll choose to set the **Desired tasks** to `1` (i.e., only run a single task with a single instance of our server).
1. Leave the rest of the defaults.

#### Networking
1. Under **VPC** select **Wordpress VPC**
1. Under **Subnets** make sure the following are selected (they should be by default)
  - Public Subnet 1
  - Public Subnet 2
  - Private Subnet 1
  - Private Subnet 2

#### Security Group
1. Under **Security group** select **Use an existing security group**
1. From the **Security group name** drop down menu check **Wordpress Website SG** and uncheck **default**
1. Leave **Public IP** as **Turned on** to auto-assign a public IP to your tasks.

Click **Create**. It may take several minutes to create.

### Viewing your Apache web page
1. Go to **EC2** > **Network & Security** > **Network Interfaces**
1. You should notice a new interface with a description beginning with **arn:aws:ecs:us-east-1:**
1. Check the box beside it
1. Copy the **Public IPv4 address** and paste it in your web browser.
1. You should see the default Apache web page, displaying the text **It works!**

In this lab you have created a local Apache container, and a container in Amazon ECS with the default Apache image. You will be using Elastic Beanstalk to deploy a Wordpress website in Lab 8. Elastic Beanstalk uses containers, although it automates the process of creating and deploying them.
