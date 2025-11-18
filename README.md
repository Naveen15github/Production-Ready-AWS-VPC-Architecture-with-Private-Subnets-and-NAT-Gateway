# Production-Ready-AWS-VPC-Architecture-with-Private-Subnets-and-NAT-Gateway
## 🏗️ VPC Creation (Step-by-Step Guide)

This section explains how to create a custom VPC in AWS with public and private subnets, Internet Gateway, NAT Gateway, and proper route tables. This is the networking foundation for Auto Scaling, Load Balancing, and EC2 deployment.

🔹 1. Create a New VPC

Go to AWS Console → VPC

Click Create VPC

Select VPC Only / VPC and More

Enter:

Name: prod-vpc

IPv4 CIDR: 10.0.0.0/16

Click Create VPC

🔹 2. Create Subnets

You need Public Subnets (for Load Balancer, NAT) and Private Subnets (for EC2/ASG).

Create Public Subnets

Go to Subnets → Create Subnet

Select your VPC (prod-vpc)

Add:

Subnet Name: public-subnet-1

Availability Zone: ap-south-1a

CIDR: 10.0.1.0/24

Repeat for:

public-subnet-2 → ap-south-1b → 10.0.2.0/24

Create Private Subnets

Create two more subnets:

private-subnet-1 → ap-south-1a → 10.0.11.0/24

private-subnet-2 → ap-south-1b → 10.0.12.0/24

🔹 3. Create and Attach an Internet Gateway

Go to Internet Gateways → Create

Name: prod-igw

Click Create and Attach

Select prod-vpc → Attach

✔ Public subnets need Internet access, so IGW is required.

🔹 4. Create a NAT Gateway

Private subnets need outgoing internet to download packages. For this:

Go to NAT Gateways → Create

Subnet: public-subnet-1

Allocate Elastic IP

Name it: prod-nat-gateway

Click Create NAT Gateway

✔ NAT Gateway allows private EC2 instances to access the internet securely.

🔹 5. Create Route Tables
Public Route Table

Go to Route Tables → Create

Name: public-rt

Select prod-vpc

Add route:

Destination: 0.0.0.0/0

Target: Internet Gateway

Associate with:

public-subnet-1

public-subnet-2

Private Route Table

Create another route table → name private-rt

Add route:

Destination: 0.0.0.0/0

Target: NAT Gateway

Associate with:

private-subnet-1

private-subnet-2

![Alt Text](image-name.png)

![Alt Text](image-name.png)

![Alt Text](image-name.png)

## 🚀 Launch Template Creation (for Auto Scaling Group)

The Launch Template defines how your EC2 instances will be launched inside the Auto Scaling Group.
It includes the AMI, instance type, key pair, security groups, user data, and networking settings.

This ensures every new instance launched by the ASG follows the same configuration.

🔹 1. Go to Launch Templates

Open AWS Console → EC2

On the left side, select Launch Templates

Click Create launch template

🔹 2. Basic Details

Fill the template information:

Launch template name:
prod-launch-template

Template version description:
v1 - base configuration

🔹 3. Choose an Amazon Machine Image (AMI)

Select:

AMI: Amazon Linux 2
(or your custom application AMI)

This AMI defines the base OS that your Auto Scaling instances will run.

🔹 4. Choose Instance Type

Select a lightweight instance type for demos:

t2.micro (Free Tier eligible)

This ensures low cost while testing.

🔹 5. Key Pair (Optional for ASG)

Choose a key pair if you want SSH access:

naveen-keypair (Example)

If SSH access is not required, select "Proceed without key pair".

🔹 6. Network Settings
Security Group

Create or select a security group:

Name: prod-sg

Rules:

Allow HTTP (80) from anywhere → required for Load Balancer traffic

Allow SSH (22) from your IP → optional

Outbound → allow all

This ensures the instance can receive traffic only through the load balancer.

![Alt Text](image-name.png)
![Alt Text](image-name.png)

## 🔎 Checking EC2 Instance Launch Source (Launch Template or Not)
This guide outlines the steps to programmatically or manually check if an Amazon EC2 instance was launched using a specified Launch Template within your AWS VPC environment, which utilizes private subnets and a NAT Gateway.

I. The Core Method: Checking Instance Metadata
When an EC2 instance is launched using a Launch Template, the ID of that template is often included in the instance's metadata. This is the most reliable way to verify the launch source.

A. Using the AWS Management Console (Manual Check)
Navigate to the EC2 Dashboard in the AWS Management Console.

In the navigation panel, choose Instances.

Select the specific EC2 instance you want to check.

In the details pane below, click on the Details tab.

Scroll down to the Launch Details section.

Look for a field labeled Launch template ID.

If a Launch Template was used, the ID (e.g., lt-0abcdef1234567890) will be listed.

If the instance was launched without a Launch Template (e.g., directly from an AMI or using the old Launch Wizard), this field will likely be blank or hyphenated.

![Alt Text](image-name.png)

