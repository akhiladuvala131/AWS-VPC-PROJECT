
# VPC with servers in private subnets and NAT



This document provides a detailed guide to creating a Virtual Private Cloud (VPC) suitable for a production environment. The setup ensures high availability, resiliency, and security by leveraging AWS resources like Auto Scaling groups, Application Load Balancers (ALBs), private subnets, and NAT Gateways across multiple Availability Zones (AZs).


## Prerequisites
Before starting, ensure you have the following:

AWS Account: Access to an AWS account with appropriate permissions to create VPCs, subnets, NAT Gateways, ALBs, and EC2 instances.

Basic Networking Knowledge: Familiarity with CIDR blocks, subnets, and routing.

IAM Role and Policies:

An IAM role with necessary permissions to manage VPC, EC2, Auto Scaling, and Load Balancers.

AWS-managed policies such as AmazonVPCFullAccess and EC2FullAccess.

AWS CLI or Management Console: Ensure you have the AWS CLI installed and configured, or access to the AWS Management Console.

Elastic IPs: At least two Elastic IPs available for NAT Gateways.

Application Image: A pre-configured AMI (Amazon Machine Image) for your application servers.

Key Pair: An existing or newly created key pair for SSH access to EC2 instances.

Estimated Costs: Understand the potential costs associated with the resources being deployed, such as NAT Gateways, ALBs, and EC2 instances.
## Architecture Overview
The architecture includes:

VPC: A custom VPC with CIDR block 10.0.0.0/16.

Subnets:

Two public subnets for the ALB and NAT Gateways (one in each AZ).

Two private subnets for the application servers (one in each AZ).

NAT Gateways: Deployed in both AZs to allow private subnets to access the internet.

Application Load Balancer (ALB): Distributes traffic to application servers across AZs.

Auto Scaling Group: Ensures scalability and availability of the application servers.

Route Tables:

Public route table for public subnets.

Private route table for private subnets.
## Step-by-Step Instructions
1. Create a VPC

Log in to the AWS Management Console.

Navigate to the VPC Dashboard.

Click Create VPC and provide the following details:

Name tag: Production-VPC

IPv4 CIDR block: 10.0.0.0/16

Tenancy: Default

Click Create VPC.

2. Create Subnets

Public Subnets

Go to Subnets in the VPC Dashboard.

Create two public subnets:

Subnet 1 (Public):

Name tag: Public-Subnet-1

VPC: Production-VPC

Availability Zone: us-east-1a

CIDR block: 10.0.1.0/24

Subnet 2 (Public):

Name tag: Public-Subnet-2

VPC: Production-VPC

Availability Zone: us-east-1b

CIDR block: 10.0.2.0/24

Private Subnets

Create two private subnets:

Subnet 1 (Private):

Name tag: Private-Subnet-1

VPC: Production-VPC

Availability Zone: us-east-1a

CIDR block: 10.0.3.0/24

Subnet 2 (Private):

Name tag: Private-Subnet-2

VPC: Production-VPC

Availability Zone: us-east-1b

CIDR block: 10.0.4.0/24

3. Create an Internet Gateway (IGW)

Navigate to Internet Gateways in the VPC Dashboard.

Click Create Internet Gateway and provide the name tag: Production-IGW.

Attach the IGW to the Production-VPC.

4. Create NAT Gateways

Navigate to NAT Gateways in the VPC Dashboard.

Create two NAT Gateways:

NAT Gateway 1:

Name tag: NAT-Gateway-1

Subnet: Public-Subnet-1

Elastic IP: Allocate a new Elastic IP.

NAT Gateway 2:

Name tag: NAT-Gateway-2

Subnet: Public-Subnet-2

Elastic IP: Allocate a new Elastic IP.

5. Configure Route Tables

Public Route Table

Navigate to Route Tables in the VPC Dashboard.

Create a new route table:

Name tag: Public-Route-Table

VPC: Production-VPC

Add a route:

Destination: 0.0.0.0/0

Target: Production-IGW

Associate this route table with Public-Subnet-1 and Public-Subnet-2.

Private Route Table

Create a new route table:

Name tag: Private-Route-Table

VPC: Production-VPC

Add routes:

Route 1:

Destination: 0.0.0.0/0

Target: NAT-Gateway-1 (for Private-Subnet-1)

Route 2:

Destination: 0.0.0.0/0

Target: NAT-Gateway-2 (for Private-Subnet-2)

Associate this route table with Private-Subnet-1 and Private-Subnet-2.

6. Launch an Application Load Balancer (ALB)

Navigate to EC2 Dashboard > Load Balancers.

Click Create Load Balancer > Application Load Balancer.

Provide the following details:

Name: Production-ALB

Scheme: Internet-facing

Listeners: HTTP (Port 80)

Availability Zones:

Public-Subnet-1

Public-Subnet-2

Configure the security group to allow HTTP (port 80) traffic.

Click Create Load Balancer.

7. Create an Auto Scaling Group

Navigate to EC2 Dashboard > Auto Scaling Groups.

Click Create Auto Scaling Group.

Configure the following:

Name: Production-ASG

Launch Template or Configuration:

Use a pre-created launch template that specifies the AMI, instance type, and key pair.

Network:

VPC: Production-VPC

Subnets: Private-Subnet-1 and Private-Subnet-2

Configure scaling policies and health checks.

Attach the Auto Scaling group to the Production-ALB.

8. Test the Setup

Deploy a sample application to the instances in the Auto Scaling group.

Access the application via the ALB DNS name.

Verify that the application is reachable and traffic is distributed across instances.
## Terraform Files (terraform/)
This directory contains the Terraform files used to define the infrastructure.

main.tf ,variables.tf ,outputs.tf ,provider.tf

Contains the core infrastructure definitions (VPC, subnets, NAT Gateways, ALB, Auto Scaling, etc.).
## CloudFormation Template (cloudformation/)
This directory contains a CloudFormation YAML template to set up the VPC and related resources.

A CloudFormation YAML template for setting up the VPC, subnets, NAT Gateways, route tables, and other resources.
## parameters.json (optional)
A JSON file for parameterized input if you want to use CloudFormation stack parameters.

Example:

{
  "VpcCIDR": "10.0.0.0/16",
  
  "Region": "us-east-1"
}
## Best Practices
Security Groups:

Restrict inbound traffic to only necessary ports.

Ensure the instances in private subnets allow only required outbound traffic.

Monitoring and Logging:

Enable CloudWatch metrics and alarms for the ALB, Auto Scaling group, and NAT Gateways.

Use AWS CloudTrail for auditing.

High Availability:

Deploy resources across multiple AZs.

Use health checks for Auto Scaling and ALB to ensure resiliency.


## License
This project is licensed under the MIT License. You are free to use, modify, and distribute this setup, provided proper attribution is given to the original authors.


## Conclusion
This setup ensures a secure, highly available, and scalable environment for production workloads. By leveraging AWS services such as Auto Scaling, ALB, NAT Gateways, and private subnets, this architecture is designed to handle traffic efficiently while maintaining strong security and fault tolerance.
## References
aws.amazon.com:Related Resources

VPC Peering Guide:-https://docs.aws.amazon.com/vpc/latest/peering/
Amazon EC2 Developer Guide:-
https://docs.aws.amazon.com/ec2/latest/devguide/
Amazon VPC Transit Gateways:-
https://docs.aws.amazon.com/vpc/latest/tgw/
