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
