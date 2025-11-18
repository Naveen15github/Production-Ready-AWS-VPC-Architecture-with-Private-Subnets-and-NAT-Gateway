# Production-Ready-AWS-VPC-Architecture-with-Private-Subnets-and-NAT-Gateway
## 🏗️ VPC Creation (Step-by-Step Guide)

## Architecture Diagram

![Alt Text](https://github.com/Naveen15github/Production-Ready-AWS-VPC-Architecture-with-Private-Subnets-and-NAT-Gateway/blob/4e0ecce9d124b36fcc3e07a9cbd101b8c90c9992/vpc-example-private-subnets.png)


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

![Alt Text](https://github.com/Naveen15github/Production-Ready-AWS-VPC-Architecture-with-Private-Subnets-and-NAT-Gateway/blob/4e0ecce9d124b36fcc3e07a9cbd101b8c90c9992/Screenshot%20(124).png)

![Alt Text](https://github.com/Naveen15github/Production-Ready-AWS-VPC-Architecture-with-Private-Subnets-and-NAT-Gateway/blob/4e0ecce9d124b36fcc3e07a9cbd101b8c90c9992/Screenshot%20(125).png)

![Alt Text](https://github.com/Naveen15github/Production-Ready-AWS-VPC-Architecture-with-Private-Subnets-and-NAT-Gateway/blob/4e0ecce9d124b36fcc3e07a9cbd101b8c90c9992/Screenshot%20(126).png)

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

![Alt Text](https://github.com/Naveen15github/Production-Ready-AWS-VPC-Architecture-with-Private-Subnets-and-NAT-Gateway/blob/4e0ecce9d124b36fcc3e07a9cbd101b8c90c9992/Screenshot%20(127).png)
![Alt Text](https://github.com/Naveen15github/Production-Ready-AWS-VPC-Architecture-with-Private-Subnets-and-NAT-Gateway/blob/4e0ecce9d124b36fcc3e07a9cbd101b8c90c9992/Screenshot%20(128).png)

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

## 🔐 Bastion Host (Jump Server) Creation

A Bastion Host is a secure EC2 instance placed in a public subnet that allows you to SSH into private subnet EC2 instances.
Since private EC2s do not have public IPs, the bastion host acts as the gateway for administrative access.

This improves security by preventing direct public access to backend servers.

1️⃣ Purpose of the Bastion Host

Provides SSH access to private EC2 instances

Lives in a public subnet

Has a public IP

Only accessible from your IP address

Uses the same key pair used for instances in private subnets

🏗️ 2️⃣ Steps to Create the Bastion Host
🔹 Step 1: Launch a New EC2 Instance

Go to AWS Console → EC2

Click Launch Instance

Enter:

Name: bastion-host

AMI: Amazon Linux 2

Instance Type: t2.micro

This instance will be used to SSH into private servers.

🔹 Step 2: Key Pair

Select an existing key pair or create a new one:

Key Pair Name: prod-keypair

This key will also be used later to access private EC2 instances.

🔹 Step 3: Network Settings
✔ Choose VPC

Select your VPC:
prod-vpc

✔ Choose the Public Subnet

Select:
public-subnet-1 (example: ap-south-1a)

✔ Enable Public IP

Enable Auto-assign Public IP
(required for SSH from your laptop)

🔹 Step 4: Configure Security Group

Create a new security group for the bastion:

Security Group Name:

bastion-sg

Inbound Rules
Type	Port	Source	Purpose
SSH	22	Your IP (recommended)	Secure SSH access

👉 Do NOT allow SSH from “Anywhere (0.0.0.0/0)”
Use My IP for maximum security.

Outbound Rules

Allow All traffic (default)

🔹 Step 5: Storage

Use default (8GB–10GB).

🔹 Step 6: Launch the Bastion Host

Click Launch Instance and wait for the instance to reach running state.

🔌 3️⃣ Connect to the Bastion Host

Open your terminal and run:

ssh -i prod-keypair.pem ec2-user@<Bastion-Public-IP>


Once logged in, you can SSH into private servers using:

ssh -i prod-keypair.pem ec2-user@<Private-EC2-Private-IP>

🔐 4️⃣ Security Best Practices

Allow SSH only from your IP

Never install applications on the bastion host

Use it only for accessing private EC2 servers

Rotate key pairs periodically

Deny all inbound traffic except SSH

If the instance was launched without a Launch Template (e.g., directly from an AMI or using the old Launch Wizard), this field will likely be blank or hyphenated.


![Alt Text](https://github.com/Naveen15github/Production-Ready-AWS-VPC-Architecture-with-Private-Subnets-and-NAT-Gateway/blob/4e0ecce9d124b36fcc3e07a9cbd101b8c90c9992/Screenshot%20(129).png)

## SSH into the Bastion Host

![Alt Text](https://github.com/Naveen15github/Production-Ready-AWS-VPC-Architecture-with-Private-Subnets-and-NAT-Gateway/blob/4e0ecce9d124b36fcc3e07a9cbd101b8c90c9992/Screenshot%20(130).png)

## SSH into the server which is in the private Subnet

![Alt Text](https://github.com/Naveen15github/Production-Ready-AWS-VPC-Architecture-with-Private-Subnets-and-NAT-Gateway/blob/4e0ecce9d124b36fcc3e07a9cbd101b8c90c9992/Screenshot%20(131).png)

## 🟩 Target Group Creation (For Auto Scaling Group + Load Balancer)

The Target Group is where your EC2 instances (launched by the Auto Scaling Group) will register and receive traffic from the Load Balancer.
Follow these steps to create an Application Load Balancer Target Group:

✅ Step 1: Open Target Groups

Go to AWS Console → EC2

On the left menu, scroll down to Load Balancing → Target Groups

Click Create target group

✅ Step 2: Choose Target Type

Select:

Target Type: Instances

This means the load balancer will route traffic directly to EC2 instances.

Click Next.

✅ Step 3: Configure Target Group

Fill in the details:

Field	Value
Target group name	my-app-tg
Protocol	HTTP
Port	80
VPC	Select your project VPC (example: prod-vpc)
IP Address Type	IPv4
✅ Step 4: Health Check Settings

Health checks ensure only healthy instances receive traffic.

Use:

Protocol: HTTP

Path: /

Healthy threshold: 5

Unhealthy threshold: 2

Timeout: 5s

Interval: 10s

(Default values are fine for beginners.)

❗ Do NOT register any targets now

Since the Auto Scaling Group will automatically launch instances and attach them to this Target Group, you must skip manual registration.

Click Create target group.

🎉 Target Group Successfully Created

You now have:

A Target Group ready to receive traffic

Health checks configured

ASG will auto-register instances

Load Balancer will forward all traffic to this Target Group


![Alt Text](https://github.com/Naveen15github/Production-Ready-AWS-VPC-Architecture-with-Private-Subnets-and-NAT-Gateway/blob/4e0ecce9d124b36fcc3e07a9cbd101b8c90c9992/Screenshot%20(132).png)

## 🟦 Load Balancer Creation (Application Load Balancer)

The Load Balancer distributes incoming traffic across your EC2 instances inside the Auto Scaling Group.
We will create an Application Load Balancer (ALB) to route HTTP traffic to the Target Group created earlier.

✅ Step 1: Open Load Balancer Page

Go to AWS Console → EC2

On the left menu, select Load Balancers

Click Create load balancer

Choose Application Load Balancer

✅ Step 2: Configure Basic Settings

Fill in the details:

Field	Value
Name	my-app-alb
Scheme	Internet-facing
IP Address Type	IPv4
Load Balancer Type	Application Load Balancer
✅ Step 3: Select Network & Subnets

Choose:

VPC: Select your project VPC (example: prod-vpc)

Availability Zones: Select both AZs

Public Subnets:

public-subnet-1

public-subnet-2

These public subnets allow the ALB to be accessible from the internet.

✅ Step 4: Security Groups

Attach a Security Group that allows:

Inbound:

HTTP → port 80

(Optional) HTTPS → port 443

Outbound: All traffic allowed

Example SG name:
alb-sg

✅ Step 5: Configure Listeners & Forwarding

Under Listeners:

Listener	Protocol	Port	Action
Listener 1	HTTP	80	Forward to Target Group

Select:

Forward to: my-app-tg (your previously created Target Group)

✅ Step 6: (Optional) Add Tags

Add tags for better project identification:

Example:

Key: Project
Value: VPC-AutoScaling-ALB

✅ Step 7: Create Load Balancer

Click Create load balancer

AWS will take a minute or two to provision it.

🎉 Load Balancer Successfully Created

At this stage, you have:

✔ An internet-facing Application Load Balancer
✔ Traffic forwarding to Target Group → Auto Scaling EC2 Instances
✔ Public subnets for external access
✔ Health check integration with your ASG instances

![Alt Text](https://github.com/Naveen15github/Production-Ready-AWS-VPC-Architecture-with-Private-Subnets-and-NAT-Gateway/blob/4e0ecce9d124b36fcc3e07a9cbd101b8c90c9992/Screenshot%20(133).png)

![Alt Text](https://github.com/Naveen15github/Production-Ready-AWS-VPC-Architecture-with-Private-Subnets-and-NAT-Gateway/blob/4e0ecce9d124b36fcc3e07a9cbd101b8c90c9992/Screenshot%20(134).png)

## 🎯 Final Result

![Alt Text](https://github.com/Naveen15github/Production-Ready-AWS-VPC-Architecture-with-Private-Subnets-and-NAT-Gateway/blob/4e0ecce9d124b36fcc3e07a9cbd101b8c90c9992/Screenshot%20(135).png)

By the end of this project, you will have successfully deployed a production-ready, scalable, secure, and highly available AWS architecture, consisting of:

✅ Fully Configured VPC

Custom VPC with public and private subnets across two Availability Zones

Route tables, NAT Gateway, and Internet Gateway configured

Secure network segmentation for real-world workloads

✅ Bastion Host for Secure Access

Public Bastion EC2 to SSH into private instances

Restricted access via Security Groups for maximum security

✅ Launch Template + Auto Scaling Group

Automated EC2 instance provisioning

Scaling based on CPU or traffic load

Instances deployed only in private subnets

✅ Target Group & Load Balancer

Application Load Balancer (ALB) in public subnets

Health checks and intelligent traffic routing

Auto-registration of ASG instances into the target group

✅ End-to-End Infrastructure Workflow

User → ALB → Target Group → Auto Scaling EC2 Instances

Private EC2 → NAT Gateway → Internet

Bastion Host → SSH → Private EC2

🚀 What You Achieve

With this setup, you get:

High Availability – multi-AZ architecture

Scalability – Auto Scaling responds to demand

Security – isolated private resources, controlled access

Production Standards – similar topology used in real projects

Beginner-Friendly Learning – deep understanding of AWS networking and compute
