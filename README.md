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

Using CloudFormation to deploy and manage services on AWS brings more benefits than traditional methods like using the AWS CLI, scripting or even using the AWS console. These CloudFormation Templates are designed to simplify the creation of AWS resources and create complex environments with low effort.


# Table of Content
- [Why Cloudformation ?](#why-cloudformation-?)
- [Table of Content](#table-of-content)
- [Template Details](#template-details)
- [Feature Summary](#feature-summary)
- [Getting started](#getting-started)
  - [Requirements](#requirements)
  - [Installation](#installation)
- [Quickstart](#quickstart)
- [About](#about)

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

## Feature Summary

- VPC creation (vpc-base.yaml)
- Private Subnet with NAT Gateway(vpc-az-private.yaml)
- Public Subnet with Internet Gateway (vpc-az-public.yaml)
- ECS Cluster (cluster.yaml)
- ECR for store images (ecr.yaml)
- ALB setup with granular settings (certificate and listeners) (alb.yaml)
- ECS Service with service and task definitions (ecs.yaml)

# Getting started

## Requirements

In order to use this framework, you will need:

- The AWS CLI
- AWS CLI access (Access Key and Secret Access Key)
- An AWS S3 bucket to upload the CloudFormation Templates to
- Optional: AWS console management access (if you don't want to use the AWS CLI to create/manage required resources)

**AWS CLI**

Depending on the operating system / platform you are using, there are different ways to install the AWS CLI.
* Windows: [Installing, updating, and uninstalling the AWS CLI version 2 on Windows](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-windows.html)
* MacOS: Other than described in the official documentation by Amazon, we recommend to install the AWS CLI using [Homebrew](https://brew.sh/)
    ```
    $ brew install awscli
    ```
* Linux: [Installing, updating, and uninstalling the AWS CLI version 2 on Linux](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html)
* Docker: [Using the official AWS CLI version 2 Docker image](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-docker.html)

**AWS CLI access (Access and Secret Access Key)**

The Access and the Secret Key are personalized and need to be generated. For further information, please consult the AWS documentation on [Understanding and getting your AWS credentials](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html#access-keys-and-secret-access-keys).

**Amazon S3 bucket**

CloudFormation requires the templates to be stored in an Amazon S3 bucket. A bucket can be easily created using the AWS CLI.
```
$ aws s3 mb s3://<your-project>-templates --profile <your-profile>
```

## Installation

**AWS CLI Named Profiles**

To ensure that changes are only applied to a selected account and region, we recommend using the AWS CLI with [named profiles](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-profiles.html)
```
$ aws configure --profile <your-profile>
```

Finally test your setup, you should get a print out of all CloudFormation Stacks in the region you chose
```
$ aws cloudformation --profile <your-profile> list-stacks
```

# Quickstart

1. Create a S3 bucket
   ```
   $ aws s3 mb s3://<your-project>-templates --profile <your-profile>
   ```

2. Create a copy of the example configuration `vpc-base.yaml` and change it as needed. When done, package and upload it
    ```
    $ aws cloudformation package --profile <your-profile> --template-file <your-template>.yaml --s3-bucket <your-project>-templates --output-template-file <your-template>.yaml
    ```

3. Finally create a stack, using the packaged template
    ```
    $ aws cloudformation deploy --profile <your-profile> --template-file <your-template>.yaml --stack-name <your-project>
    ```

3. The stack output is presented by: 

<img width="1342" alt="Screen Shot 2022-09-01 at 5 29 34 am" src="https://user-images.githubusercontent.com/5481198/187766137-d866cd69-135a-4cb6-a82f-da88fd896df5.png">


# About

## Development guidelines

1. Before adding new logic, check in the code if a similar logic already exists
2. When adding new logic, do it bottom up. For example when adding something to the template `resources/ec2/instance.yaml`, add the new logic there and then recursively add the required parameters to all templates will end up using it, until you map them in the `example.yaml` file.
3. As a piece of advice, try not to hard code!
4. Enjoy!



## CI/CD Workflow

The build and deployment automation is done through GitHub Actions, in the following steps:
  - Login to AWS using STS Credentials.
- Build and add the resulting image in ECR (Elastic Container Registry)
- Deploys the ECS Fargate cluster with the changes inferred in the current commit, the template is stored in an S3 bucket to which only the repository has access through a trust relationship.
- It is important to remember that an S3 bucket contains a file that represents the environment variables that are consequently embedded in the container during deployment.

## Application

Used a domain for deployment of the application, domain registered in Route53 and routing to the Load Balancer of the present solution.

<img width="1324" alt="Screen Shot 2022-09-01 at 5 31 36 am" src="https://user-images.githubusercontent.com/5481198/187766755-665e9cf4-50dd-456c-9425-7ceab015e221.png">




