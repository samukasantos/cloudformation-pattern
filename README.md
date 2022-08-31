# Servian DevOps - Deploying Microservices with Amazon Fargate, AWS CloudFormation and an Application Load Balancer.


This reference architecture provides a set of YAML templates for deploying microservices using cloudformation.

![architecture](https://user-images.githubusercontent.com/5481198/187695658-e9a0862b-2b39-4f7f-afc9-4664fd2064a7.png)

The repository features a set of templates that implement the following:

- A VPC with public and private subnets, spanning an AWS Region.
- A highly available ECS cluster deployed across two Availability Zones in an Auto Scaling group.
- A pair of NAT gateways (one in each zone) to handle outbound traffic.
- A microservice referring to the TechChallengeApp application.
- An Application Load Balancer (ALB) for the public subnets to handle incoming traffic.
- ALB path-based routes for each ECS Fargate task to route incoming traffic to the correct service.
- Centralized container logging with Amazon CloudWatch Logs.
- Database instances (PostgreSQL) in different Availability Zones.
- Auto Scaling with alarm to validate requests submitted to applications.
- Roles associated with running tasks on the Fargate, AutoScaling, and OIDC cluster.


