# Developing-a-Three-Tier-Architecture-using-Docker-Containers-Swarm
Scaling Your Web Application with Docker Swarm on AWS

Project Scope

Suppose your business is building a web application that requires a highly available and scalable infrastructure.

To achieve this goal effectively, your team decides to leverage Docker Swarm to manage their containerized services in a more efficient, highly available and automated way.

Using AWS, the DevOps engineer can create a Docker Swarm that consists of one manager and multiple worker nodes. The manager node acts as the central control point for the cluster, while the worker nodes will execute the containers that run the application services.

To provide maximum availability and scalable services, the team decides to deploys a three-tier architecture that consists of three container services:

Apache (Web-Tier)

Redis (Application-Tier)

Postgres (Database-Tier)

With this architecture, the team can ensure that the application has the necessary resources to handle user requests and that the infrastructure can automatically scale up or down based on demand. Furthermore, by using Docker Swarm, the team can easily manage the containerized services, enabling them to focus on building the application’s core functionality and features.

In this article, I will walk you thru step-by-step how to create this 3-tiered architecture using docker swarm.

Pre-requisites

An AWS account

Knowledge of containerization and Docker

AWS cloud knowledge: Basic knowledge of AWS cloud services, including EC2 instances, VPCs, and security groups, and how to launch and manage them.

Networking knowledge: Understanding networking concepts such as IP addressing, subnets, and routing, and how to configure networking for containerized applications.

Phase 1— Creating our VPC, Subnets, & Security Groups

First, we start by creating our VPC and subnets. We will create each subnet in a different availability zone per our high availability requirements:

<img width="1121" alt="Screenshot 2023-04-02 at 16 57 47" src="https://user-images.githubusercontent.com/67044030/229369383-c8d5ab73-6eae-4675-9a18-33fdf420edee.png">

<img width="1185" alt="Screenshot 2023-04-02 at 16 58 09" src="https://user-images.githubusercontent.com/67044030/229369415-03b63776-9265-4644-828b-0b177666b07c.png">

<img width="975" alt="Screenshot 2023-04-02 at 17 08 27" src="https://user-images.githubusercontent.com/67044030/229369466-2465e0cd-255c-47f0-b022-d5ce4d49505e.png">

Next I need to create two different security groups. One for our Manager node and one for our worker nodes.

These security group rules will allow communication between the manager node and the worker nodes on the necessary ports, while restricting access from other sources. By applying these rules, I can ensure that the Docker Swarm cluster is configured securely and can communicate smoothly between the instances, while preventing unauthorized access or attacks from outside the cluster.

Here are the ingress and egress rules for my manager node security group:


Here are the ingress and egress rules for my worker node security group, which allows communication between the worker nodes and the manager node. Additionally, it allows our worker nodes outbound access to the public internet traffic and SSH access in order for us to update our packages and swarm:

<img width="994" alt="Screenshot 2023-04-02 at 17 19 01" src="https://user-images.githubusercontent.com/67044030/229369552-4ef7610c-45a4-4adc-b1f3-229aa707ef9e.png">


Phase 2 — Launching our Instances for Swarm Nodes

From my EC2 Console, I am going to launch four instances, one for the manager node and three for the worker nodes.

<img width="705" alt="Screenshot 2023-04-02 at 17 23 31" src="https://user-images.githubusercontent.com/67044030/229369657-27aec1fd-1e37-44c0-8bed-e71237bbf552.png">

<img width="726" alt="Screenshot 2023-04-02 at 17 23 41" src="https://user-images.githubusercontent.com/67044030/229369745-abedfaae-7192-4f41-9468-c3eef3442ddf.png">

Here are my details for launching my manager node instance:

<img width="685" alt="Screenshot 2023-04-02 at 17 28 59" src="https://user-images.githubusercontent.com/67044030/229369800-92044b27-22de-4cc5-8a4a-5d6916f7144c.png">


Be careful to select the proper VPC, subnet, and swarm-manager-security group

Here are my details for launching my Apache node instance for my web-tier container environment:


Here is my configuration for launching my Redis node instance for my app-tier container environment:


Here is my configuration for launching my Redis node instance for my app-tier container environment:


We have successfully launched our instances for our three-tier web application.


Phase 3 — Install and Initialize Docker Swarm

Next, I need to SSH into our manager node instance, update our packages, and install docker.

Since I installed Amazon Linux 2 as our OS on our instances so we will run the following commands to install and initialize Docker:

sudo yum update -y # Updates the instance

sudo amazon-linux-extras install docker -y # Installs Docker

<img width="717" alt="Screenshot 2023-04-02 at 17 46 57" src="https://user-images.githubusercontent.com/67044030/229370027-33995e4f-6317-443d-acf0-2eb7b0b756c0.png">

<img width="681" alt="Screenshot 2023-04-02 at 17 52 56" src="https://user-images.githubusercontent.com/67044030/229370134-2d822580-622f-4814-b5ec-4a7d28739e49.png">

sudo service docker start # Starts the Docker service

<img width="724" alt="Screenshot 2023-04-02 at 17 29 08" src="https://user-images.githubusercontent.com/67044030/229369855-321d3582-9e40-42f9-accd-e581cf0cd75b.png">


<img width="713" alt="Screenshot 2023-04-02 at 17 29 53" src="https://user-images.githubusercontent.com/67044030/229369899-bc73f1d7-aad6-4f44-bb9d-61d6eaa3139a.png">

sudo service docker enable # Ensures Docker Service Starts upon instance reboot

<img width="722" alt="Screenshot 2023-04-02 at 17 53 31" src="https://user-images.githubusercontent.com/67044030/229370365-5ea003aa-3d5f-45ba-8e46-55dc69edb0e1.png">


Next, I will initialize Docker Swarm by running the following command and adding the IP address for my manager node:

sudo docker swarm init --advertise-addr 172.31.19.53

<img width="726" alt="Screenshot 2023-04-02 at 17 37 14" src="https://user-images.githubusercontent.com/67044030/229369975-823c5838-b53e-4e9b-a818-b232eb0cd233.png">

Notice that we receive back some information that we are going to need to need when we add our workers to our swarm:


We will retain the value output from our “docker swarm init” command to use for adding our worker nodes to our swarm in the next step.


sudo docker swarm join --token SWMTKN-1-2tjq3yi3sxo16rlt3cc4231pplpm1rc75pqrgyisjfzeba4xmc-8xnqf3i8kduolqofz6tpahcia 172.31.19.53:2377

Phase 4 — Join Worker nodes to the Docker Swarm

Now that I have created my swarm and manager node, Ineed to configure my worker nodes to join.

First, we SSH into each one of our worker node instances. We will run the same commands to install and initialize Docker and update the packages similar to setting up our manager node, your values may be different:

For our Apache node:

sudo yum update -y # Updates the instance
sudo amazon-linux-extras install docker -y # Installs Docker
sudo service docker enable # Ensures Docker Service Starts upon instance reboot
sudo service docker start # Starts the Docker service

sudo docker swarm join --token SWMTKN-1-2tjq3yi3sxo16rlt3cc4231pplpm1rc75pqrgyisjfzeba4xmc-8xnqf3i8kduolqofz6tpahcia 172.31.19.53:2377

My Apache node was successfully added to my swarm

<img width="728" alt="Screenshot 2023-04-02 at 17 57 21" src="https://user-images.githubusercontent.com/67044030/229370462-cfccba50-6478-42f6-ab6a-2ec45a5bdd42.png">

For our Redis caching node:

sudo yum update -y # Updates the instance
sudo amazon-linux-extras install docker -y # Installs Docker
sudo service docker enable # Ensures Docker Service Starts upon instance reboot
sudo service docker start # Starts the Docker service

sudo docker swarm join --token SWMTKN-1-2tjq3yi3sxo16rlt3cc4231pplpm1rc75pqrgyisjfzeba4xmc-8xnqf3i8kduolqofz6tpahcia 172.31.19.53:2377

My Redis node was successfully added to my swarm

<img width="733" alt="Screenshot 2023-04-02 at 17 57 36" src="https://user-images.githubusercontent.com/67044030/229370552-5520e4ee-01dc-4728-a98a-366a73da77ca.png">

Finally, for our Postgres Database-Node

sudo yum update -y # Updates the instance
sudo amazon-linux-extras install docker -y # Installs Docker
sudo service docker enable # Ensures Docker Service Starts upon instance reboot
sudo service docker start # Starts the Docker service

sudo docker swarm join --token SWMTKN-1-2tjq3yi3sxo16rlt3cc4231pplpm1rc75pqrgyisjfzeba4xmc-8xnqf3i8kduolqofz6tpahcia 172.31.19.53:2377

My Postgres node was successfully added to my swarm

<img width="732" alt="Screenshot 2023-04-02 at 17 59 11" src="https://user-images.githubusercontent.com/67044030/229370628-63412798-9c4d-40a8-bcf9-43a5bddcb783.png">

The last step in this phase is to verify that all of my worker nodes have joined my swarm. Back on my manager node, I run the command docker node ls to see a list of all the nodes in the Swarm cluster, including the worker node(s) that just joined.

sudo docker node ls

<img width="728" alt="Screenshot 2023-04-02 at 17 59 52" src="https://user-images.githubusercontent.com/67044030/229370682-8928db2c-0d10-4cb2-82e6-d03f04ba391a.png">

My swarm is configured correctly.

Phase 5— Creating Docker services for Apache, Redis, and Postgres

Next, I need to pull down the official docker image files for what will be our Apache, Redis, and Postgres containers:

sudo docker pull httpd:latest # Pulls the latest official Apache image from Docker

<img width="726" alt="Screenshot 2023-04-02 at 18 03 58" src="https://user-images.githubusercontent.com/67044030/229370723-3f4ebf62-fccc-47fc-9abc-545c3cfd0378.png">

sudo docker pull redis:latest # Pulls the latest official Redis image from Docker

<img width="737" alt="Screenshot 2023-04-02 at 18 07 50" src="https://user-images.githubusercontent.com/67044030/229370836-c39a9c67-3ae9-4494-9bb3-7d5bbd321592.png">

sudo docker pull postgres:latest # Pulls the latest official Redis image from Docker

<img width="727" alt="Screenshot 2023-04-02 at 18 07 32" src="https://user-images.githubusercontent.com/67044030/229370758-49721ce7-790d-4193-9bab-37153527e659.png">

You will need to find the hostname for each of your instances in order to launch your services. You can find the host name by SSHing into each of your instances and running the “hostname” command.

Now I can start deploying my tiered environments to my containers starting with my Apache services.

sudo docker service create --name web_server --replicas 10 --constraint node.hostname==ip-172-31-3-84.us-east-2.compute.internal httpd

Our Web Server Services were deployed correctly

Next, I will deploy my Postgres services for our database-tier. Similar to launching our Apache services, we run a similar command from our manager node.

For our Postgres services, we need to set a password to keep the command from failing. I’ve set a temporary password that I will go in and change.

sudo docker service create --name database --replicas 1 --constraint node.hostname==ip-172-31-3-225.us-east-2.compute.internal --env POSTGRES_PASSWORD=password postgres

<img width="744" alt="Screenshot 2023-04-02 at 18 15 31" src="https://user-images.githubusercontent.com/67044030/229370922-7dbe0663-243c-4ddb-aaaf-80d00d5eea1e.png">

My Postgres database and replica was launched correctly

Lastly, I will create a Redis service for our application layer of our 3-tiered architecture by once again SSH’ing into my ‘cache’ worker node, in order to deploy four docker services each running a Redis replica

sudo docker service create --name caching_DB --replicas 4 --constraint node.hostname==ip-172-31-15-152.us-east-2.compute.internal redis:6

<img width="737" alt="Screenshot 2023-04-02 at 18 15 44" src="https://user-images.githubusercontent.com/67044030/229371021-335ba9bb-16b7-4dc7-b85e-43e0486a2831.png">

My Redis caching database and replicas were launched correctly


<img width="907" alt="Screenshot 2023-04-02 at 18 26 45" src="https://user-images.githubusercontent.com/67044030/229371088-af8b3f07-3328-44ca-b34a-d5065e3180cf.png">

Phase 6- Verify our Docker Services

To ensure that our docker swarm has been configured to spec, I run ‘docker service ls’ , in order to verify that all services have been created and are in the correct state. I run this command from within my manager node instance:


<img width="732" alt="Screenshot 2023-04-02 at 18 25 06" src="https://user-images.githubusercontent.com/67044030/229371127-25c1a83d-ca77-496a-8dfd-a724b4079d4a.png">

sudo docker service ps caching_DB web_server database #To View all of our docker running services






