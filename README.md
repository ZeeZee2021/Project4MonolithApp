# Break a Monolith Application into Microservices with Amazon Elastic Container Service, Docker, and Amazon EC2

In this project, you will deploy a monolithic node.js application to a Docker container, thenl break the application into microservices using Amazon Elastic Container service, Docker and Amazon Elastic Compute Cloud (EC2) without any downtime.

## Monolithic Application/Architecture?

It is a single-tiered application designed to handle multiple related tasks from a single platform. They are typically complex applications that encompass several tightly coupled functions. Given its broad scope, monolithic tools tend to have large code bases hence when making a small change in a single function, you are required to compile and test the entire platform. 


##  Microservice Architecture

This is an application architecture that involves smaller applications deployed independently as loosely coupled services, tied together through application integration. Typically microservices are used to speed up application development.

## What are Containers?

Containers are a form of operating system virtualization that bundles an application’s code together with the related configuration files, libraries as well as dependencies required for the app to run.  They provide a lightweight, immutable infrastructure for application packaging and deployment. 

## Why Use Containers?

* They enable developers and IT professionals to deploy applications across environments with little to no modification.
* It provides more consistent operation.
* They require less system resources resulting in less overhead.
* It allows for greater modularity.
* Supports modern development and architecture. 


## **Containerize the Monolith**


You will build a Docker Container image using the `node.js` application and then push it to Amazon Elastic Container Registry. 

### **Prerequisite**

Create / Sign in to your AWS account

Install Docker

Install the AWS CLI 

Have a text editor (Atom, VS Code)

### Step 1 

***Install Docker***

The first step is to install Docker on your local machine. To get started, you need to update your existing package by running:

    $ sudo apt update

Next, you need to run some prerequisite packages, click `Y` when prompted:

    $ sudo apt install apt-transport-https ca-certificates curl software-properties-common 

Now you need to add the `GPG` key for the official Docker repo:

     $ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add - 

Next, add the Docker repository to `APT` sources:

    $ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"


To make sure you installed from the Docker repository instead of the default Ubuntu repo, run:

    $ apt-cache policy docker-ce

Now to install Docker, run the command and press `Y` and `ENTER` when prompted:

    $ sudo apt install docker-ce

Docker is installed and the daemon started. To check that it is running:

    $ docker - - version


![](./images/docker-version.png)



**Install AWS CLI** 

You need to download the AWS CLI package in a zip file using :


    $ curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"

![](./images/cli-install1.png)


Next you unzip the file:

    $ unzip awscliv2.zip

![](./images/cli-install2.png)

Now, run and install the program:

    $ sudo ./aws/install

Once completed, you can verify the installation by checking the version:

    $ aws --version

![](./images/cli-version.png)


### Step 2 

***Download and Open the Project*** 

*Getting the Repo*

For this project, I will use VS Code as my editor but feel free to use whatever editor you are most comfortable with. 

Navigate to https://github.com/awslabs/amazon-ecs-nodejs-microservices , using `git clone` on your local terminal, download/clone this repository. 

Start your editor, select **Open New Folder** or **Add Project Folder**, select the folder where you saved the repository ***amazon-ecs-nodejs-microservices***. 


### Step 3

***Creating the Repository***

On your AWS account, using the search box, type `ECR` and select the *Amazon Elastic Container Registry*. 

On the Repository page, select **Create a Repository**.

![](./images/ECR1.png)

In the **General** settings section, under Repository name, type ***api*** as the name. Leave everything else as is. Scroll to the bottom of the page and select **Create repository**.

The repository address is in the following format: 

[account-ID].dkr.ecr.[region].amazonaws.com/[repo-name]. The [account-ID], [region], and [repo-name] will be specific to your setup.

![](./images/ecr2.png)


### Step 4 


***Using the terminal to authenticate Docker log in***

Navigate into the following directory: 

 `~/amazon-ecs-nodejs-microservices/2-containerized/services/api`.

    $ cd ~/amazon-ecs-nodejs-microservices/2-containerized/services/api 

 Now run:

    $ sudo docker login -u AWS -p $(aws ecr get-login-password --region us-east-1) 286396332837.dkr.ecr.us-east-1.amazonaws.com


> *Error: Unable to locate credentials*. 

![](./images/cli-login-error.png)


If you don’t have an access key configured for your user, go back to your AWS account and follow the below steps. If you already have one, move on to the next step.

#### To create access keys for an IAM user
1. Sign in to the AWS Management Console and open the IAM console at https://console.aws.amazon.com/iam/.
2. In the navigation pane, choose **Users**.
3. Choose the name of the user whose access keys you want to create, and then choose the **Security credentials** tab.
4. In the **Access keys** section, choose **Create access key**.
5. To view the new access key pair, choose **Show**. You will not have access to the secret access key again after this dialog box closes. Your credentials will look something like this:

* Access key ID: AKIAIOSFODNN7EXAMPLE
* Secret access key: wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY

6. To download the key pair, choose **Download .csv file**. Store the keys in a secure location. You will not have access to the secret access key again after this dialog box closes.

    Keep the keys confidential in order to protect your AWS account and never email them. Do not share them outside your organization, even if an inquiry appears to come from AWS or Amazon.com. No one who legitimately represents Amazon will ever ask you for your secret key.
7. After you download the `.csv `file, choose **Close**. When you create an access key, the key pair is active by default, and you can use the pair right away.

Configure credentials by running `aws configure`. Using your **Access key ID** and **Secret key**, log in to your AWS. 

Now you can rerun the below code and should get a **Login Succeeded**. 

    $ sudo docker login -u AWS -p $(aws ecr get-login-password --region us-east-1) 286396332837.dkr.ecr.us-east-1.amazonaws.com

![](./images/cli-login.png)



To build the image, run:

    $ sudo docker build -t api .

**Note: The period (.) after api is needed**.

Docker build image

Next we tag the image so we can push it to the repository:

    $ sudo docker tag api:latest [account-ID].dkr.ecr.[region].amazonaws.com/api:v1 

    Example  $ sudo docker tag api:latest 286396332837.dkr.ecr.us-east-1.amazonaws.com/api:v1 

Finally, we push the image to ECR by running:

    $ sudo docker push [account-ID].dkr.ecr.[region].amazonaws.com/api:v1 

.

    $ sudo docker push 286396332837.dkr.ecr.us-east-1.amazonaws.com/api:v1

Docker push image 

Go to your AWS ECR, click on the repo created and under images you should see your image tagged `v1`. 

![](./images/docker-push2.png)


## **Deploy the Monolith** 

 You are going to deploy the node.js application using Amazon Elastic Container Service.

## What is Amazon Elastic Container Service?

It is a highly scalable, high performance container management service that supports Docker containers and allows you to easily run applications on a managed cluster of Amazon Elastic Compute Cloud (EC2) instances. It eliminates the need to install, operate, and scale your own cluster management infrastructure.

### Step 1

You will need to first create an Amazon ECS cluster deployed behind an Application Load Balancer. 

Navigate to the AWS CloudFormation console

Select **Create stack**

You will be given two options, select **With New Resources (standard)**


On the **Specify template page** select **Upload a template file** and choose the `ecs.yml` file from the project at `amazon-ecs-nodejs-microservices/2-containerized/infrastructure/` folder then select **Next**

Note: leave the prerequisite - prepare template as is

![](./images/cf1.png)


For the stack name, enter *BreakTheMonolith-Demo*
Verify the other parameters have the following values:

- *Desired Capacity* = 2

- *Instance Type* = t2.micro

- *MaxSize* = 2

Select **Next**

On the **Configure stack options** page, keep the default settings, scroll to the bottom of the page and select **Next**

On the **Review BreakTheMonolith-Demo page,** scroll the bottom, check the **Capabilities** statement checkbox and select **Create stack**

![](./images/cf2.png)


Your stack will have the status **Create_IN_PROGRESS**. 

![](./images/cf3.png)


### Step 2 

***Check the Cluster is Running***

Navigate to the *Elastic Container Service* (ECS) console, there your cluster shoup appear in the list. 

![](./images/ecs1.png)


Select your cluster *BreakTheMonolith-Demo*, then select the **Tasks** tab  to verify that there are no tasks running.

Select the **ECS instance** tab, verify that there are two EC2 instances created by the CloudFormation template. 

![](./images/ecs2.png)

### Step 3

On the **ECS** left navigation menu, select **Task Definition**.

![](./images/ecs3.png)

Select **Create new Task Definition**.

On the **Select launch type compatibility** page, select the **EC2** option and then select **Next step**.

![](./images/ecs4.png)


On the **Configure task and container definition** page, enter the below options:

* *Task Definition Name* - enter api

* *On the Container Definitions* - select add container 

* *Container name* - enter api

* *Image field* -  [account-ID].dkr.ecr.[region].amazonaws.com/api:v1. Replace [account-ID] and [region] with your specific information. 

> 286396332837.dkr.ecr.us-east-1.amazonaws.com/api:v1

* *Memory Limits* -  Hard Limit 256 as the value 
* *Port mappings* -  Host port = 0, Container port = 3000
* *Environment* - CPU units = 256

Select **Add**, you will return to the previous page. 

Scroll to the bottom of the page and select **Create**

Your task definition will be listed in the console

![](./images/ecs5.png)



### Step 4

**Configuring the Application Load Balancer: Target Group**

What is Application Load Balancer?

The Application Load Balancer (ALB) is a feature of Elastic Load Balancer (ELB) that allows a developer to configure and route incoming end-user traffic to applications based in the AWS public cloud. It operates on the 7th layer of the *Open Systems Interconnection* (OSI) model. 

The first step is to configure your *Target Group* with the correct *VPC*. To do this, navigate to the **Load Balancer** Section of the **EC2** Console.

Locate the Load Balancer named **Break-ECSAL-1OOI3WXPP3KVV**.

Select the checkbox next to it to see the details.

In the **Description** tab, locate the **VPC** attribute. You will need it when configuring the ALB target group. 

![](./images/alb1.png)


**Configuring the ALB Target Group** 

Navigate to the **Target Group** section of the EC2 console

Select **Create target group**

Configure the following parameters, keep the default for anything not listed:

* *Target group name* - enter api
* *Protocol* - Select HTTP
* *Port* - enter 80
* *VPC* - Select the value that matches the one from the Load Balancer description

Access the Advanced Health check settings and edit the following parameters:

* *Healthy threshold* - enter 2
* *Unhealthy threshold* - enter 2
* *Timeout* - enter 5
* *Interval* - enter 6

Select **Next** 

Scroll to the bottom of the page and select **Create target group**

![](./images/target-group.png)


### Step 5 

**Configuring the ALB: Listener**

***Application Load Balancer listener***

A listener is one that checks for connection requests from clients using the protocol and port configured by an organization using rules that determine how ALB routes requests to registered targets. 

To add a listener to the ALB, navigate to the **Load Balancer** section of the EC2 console.

Select the checkbox next to **Break-ECSAL-1OOI3WXPP3KVV** to see the details.

Select the **Listeners** tab,  select **Add listener** and edit the following parameters:

* *Protocol:port* - select HTTP and enter 80
* *Default action(s)* - select Forward to 
* *Target group* - select api

Select **Add**.

![](./images/listener.png)


### Step 6 

***Deploying the Monolith as a Service***

Navigate to the **ECS*** console and select **Clusters** from the left menu bar.

Select the cluster **BreakTheMonolith-Demo**, select the **Services** tab then select **Create**.

On the **Configure service** page, edit the following parameters and keep the default values for every other thing. 

* *Launch type* - select EC2
* *Service name* -  enter api
* *Number of tasks* - enter 1

Select **Next step**


On the **Configure network** page, **Load Balancing** section, select **Application Load Balancer**, Additional parameters will appear: *Service IAM role* and *load balancer name*. 

* *Service IAM role* -  select  BreakTheMonolith-Demo-ECSServiceRole
* *Load balancer name* -  select your load balance name

In the **Container to load balance** section, select **Add to load balancer** and additional information labeled `api:3000` is shown.

In the `api:3000` section, complete the following:

* *Production listener port* -  select 80:HTTP
* *Target group name* - select your group - api

Select **Next step**

On the **Set Auto Scaling** page, leave the default setting and select **Next step**

On the **Review** page, review the settings then select **Create Service**

Once created, select **View Service**.

![](./images/cluster.png)


***Congratulations!!! You have a running service***. 


### Step 7 

***Testing the Monolith*** 

It is time to validate the deployment by checking if the service is available from the internet and pinging it. 

First navigate to the **Load Balancer** section of the EC2 console.

Select your **load balancer**.

In the **Description tab**, copy the *DNS* name and paste into your browser.

![](./images/lb1.png)

The message **Ready to receive requests** should be displayed. 

![](./images/lb2.png)


To see each part of the service (worker), simply add the worker name `“api/[worker-name]”` to the end of the *DNS* name.

>Example:
>
> http://[DNS-name]/api/users
> http://[DNS-name]/api/threads
>


![](./images/lb3.png)


You can also add a record number at the end of the URL to get a particular record. 

> Example: http://[DNS-name]/api/users/2


![](./images/lb4.png)


## Break the Monolith 

In this section, you will break the node.js application into several interconnected services and push each one into an Amazon Elastic Container Registry repository. 

To deploy the application as three microservices, you will need to provision three repositories in ECR. 

The three services are:
-  Users
- Threads
- Posts

### Step 1

Navigate to the **ECR** console

Select **Create repository**

In the **Create repository** page, for the **Repository name** field, create a repo for each service (i.e use the service as the name).

Leave the default setting for the **Tag immutability** option 

![](./images/service-repo.png)


The next step is building and pushing images which requires access to Docker. Log into Docker using the command in your terminal:

    $ sudo docker login -u AWS -p $(aws ecr get-login-password --region us-east-1) 286396332837.dkr.ecr.us-east-1.amazonaws.com

Next, navigate into the `3-microservices/services` directory:

    $ cd ~/amazon-ecs-nodejs-microservices/3-microservices/services

Now, to build the images run:

    $ sudo docker build -t [service name] ./[service-name]

![](./images/docker-build2.png)

Tag the image so it can be pushed to the repository. 

    $ sudo docker tag [service name]:latest [account-ID].dkr.ecr.[region].amazonaws.com/[service-name]:v1

>Example:
>$ sudo docker tag posts:latest 286396332837.dkr.ecr.us-east-1 amazonaws.com/posts:v1
>

Finally,  you push the image to ECR:

    $ sudo docker push [account-id].dkr.ecr.[region].amazonaws.com/[service-name]:v1

>Example: 
>$ sudo docker push 286396332837.dkr.ecr.us-east-1.amazonaws.com/threads:v1
>
>or
>
>$ sudo docker push 286396332837.dkr.ecr.us-east-1.amazonaws.com/posts:v1

![](./images/docker-push3.png)

***Note: You need to run the commands for all three services***. 

All your images tagged with *v1* can be found in your ECR repository. 





## Deploy Microservices

It is now time to deploy the node.js application as a set of interconnected services behind an Application Load Balancer.  We will also seamlessly shift traffic from the monolith to the microservices. 

### Step 1  

***Writing Task Definitions for each service***

You need to create task definitions for each microservice. You can create this from the ECS console or by writing a **JSON** file. 

Navigate to the ECS console, select **Task definition**

In the task definition page, select **Create new Task Definition** button

In the **Select launch type compatibility** page, select the **EC2** option and then select **Next step**.

In the **Configure task and container definition** page, scroll to **Volumes** section, select **Configure via JSON** button

Copy and paste the below code snippet into the JSON field. Replace the [service-name], [account-ID], [region] and tag placeholders with the corresponding values for each micro service. 

Note: The following parameters are used for the task definition:
* *Name* = [service-name: posts, threads, and users]
* *Image* = [Amazon ECR repository image URL]:latest
* *CPU* = 256
* *Memory* = 256
* *Container Port* = 3000
* *Host Port* = 0


        { 
            "containerDefinitions": [ 
                { 
                    "name": "[service-name]",
                    "image": "[account-id].dkr.ecr.[region].amazonaws.com/[service-name]:[tag]", 
                    "memoryReservation": "256", 
                    "cpu": "256",
                    "essential": true, 
                    "portMappings": [ 
                        { 
                            "hostPort": "0",
                            "containerPort": "3000",
                            "protocol": "tcp"
                        } 
                    ] 
                } 
            ], 
            "volumes": [],
            "networkMode": "bridge", 
            "placementConstraints": [], 
            "family": "[service-name]"
        }

Save and then select **Create**

![](./images/task-definition1.png)

![](./images/task-definition2.png)


### Step 2

***Configuring ALB target groups***

You need to create a target group for each service to allow traffic to correctly reach a specified service. We will need the *VPC* name for this project to proceed.

To access *VPC* name, navigate to the load balancer section of the EC2 console.

Select the checkbox next to your current project name, select the **Description** tab and locate the **VPC attribute**. 

To configure the attribute using the CLI in the terminal, enter the following command for each service:

Service  names: **posts**, **threads**, **users** and **drop-traffic**.

    $ aws elbv2 create-target-group --region [region] --name [service-name] --protocol HTTP --port 80 --vpc-id [vpc-attribute] --healthy-threshold-count 2 --unhealthy-threshold-count 2 --health-check-timeout-seconds 5 --health-check-interval-seconds 6

>Example:
>
>$ aws elbv2 create-target-group --region us-east-1 --name drop-traffic --protocol HTTP --port 80 --vpc-id vpc-0d17419a2e8809480 --healthy-threshold-count 2 --unhealthy-threshold-count 2 --health-check-timeout-seconds 5 --health-check-interval-seconds 6


![](./images/alb-targetgroup.png)

![](./images/alb-targetgroup2.png)



### Step 3

***Configuring ALB listener Rules*** 

Navigate to the **load balancer** section of the EC2 console.

Select the checkbox next to your current project name to see the load balancer details.

Select the **listeners** tab.

There should only be one listener listed in this tab. Now you need to edit the listener rules.

Select **view/edit** rules under the **Rules** column.

On the **Rules** page, select the *(+)* button and the option to **Insert Rule** is displayed, select the option.

Use the following rule template to insert the necessary rules which include one to maintain traffic to the monolith and one for each micro service:

* IF Path = /api/[service-name]* THEN Forward to [service-name]

>For example: IF Path = /api/posts* THEN Forward to posts


Insert the rules in the following order:

* api: /api* forwards to api

* users: /api/users* forwards to users

* threads: /api/threads* forwards to threads

* posts: /api/posts* forwards to posts

Select **Save** and return to the load balancer console.

![](./images/alb-rules.png)

### Step 4 

***Deploying the microservices***

Note: Repeat the steps for each of the three microservices

Navigate to the **ECS** console and select **Clusters**
 
Select the project cluster, select the **Services** tab and then click on **Create**.
On the **Configure service** page, edit the following parameters:

* *Launch type* -  EC2
* *Task Definition* - Select the **Enter a value** button to automatically select the highest revision value
* *Service name* -  Enter a service name (posts, threads, or users)
* *Number of task* - enter 1

Select **Next step**

On the **Configure network** page, load balancing section choose:

* *Load balancer type* - select *Application load balancer*
* *Service IAM role* - select *BreakTheMonolith-Demo-ECSServiceRole*
* *Load balancer name* - verify project balancer name

In the **Container to load balance** section, select the **Add to load balancer** button and edit the following:

* *Production listener port* - set to **80:HTTP**
* *Target group name* - select the appropriate group based on the service name 

Select **Next step**

On the **Set Auto Scaling** page, select **Next step**
Select **Create Service** on the **Review** page  and select **View Service**

Double check that all services are active and running before you proceed. 

## Deploy service image 

### Step 5

***Switch traffic to microservices***

Now that the microservices are running, we need to reroute  traffic from the monolith to the services.

Navigate to the **Load balancer** section of the EC2 console.

Select the checkbox next to the current project name to see the details.

Select the **Listeners** tab.

Under the **Rules** column, select **View/edit** rules.

On the **Rules** page, select the **minus (-)** button.

Delete the first rule `(/api* forwards to api)`. 

Update the default rule to forward to drop-traffic by:

Select the edit button from the top.

Select the edit icon by the default rule **(HTTP 80: default action)**.

Select the edit icon bt **THEN** column to edit the **Forward to**.

In the **Target group** field, select **drop-traffic**.

Select the **Update** button

![](./images/alb-rules2.png)


With traffic now flowing to the microservices, you need to disable the monolith service.

Navigate to the ECS cluster **BreakTheMonolith-Demo-ECSCluster**.

Under the **Service** tab, select the checkbox next to **api** and 
select **Update**.

On the **Configure service** page, ocate **Number of tasks** and enter **0**

Select **Skip to review**.

Select **Update service**.

The ECS will now stop the container. You can also delete the api service under the **Service** tab by clicking the checkbox and selecting **Delete**. 

### Step 6

***Validating the Deployment***. 

Navigate to the **Load balancer** section.

Select the current project.

In the **Description** tab, locate the **DNS** name.
 
Copy and paste the URL in a browser of your choice

You should see a message **‘Ready to receive requests’**


## Clean Up

You have completed your deployment and it is time to terminate the resources you created. You need to stop services running on ECS, delete the ALB and delete the cloudformation stack to terminate the ECS cluster and EC2 instances. 


### Step 1

***Turn off /Delete your services***

Go to the **ECS** console and select **Clusters**.

Select the **BreakTheMonolith-Demo cluster**.

In the **Service**  tab, select a service and select **Delete**.
 
Confirm deletion and then repeat the process for all the other services.

### Step 2 

***Delete Listeners and Target Groups***

Go to **Load balancer** section.

Select the project name and select the **Listeners** tab.

Select the **Listener** and then select **Delete**.

Confirm the deletion 

Next, click on the **Target Group**.

Check the checkbox next to the project name.

Select all target groups and select **Actions**.

Select **Delete** and confirm the deletion.

### Step 3

***Delete AWScloudformation stack***

Go to the **CloudFormation** console.

Select the checkbox next to the stack **BreakTheMonolith-Demo**.

Select **Action** and then Select **Delete Stack**.

Confirm the deletion.

Stack status should change to **DELETE_IN_PROGRESS**.


### Step 4 

***Deregister Task definitions***

 Navigate to **Task Definition** in the **ECR** console.

Select a task definition(api, posts, threads, or users).

On the **Task Definition Name** page, click the checkbox next to a task name.

Select **Actions** and select **Deregister**.

Confirm your action.

Repeat the process for all four task definitions.


### Step 5 

***Delete Amazon ECR Repository***

Go to the **Repositories** in the Amazon ECR console.

Select the checkbox next to a repo and select **Delete**.

Confirm the deletion.

Repeat the steps for all repositories. 


***CONGRATULATIONS!!! You have successfully created a monolith application and split it into containerized microservices using AWS***
