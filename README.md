# Servian DevOps - Deploying Microservices with Amazon Fargate, AWS CloudFormation and an Application Load Balancer.


This reference architecture provides a set of YAML templates for deploying microservices using cloudformation.

![architecture](https://user-images.githubusercontent.com/5481198/187695658-e9a0862b-2b39-4f7f-afc9-4664fd2064a7.png)

The repository features a set of templates that implement the following:

- A VPC with public and private subnets, spanning an AWS Region.
- A highly available ECS Fargate cluster deployed across two Availability Zones in an Auto Scaling group.
- A pair of NAT gateways (one in each zone) to handle outbound traffic.
- A microservice referring to the TechChallengeApp application.
- An Application Load Balancer (ALB) for the public subnets to handle incoming traffic.
- ALB route-based routes for each ECS Fargate task to route incoming traffic to the correct service.
- Centralized container logging with Amazon CloudWatch Logs.
- Database instances (PostgreSQL) in different Availability Zones.
- Auto Scaling with alarm to validate requests submitted to applications.
- Roles associated with running tasks on the Fargate, AutoScaling and OIDC.


## Why Cloudformation ?

Using CloudFormation to deploy and manage services on AWS brings more benefits than traditional methods like using the AWS CLI, scripting or even using the AWS console.

## Template Details

| Path | Name | Description |
| --- | --- |  --- |
| /src/templates/cloudformation/vpc/vpc-base.yaml | vpc base | This template deploys a VPC base with an Internet gateway and VPC Flow logs. |
| /src/templates/cloudformation/vpc/vpc-az-public.yaml |  vpc public | This template deploys public subnets to Availability Zone. |
| /src/templates/cloudformation/vpc/vpc-az-private.yaml | vpc private | This template deploys private subnets to Availability Zones. It deploys NAT gateway and default routes in the private subnets |
| /src/templates/cloudformation/elb/alb.yaml | load balancer | This template deploys an ALB to the public subnets with security groups associated, certificate and listeneres. |
| /src/templates/cloudformation/cluster/fargate/cluster.yaml | ecs cluster | This template deploys an ECS cluster. |
| /src/templates/cloudformation/cluster/fargate/ecs.yaml | ecs service | This template deploys an ECS service with taskdefinition to the private subnets, security group and it is responsible to inject the database credentials into Secrets Manager which will be available in the task via environment variables. |
| /src/templates/cloudformation/ecr/ecr.yaml | ecr | This template deploys an ECR repository for the images related to the current project. |
| /src/templates/cloudformation/autoscaling/autoscaler.yaml| Auto scaling | This template deploys Auto Scaling with policies and alarm. |
| /src/templates/cloudformation/rds/postgresql.yaml | rds | This template deploys a Postgres database to multiple AZs with the securit group associated. |
| /src/templates/cloudformation/roles/github-action-role.yaml| GitHub role | This template deploys a role used by this repository to deploy in the AWS provider, the account used was created using AWS Organization with AWS SSO and this is the OIDC Github provider. |
| /src/templates/cloudformation/roles/fargate-cluster-app-task-role.yaml| Farget task role | This template deploys an additional role for the Fargate to make possible the tasks communicate with AWS Apis. |

## The stack output is presented by: 

<img width="1342" alt="Screen Shot 2022-09-01 at 2 03 23 am" src="https://user-images.githubusercontent.com/5481198/187725832-35e757c7-5215-4eca-b363-92f7e3c89993.png">

## CI/CD Workflow

The build and deployment automation is done through GitHub Actions, in the following steps:
  - Login to AWS using STS Credentials.
- Build and add the resulting image in ECR (Elastic Container Registry)
- Deploys the ECS Fargate cluster with the changes inferred in the current commit, the template is stored in an S3 bucket to which only the repository has access through a trust relationship.
- It is important to remember that an S3 bucket contains a file that represents the environment variables that are consequently embedded in the container during deployment.

