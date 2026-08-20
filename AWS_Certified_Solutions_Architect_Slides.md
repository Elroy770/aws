<!-- Page 1 -->

# AWS Certified Solutions Architect Associate

By Stéphane Maarek
https://links.data
cumulus.com/aw
s-certified-saassociate-coupon
https://links.da
tacumulus.com
/aws-certifiedsa-associatecoupon
https://links.datacumulus.com/aw
s-certified-sa-associate-coupon
COURSE
https://links.dat
acumulus.com/
aws-certsolutionarchitect-pthttps://links.datacumulus.com/aws
EXTRA PRACTICE EXAMS
coupon
-cert-solution-architect-pt-coupon

<!-- Page 2 -->

# strictly for personal use only

- This document is reserved for people enrolled into the
Ultimate AWS Solutions Architect Associate Course
- Please do not share this document, it is intended for personal use and exam
preparation only, thank you.
- If you’ve obtained these slides for free on a website that is not the course’s
website, please reach out to piracy@datacumulus.com. Thanks!
- Best of luck for the exam and happy learning!
Disclaimer: These slides are copyrighted and

<!-- Page 3 -->

# Table of Contents

- Getting Started with AWS
- AWS Identity & Access Management (AWS IAM)
- Amazon EC2 – Basics
- Amazon EC2 – Associate
- Amazon EC2 – Instance Storage
- High Availability & Scalability
- RDS, Aurora & ElastiCache
- Amazon Route 53
- Classic Solutions Architecture
- Amazon S3

<!-- Page 4 -->

# Table of Contents

- Amazon S3 – Advanced
- Amazon S3 – Security
- CloudFront & Global Accelerator
- AWS Storage Extras
- AWS Integration & Messaging
- Containers on AWS
- Serverless Overview
- Serverless Architectures
- Databases in AWS
- Data & Analytics

<!-- Page 5 -->

# Table of Contents

- Machine Learning
- AWS Monitoring, Audit & Performance
- Advanced Identity in AWS
- AWS Security & Encryption
- Amazon VPC
- Disaster Recovery & Migrations
- More Solutions Architecture
- Other Services
- White Papers & Architectures
- Exam Preparation
- Congratulations

<!-- Page 6 -->

# Architect Associate Course

SAA-C03
AWS Certified Solutions

<!-- Page 7 -->

# Welcome! We’re starting in 5 minutes

- We’re going to prepare for the Solutions Architect exam - SAA-C03
- It’s a challenging certification, so this course will be long and interesting
- Basic IT knowledge is necessary
- This course contains videos…
- From the Cloud Practitioner, Developer and SysOps course - shared knowledge
- Specific to the Solutions Architect exam - exciting ones on architecture!
- We will cover over 30 AWS services
- AWS / IT Beginners welcome! (but take your time, it’s not a race)

<!-- Page 8 -->

# My SAA-C03 certification: 96.1%

<!-- Page 9 -->

# About me

- I’m Stephane!
- Worked as in IT consultant and AWS Solutions Architect, Developer & SysOps
- Worked with AWS many years: built websites, apps, streaming platforms
- Veteran Instructor on AWS (Certifications, CloudFormation, Lambda, EC2…)
- You can find me on
- GitHub: https://github.com/simplesteph
- LinkedIn: https://www.linkedin.com/in/stephanemaarek
- Medium: https://medium.com/@stephane.maarek
- Twitter: https://twitter.com/stephanemaarek

<!-- Page 10 -->

# What’s AWS?

- AWS (Amazon Web Services) is a Cloud Provider
- They provide you with servers and services that you can use on
demand and scale easily
- AWS has revolutionized IT over time
- AWS powers some of the biggest websites in the world
- Amazon.com
- Netflix

<!-- Page 11 -->

# What we’ll learn in this course (and more!)

Amazon EC2
Amazon ECR
Amazon
SES
Amazon
CloudWatch
Amazon ECS
Amazon
RDS
AWS
CloudFormation
AWS Elastic
Beanstalk
Amazon
Aurora
AWS
CloudTrail
Amazon
DynamoDB
Amazon API
Gateway
AWS
Lambda
Auto Scaling
Amazon
ElastiCache
Elastic Load
Balancing
Amazon
SQS
Amazon
CloudFront
IAM
Amazon
S3
AWS KMS
Amazon
SNS
Amazon
Kinesis
AWS Step Functions
Amazon
Route 53

<!-- Page 12 -->

# Navigating the AWS spaghetti bowl

<!-- Page 13 -->

# Udemy Tips

<!-- Page 14 -->

# Getting started with AWS

<!-- Page 15 -->

# AWS Cloud History

2002:
Internally
launched
2004:
Launched publicly
with SQS
2003:
Amazon infrastructure is
one of their core strength.
Idea to market
2007:
Launched in
Europe
2006:
Re-launched
publicly with
SQS, S3 & EC2

<!-- Page 16 -->

# AWS Cloud Number Facts

- In 2023, AWS had $90 billion
in annual revenue
- AWS accounts for 31% of the
market in Q1 2024 (Microsoft
is 2nd with 25%)
- Pioneer and Leader of the
AWS Cloud Market for the
13th consecutive year
- Over 1,000,000 active users
Gartner Magic Quadrant

<!-- Page 17 -->

# AWS Cloud Use Cases

- AWS enables you to build sophisticated, scalable applications
- Applicable to a diverse set of industries
- Use cases include
- Enterprise IT, Backup & Storage, Big Data analytics
- Website hosting, Mobile & Social Apps
- Gaming

<!-- Page 18 -->

# AWS Global Infrastructure

- AWS Regions
- AWS Availability Zones
- AWS Data Centers
- AWS Edge Locations /
Points of Presence
- https://infrastructure.aws/

<!-- Page 19 -->

# AWS Regions

- AWS has Regions all around the world
- Names can be us-east-1, eu-west-3…
- A region is a cluster of data centers
- Most AWS services are region-scoped
https://aws.amazon.com/about-aws/global-infrastructure/

<!-- Page 20 -->

# How to choose an AWS Region?

If you need to launch a new application,
where should you do it?
?
?
?
?
- Compliance with data governance and legal
requirements: data never leaves a region without
your explicit permission
- Proximity to customers: reduced latency
- Available services within a Region: new services
and new features aren’t available in every Region
- Pricing: pricing varies region to region and is
transparent in the service pricing page

<!-- Page 21 -->

# AWS Availability Zones

- Each region has many availability zones
(usually 3, min is 3, max is 6). Example:
- ap-southeast-2a
- ap-southeast-2b
- ap-southeast-2c
- Each availability zone (AZ) is one or more
discrete data centers with redundant power,
networking, and connectivity
- They’re separate from each other, so that
they’re isolated from disasters
- They’re connected with high bandwidth,
ultra-low latency networking
AWS Region
Sydney: ap-southeast-2
ap-southeast-2a
ap-southeast-2b
ap-southeast-2c

<!-- Page 22 -->

# AWS Points of Presence (Edge Locations)

- Amazon has 400+ Points of Presence (400+ Edge Locations & 10+
Regional Caches) in 90+ cities across 40+ countries
- Content is delivered to end users with lower latency
https://aws.amazon.com/cloudfront/features/

<!-- Page 23 -->

# Tour of the AWS Console

- AWS has Global Services:
- Identity and Access Management (IAM)
- Route 53 (DNS service)
- CloudFront (Content Delivery Network)
- WAF (Web Application Firewall)
- Most AWS services are Region-scoped:
- Amazon EC2 (Infrastructure as a Service)
- Elastic Beanstalk (Platform as a Service)
- Lambda (Function as a Service)
- Rekognition (Software as a Service)
- Region Table: https://aws.amazon.com/about-aws/global-infrastructure/regional-product-services

<!-- Page 24 -->

# Management (AWS IAM)

AWS Identity and Access

<!-- Page 25 -->

# IAM: Users & Groups

- IAM = Identity and Access Management, Global service
- Root account created by default, shouldn’t be used or shared
- Users are people within your organization, and can be grouped
- Groups only contain users, not other groups
- Users don’t have to belong to a group, and user can belong to multiple groups
Group: Developers
Alice
Bob
Group: Operations
Group
Audit Team
Charles
David
Edward
Fred

<!-- Page 26 -->

# IAM: Permissions

{
"Version": "2012-10-17",
"Statement": [
{
"Effect": "Allow",
"Action": "ec2:Describe*",
"Resource": "*"
},
{
"Effect": "Allow",
"Action": "elasticloadbalancing:Describe*",
"Resource": "*"
},
{
"Effect": "Allow",
"Action": [
"cloudwatch:ListMetrics",
"cloudwatch:GetMetricStatistics",
"cloudwatch:Describe*"
],
"Resource": "*"
}
]
- Users or Groups can be
assigned JSON documents
called policies
- These policies define the
permissions of the users
- In AWS you apply the least
privilege principle: don’t give
more permissions than a user
needs
}

<!-- Page 27 -->

# IAM Policies inheritance

Audit Team
Operations
Developers
inline
Alice
Bob
Charles
David
Edward
Fred

<!-- Page 28 -->

# IAM Policies Structure

- Consists of
- Version: policy language version, always include “2012-1017”
- Id: an identifier for the policy (optional)
- Statement: one or more individual statements (required)
- Statements consists of
- Sid: an identifier for the statement (optional)
- Effect: whether the statement allows or denies access
(Allow, Deny)
- Principal: account/user/role to which this policy applied to
- Action: list of actions this policy allows or denies
- Resource: list of resources to which the actions applied to
- Condition: conditions for when this policy is in effect
(optional)

<!-- Page 29 -->

# IAM – Password Policy

- Strong passwords = higher security for your account
- In AWS, you can setup a password policy:
- Set a minimum password length
- Require specific character types:
- including uppercase letters
- lowercase letters
- numbers
- non-alphanumeric characters
- Allow all IAM users to change their own passwords
- Require users to change their password after some time (password expiration)
- Prevent password re-use

<!-- Page 30 -->

# Multi Factor Authentication - MFA

- Users have access to your account and can possibly change
configurations or delete resources in your AWS account
- You want to protect your Root Accounts and IAM users
- MFA = password you know + security device you own
Password
+
=>
Successful login
Alice
- Main benefit of MFA:
if a password is stolen or hacked, the account is not compromised

<!-- Page 31 -->

# MFA devices options in AWS

Virtual MFA device
Google Authenticator
(phone only)
Authy
(phone only)
Support for multiple tokens on a single device.
Universal 2nd Factor (U2F) Security Key
YubiKey by Yubico (3rd party)
Support for multiple root and IAM users
using a single security key

<!-- Page 32 -->

# MFA devices options in AWS

Hardware Key Fob MFA Device
Hardware Key Fob MFA Device for
AWS GovCloud (US)
Provided by Gemalto (3rd party)
Provided by SurePassID (3rd party)

<!-- Page 33 -->

# How can users access AWS ?

- To access AWS, you have three options:
- AWS Management Console (protected by password + MFA)
- AWS Command Line Interface (CLI): protected by access keys
- AWS Software Developer Kit (SDK) - for code: protected by access keys
- Access Keys are generated through the AWS Console
- Users manage their own access keys
- Access Keys are secret, just like a password. Don’t share them
- Access Key ID ~= username
- Secret Access Key ~= password

<!-- Page 34 -->

# Example (Fake) Access Keys

- Access key ID: AKIASK4E37PV4983d6C
- Secret Access Key: AZPN3zojWozWCndIjhB0Unh8239a1bzbzO5fqqkZq
- Remember: don’t share your access keys

<!-- Page 35 -->

# What’s the AWS CLI?

- A tool that enables you to interact with AWS services using commands in
your command-line shell
- Direct access to the public APIs of AWS services
- You can develop scripts to manage your resources
- It’s open-source https://github.com/aws/aws-cli
- Alternative to using AWS Management Console

<!-- Page 36 -->

# What’s the AWS SDK?

- AWS Software Development Kit (AWS SDK)
- Language-specific APIs (set of libraries)
- Enables you to access and manage AWS services
programmatically
- Embedded within your application
- Supports
- SDKs (JavaScript, Python, PHP, .NET, Ruby, Java, Go, Node.js,
C++)
- Mobile SDKs (Android, iOS, …)
- IoT Device SDKs (Embedded C, Arduino, …)
- Example: AWS CLI is built on AWS SDK for Python
AWS SDK
Your Application

<!-- Page 37 -->

# IAM Roles for Services

- Some AWS service will need to
perform actions on your behalf
- To do so, we will assign
permissions to AWS services
with IAM Roles
- Common roles:
- EC2 Instance Roles
- Lambda Function Roles
- Roles for CloudFormation
IAM Role
EC2 Instance
(virtual server)
Access AWS

<!-- Page 38 -->

# IAM Security Tools

- IAM Credentials Report (account-level)
- a report that lists all your account's users and the status of their various
credentials
- IAM Access Advisor (user-level)
- Access advisor shows the service permissions granted to a user and when those
services were last accessed.
- You can use this information to revise your policies.

<!-- Page 39 -->

# IAM Guidelines & Best Practices

- Don’t use the root account except for AWS account setup
- One physical user = One AWS user
- Assign users to groups and assign permissions to groups
- Create a strong password policy
- Use and enforce the use of Multi Factor Authentication (MFA)
- Create and use Roles for giving permissions to AWS services
- Use Access Keys for Programmatic Access (CLI / SDK)
- Audit permissions of your account using IAM Credentials Report & IAM
Access Advisor
- Never share IAM users & Access Keys

<!-- Page 40 -->

# IAM Section – Summary

- Users: mapped to a physical user, has a password for AWS Console
- Groups: contains users only
- Policies: JSON document that outlines permissions for users or groups
- Roles: for EC2 instances or AWS services
- Security: MFA + Password Policy
- AWS CLI: manage your AWS services using the command-line
- AWS SDK: manage your AWS services using a programming language
- Access Keys: access AWS using the CLI or SDK
- Audit: IAM Credential Reports & IAM Access Advisor

<!-- Page 41 -->

# Amazon EC2 – Basics

<!-- Page 42 -->

# Amazon EC2

- EC2 is one of the most popular of AWS’ offering
- EC2 = Elastic Compute Cloud = Infrastructure as a Service
- It mainly consists in the capability of :
- Renting virtual machines (EC2)
- Storing data on virtual drives (EBS)
- Distributing load across machines (ELB)
- Scaling the services using an auto-scaling group (ASG)
- Knowing EC2 is fundamental to understand how the Cloud works

<!-- Page 43 -->

# EC2 sizing & configuration options

- Operating System (OS): Linux, Windows or Mac OS
- How much compute power & cores (CPU)
- How much random-access memory (RAM)
- How much storage space:
- Network-attached (EBS & EFS)
- hardware (EC2 Instance Store)
- Network card: speed of the card, Public IP address
- Firewall rules: security group
- Bootstrap script (configure at first launch): EC2 User Data

<!-- Page 44 -->

# EC2 User Data

- It is possible to bootstrap our instances using an EC2 User data script.
- bootstrapping means launching commands when a machine starts
- That script is only run once at the instance first start
- EC2 user data is used to automate boot tasks such as:
- Installing updates
- Installing software
- Downloading common files from the internet
- Anything you can think of
- The EC2 User Data Script runs with the root user

<!-- Page 45 -->

# Launching an EC2 Instance running Linux

- We’ll be launching our first virtual server using the AWS Console
- We’ll get a first high-level approach to the various parameters
- We’ll see that our web server is launched using EC2 user data
- We’ll learn how to start / stop / terminate our instance.
Hands-On:

<!-- Page 46 -->

# EC2 Instance Types - Overview

- You can use different types of EC2 instances that are optimised for
different use cases (https://aws.amazon.com/ec2/instance-types/)
- AWS has the following naming convention:
m5.2xlarge
- m: instance class
- 5: generation (AWS improves them over time)
- 2xlarge: size within the instance class

<!-- Page 47 -->

# EC2 Instance Types – General Purpose

- Great for a diversity of workloads such as web servers or code repositories
- Balance between:
- Compute
- Memory
- Networking
- In the course, we will be using the t2.micro which is a General Purpose EC2
instance
* this list will evolve over time, please check the AWS website for the latest information

<!-- Page 48 -->

# EC2 Instance Types – Compute Optimized

- Great for compute-intensive tasks that require high performance
processors:
- Batch processing workloads
- Media transcoding
- High performance web servers
- High performance computing (HPC)
- Scientific modeling & machine learning
- Dedicated gaming servers
* this list will evolve over time, please check the AWS website for the latest information

<!-- Page 49 -->

# EC2 Instance Types – Memory Optimized

- Fast performance for workloads that process large data sets in memory
- Use cases:
- High performance, relational/non-relational databases
- Distributed web scale cache stores
- In-memory databases optimized for BI (business intelligence)
- Applications performing real-time processing of big unstructured data
* this list will evolve over time, please check the AWS website for the latest information

<!-- Page 50 -->

# EC2 Instance Types – Storage Optimized

- Great for storage-intensive tasks that require high, sequential read and write
access to large data sets on local storage
- Use cases:
- High frequency online transaction processing (OLTP) systems
- Relational & NoSQL databases
- Cache for in-memory databases (for example, Redis)
- Data warehousing applications
- Distributed file systems
* this list will evolve over time, please check the AWS website for the latest information

<!-- Page 51 -->

# EC2 Instance Types: example

Instance
vCPU Mem (GiB)
Storage
Network
Performance
EBS Bandwidth
(Mbps)
t2.micro
1
1
EBS-Only
Low to Moderate
t2.xlarge
4
16
EBS-Only
Moderate
c5d.4xlarge
16
32
1 x 400 NVMe SSD
Up to 10 Gbps
4,750
r5.16xlarge
64
512
EBS Only
20 Gbps
13,600
m5.8xlarge
32
128
EBS Only
10 Gbps
6,800
Great website: https://instances.vantage.sh

<!-- Page 52 -->

# Introduction to Security Groups

Inbound traffic
WWW
Outbound traffic
Security
Group
- Security Groups are the fundamental of network security in AWS
- They control how traffic is allowed into or out of our EC2 Instances.
EC2 Instance
- Security groups only contain
rules
- Security groups rules can reference by IP or by security group

<!-- Page 53 -->

# Deeper Dive

- Security groups are acting as a “firewall” on EC2 instances
- They regulate:
- Access to Ports
- Authorised IP ranges – IPv4 and IPv6
- Control of inbound network (from other to the instance)
- Control of outbound network (from the instance to other)
Security Groups

<!-- Page 54 -->

# Diagram

Security Group 1
Inbound
Filter IP / Port with Rules
Port 22
Your Computer - IP XX.XX.XX.XX
(authorised port 22)
Port 22
Other computer
(not authorised port 22)
EC2 Instance
IP XX.XX.XX.XX
Security Group 1
Outbound
Filter IP / Port with Rules
Any Port
WWW
Any IP – Any Port
Security Groups

<!-- Page 55 -->

# Good to know

- Can be attached to multiple instances
- Locked down to a region / VPC combination
- Does live “outside” the EC2 – if traffic is blocked the EC2 instance won’t see it
- It’s good to maintain one separate security group for SSH access
- If your application is not accessible (time out), then it’s a security group issue
- If your application gives a “connection refused“ error, then it’s an application
error or it’s not launched
- All inbound traffic is blocked by default
- All outbound traffic is authorised by default
Security Groups

<!-- Page 56 -->

# Diagram

EC2 Instance
IP XX.XX.XX.XX
Security Group 1
Inbound
Authorising Security Group 1
Authorising Security Group 2
Port 123
Security
Group 2
(attached)
EC2 Instance
IP XX.XX.XX.XX
Port 123
Security
Group 1
(attached)
EC2 Instance
IP XX.XX.XX.XX
Port 123
Security
Group 3
(attached)
EC2 Instance
IP XX.XX.XX.XX
Referencing other security groups

<!-- Page 57 -->

# Classic Ports to know

- 22 = SSH (Secure Shell) - log into a Linux instance
- 21 = FTP (File Transfer Protocol) – upload files into a file share
- 22 = SFTP (Secure File Transfer Protocol) – upload files using SSH
- 80 = HTTP – access unsecured websites
- 443 = HTTPS – access secured websites
- 3389 = RDP (Remote Desktop Protocol) – log into a Windows instance

<!-- Page 58 -->

# SSH Summary Table

SSH
Mac
Linux
Windows < 10
Windows >= 10
Putty
EC2 Instance
Connect

<!-- Page 59 -->

# Which Lectures to watch

- Mac / Linux:
- SSH on Mac/Linux lecture
- Windows:
- Putty Lecture
- If Windows 10: SSH on Windows 10 lecture
- All:
- EC2 Instance Connect lecture

<!-- Page 60 -->

# SSH troubleshooting

- Students have the most problems with SSH
- If things don’t work…
1. Re-watch the lecture. You may have missed something
2. Read the troubleshooting guide
3. Try EC2 Instance Connect
- If one method works (SSH, Putty or EC2 Instance Connect) you’re good
- If no method works, that’s okay, the course won’t use SSH much

<!-- Page 61 -->

# Linux / Mac OS X

- We’ll learn how to SSH into your EC2 instance using Linux / Mac
- SSH is one of the most important function. It allows you to control a
remote machine, all using the command line.
SSH – Port 22
WWW
EC2 Instance
Linux
Public IP
- We will see how we can configure OpenSSH ~/.ssh/config to facilitate
the SSH into our EC2 instances
How to SSH into your EC2 Instance

<!-- Page 62 -->

# Windows

- We’ll learn how to SSH into your EC2 instance using Windows
- SSH is one of the most important function. It allows you to control a
remote machine, all using the command line.
SSH – Port 22
WWW
EC2 Instance
Linux
Public IP
- We will configure all the required parameters necessary for doing SSH
on Windows using the free tool Putty.
How to SSH into your EC2 Instance

<!-- Page 63 -->

# EC2 Instance Connect

- Connect to your EC2 instance within your browser
- No need to use your key file that was downloaded
- The “magic” is that a temporary key is uploaded onto EC2 by AWS
- Works only out-of-the-box with Amazon Linux 2
- Need to make sure the port 22 is still opened!

<!-- Page 64 -->

# EC2 Instances Purchasing Options

- On-Demand Instances – short workload, predictable pricing, pay by second
- Reserved (1 & 3 years)
- Reserved Instances – long workloads
- Convertible Reserved Instances – long workloads with flexible instances
- Savings Plans (1 & 3 years) –commitment to an amount of usage, long workload
- Spot Instances – short workloads, cheap, can lose instances (less reliable)
- Dedicated Hosts – book an entire physical server, control instance placement
- Dedicated Instances – no other customers will share your hardware
- Capacity Reservations – reserve capacity in a specific AZ for any duration

<!-- Page 65 -->

# EC2 On Demand

- Pay for what you use:
- Linux or Windows - billing per second, after the first minute
- All other operating systems - billing per hour
- Has the highest cost but no upfront payment
- No long-term commitment
- Recommended for short-term and un-interrupted workloads, where you
can't predict how the application will behave

<!-- Page 66 -->

# EC2 Reserved Instances

- Up to 72% discount compared to On-demand
- You reserve a specific instance attributes (Instance Type, Region, Tenancy, OS)
- Reservation Period – 1 year (+discount) or 3 years (+++discount)
- Payment Options – No Upfront (+), Partial Upfront (++), All Upfront (+++)
- Reserved Instance’s Scope – Regional or Zonal (reserve capacity in an AZ)
- Recommended for steady-state usage applications (think database)
- You can buy and sell in the Reserved Instance Marketplace
- Convertible Reserved Instance
- Can change the EC2 instance type, instance family, OS, scope and tenancy
- Up to 66% discount
Note: the % discounts are different from the video as AWS
change them over time – the exact numbers are not needed
for the exam. This is just for illustrative purposes J

<!-- Page 67 -->

# EC2 Savings Plans

- Get a discount based on long-term usage (up to 72% - same as RIs)
- Commit to a certain type of usage ($10/hour for 1 or 3 years)
- Usage beyond EC2 Savings Plans is billed at the On-Demand price
- Locked to a specific instance family & AWS region (e.g., M5 in us-east-1)
- Flexible across:
- Instance Size (e.g., m5.xlarge, m5.2xlarge)
- OS (e.g., Linux, Windows)
- Tenancy (Host, Dedicated, Default)

<!-- Page 68 -->

# EC2 Spot Instances

- Can get a discount of up to 90% compared to On-demand
- Instances that you can “lose” at any point of time if your max price is less than the
current spot price
- The MOST cost-efficient instances in AWS
- Useful for workloads that are resilient to failure
- Batch jobs
- Data analysis
- Image processing
- Any distributed workloads
- Workloads with a flexible start and end time
- Not suitable for critical jobs or databases

<!-- Page 69 -->

# EC2 Dedicated Hosts

- A physical server with EC2 instance capacity fully dedicated to your use
- Allows you address compliance requirements and use your existing serverbound software licenses (per-socket, per-core, pe—VM software licenses)
- Purchasing Options:
- On-demand – pay per second for active Dedicated Host
- Reserved - 1 or 3 years (No Upfront, Partial Upfront, All Upfront)
- The most expensive option
- Useful for software that have complicated licensing model (BYOL – Bring Your
Own License)
- Or for companies that have strong regulatory or compliance needs

<!-- Page 70 -->

# EC2 Dedicated Instances

- Instances run on hardware that’s
dedicated to you
- May share hardware with other
instances in same account
- No control over instance placement
(can move hardware after Stop / Start)

<!-- Page 71 -->

# EC2 Capacity Reservations

- Reserve On-Demand instances capacity in a specific AZ for any duration
- You always have access to EC2 capacity when you need it
- No time commitment (create/cancel anytime), no billing discounts
- Combine with Regional Reserved Instances and Savings Plans to benefit
from billing discounts
- You’re charged at On-Demand rate whether you run instances or not
- Suitable for short-term, uninterrupted workloads that needs to be in a
specific AZ

<!-- Page 72 -->

# Which purchasing option is right for me?

- On demand: coming and staying in resort
whenever we like, we pay the full price
- Reserved: like planning ahead and if we plan to stay
for a long time, we may get a good discount.
- Savings Plans: pay a certain amount per hour for
certain period and stay in any room type (e.g.,
King, Suite, Sea View, …)
- Spot instances: the hotel allows people to bid for
the empty rooms and the highest bidder keeps the
rooms. You can get kicked out at any time
- Dedicated Hosts: We book an entire building of
the resort
- Capacity Reservations: you book a room for a
period with full price even you don’t stay in it

<!-- Page 73 -->

# Example – m4.large – us-east-1

Price Type
Price (per hour)
On-Demand
$0.10
Spot Instance (Spot Price)
$0.038 - $0.039 (up to 61% off)
Reserved Instance (1 year)
$0.062 (No Upfront) - $0.058 (All Upfront)
Reserved Instance (3 years)
$0.043 (No Upfront) - $0.037 (All Upfront)
EC2 Savings Plan (1 year)
$0.062 (No Upfront) - $0.058 (All Upfront)
Reserved Convertible Instance (1 year)
$0.071 (No Upfront) - $0.066 (All Upfront)
Dedicated Host
On-Demand Price
Dedicated Host Reservation
Up to 70% off
Capacity Reservations
On-Demand Price
Price Comparison

<!-- Page 74 -->

# EC2 Spot Instance Requests

- Can get a discount of up to 90% compared to On-demand
- Define max spot price and get the instance while current spot price < max
- The hourly spot price varies based on offer and capacity
- If the current spot price > your max price you can choose to stop or terminate your
instance with a 2 minutes grace period.
- Used for batch jobs, data analysis, or workloads that are resilient to failures.
- Not great for critical jobs or databases

<!-- Page 75 -->

# EC2 Spot Instances Pricing

User-defined max price
https://us-east-1.console.aws.amazon.com/ec2/home?region=us-east-1#SpotInstances:

<!-- Page 76 -->

# How to terminate Spot Instances?

You can only cancel Spot Instance requests that are open, active, or disabled.
Cancelling a Spot Request does not terminate instances
You must first cancel a Spot Request, and then terminate the associated Spot Instances
https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/spot-requests.html

<!-- Page 77 -->

# Spot Fleets

- Spot Fleets = set of Spot Instances + (optional) On-Demand Instances
- The Spot Fleet will try to meet the target capacity with price constraints
- Define possible launch pools: instance type (m5.large), OS, Availability Zone
- Can have multiple launch pools, so that the fleet can choose
- Spot Fleet stops launching instances when reaching capacity or max cost
- Strategies to allocate Spot Instances:
- lowestPrice: from the pool with the lowest price (cost optimization, short workload)
- diversified: distributed across all pools (great for availability, long workloads)
- capacityOptimized: pool with the optimal capacity for the number of instances
- priceCapacityOptimized (recommended): pools with highest capacity available, then select the
pool with the lowest price (best choice for most workloads)
- Spot Fleets allow us to automatically request Spot Instances with the lowest price

<!-- Page 78 -->

# Amazon EC2 – Associate

<!-- Page 79 -->

# Private vs Public IP (IPv4)

- Networking has two sorts of IPs. IPv4 and IPv6:
- IPv4: 1.160.10.240
- IPv6: 3ffe:1900:4545:3:200:f8ff:fe21:67cf
- In this course, we will only be using IPv4.
- IPv4 is still the most common format used online.
- IPv6 is newer and solves problems for the Internet of Things (IoT).
- IPv4 allows for 3.7 billion different addresses in the public space
- IPv4: [0-255].[0-255].[0-255].[0-255].

<!-- Page 80 -->

# Example

Server (public):
211.139.37.43
Web Server (public):
79.216.59.75
WWW
Internet Gateway (public):
149.140.72.10
Company A
Private Network
192.168.0.1/22
Internet Gateway (public):
253.144.139.205
Company B
Private Network
192.168.0.1/22
Private vs Public IP (IPv4)

<!-- Page 81 -->

# Fundamental Differences

- Public IP:
- Public IP means the machine can be identified on the internet (WWW)
- Must be unique across the whole web (not two machines can have the same public IP).
- Can be geo-located easily
- Private IP:
- Private IP means the machine can only be identified on a private network only
- The IP must be unique across the private network
- BUT two different private networks (two companies) can have the same IPs.
- Machines connect to WWW using a NAT + internet gateway (a proxy)
- Only a specified range of IPs can be used as private IP
Private vs Public IP (IPv4)

<!-- Page 82 -->

# Elastic IPs

- When you stop and then start an EC2 instance, it can change its public
IP.
- If you need to have a fixed public IP for your instance, you need an
Elastic IP
- An Elastic IP is a public IPv4 IP you own as long as you don’t delete it
- You can attach it to one instance at a time

<!-- Page 83 -->

# Elastic IP

- With an Elastic IP address, you can mask the failure of an instance or software
by rapidly remapping the address to another instance in your account.
- You can only have 5 Elastic IP in your account (you can ask AWS to increase
that).
- Overall, try to avoid using Elastic IP:
- They often reflect poor architectural decisions
- Instead, use a random public IP and register a DNS name to it
- Or, as we’ll see later, use a Load Balancer and don’t use a public IP

<!-- Page 84 -->

# In AWS EC2 – Hands On

- By default, your EC2 machine comes with:
- A private IP for the internal AWS Network
- A public IP, for the WWW.
- When we are doing SSH into our EC2 machines:
- We can’t use a private IP, because we are not in the same network
- We can only use the public IP.
- If your machine is stopped and then started,
the public IP can change
Private vs Public IP (IPv4)

<!-- Page 85 -->

# Placement Groups

- Sometimes you want control over the EC2 Instance placement strategy
- That strategy can be defined using placement groups
- When you create a placement group, you specify one of the following
strategies for the group:
- Cluster—clusters instances into a low-latency group in a single Availability Zone
- Spread—spreads instances across underlying hardware (max 7 instances per
group per AZ)
- Partition—spreads instances across many different partitions (which rely on
different sets of racks) within an AZ. Scales to 100s of EC2 instances per group
(Hadoop, Cassandra, Kafka)

<!-- Page 86 -->

# Cluster

EC2
EC2
EC2
EC2
EC2
EC2
Same AZ
Placement group
Low latency
10 Gbps network
- Pros: Great network (10 Gbps bandwidth between instances with Enhanced
Networking enabled - recommended)
- Cons: If the AZ fails, all instances fails at the same time
- Use case:
- Big Data job that needs to complete fast
- Application that needs extremely low latency and high network throughput
Placement Groups

<!-- Page 87 -->

# Spread

Us-east-1a
Us-east-1b
Us-east-1c
EC2
EC2
EC2
Hardware 1
Hardware 3
Hardware 5
- Pros:
- Can span across Availability
Zones (AZ)
- Reduced risk is simultaneous
failure
- EC2 Instances are on different
physical hardware
- Cons:
- Limited to 7 instances per AZ
per placement group
EC2
EC2
EC2
Hardware 2
Hardware 4
Hardware 6
- Use case:
- Application that needs to
maximize high availability
- Critical Applications where
each instance must be isolated
from failure from each other
Placement Groups

<!-- Page 88 -->

# Partition

us-east-1b
us-east-1a
EC2
EC2
EC2
EC2
EC2
EC2
EC2
EC2
EC2
EC2
EC2
EC2
Partition 1
Partition 2
Partition 3
- Up to 7 partitions per AZ
- Can span across multiple AZs in the
same region
- Up to 100s of EC2 instances
- The instances in a partition do not
share racks with the instances in the
other partitions
- A partition failure can affect many
EC2 but won’t affect other partitions
- EC2 instances get access to the
partition information as metadata
- Use cases: HDFS, HBase, Cassandra,
Kafka
Placements Groups

<!-- Page 89 -->

# Elastic Network Interfaces (ENI)

- Logical component in a VPC that represents a
virtual network card
- The ENI can have the following attributes:
- Primary private IPv4, one or more secondary IPv4
- One Elastic IP (IPv4) per private IPv4
- One Public IPv4
- One or more security groups
- A MAC address
- You can create ENI independently and attach
them on the fly (move them) on EC2 instances
for failover
- Bound to a specific availability zone (AZ)
Availability Zone
EC2
Eth0 – primary ENI
192.168.0.31
Eth1 – secondary ENI
192.168.0.42
Can be moved
EC2
Eth0 – primary ENI

<!-- Page 90 -->

# EC2 Hibernate

- We know we can stop, terminate instances
- Stop – the data on disk (EBS) is kept intact in the next start
- Terminate – any EBS volumes (root) also set-up to be destroyed is lost
- On start, the following happens:
- First start: the OS boots & the EC2 User Data script is run
- Following starts: the OS boots up
- Then your application starts, caches get warmed up, and that can take time!

<!-- Page 91 -->

# RAM

EC2 Instance
Running
- Introducing EC2 Hibernate:
- The in-memory (RAM) state is preserved
- The instance boot is much faster!
(the OS is not stopped / restarted)
- Under the hood: the RAM state is written
to a file in the root EBS volume
- The root EBS volume must be encrypted
- Use cases:
- Long-running processing
- Saving the RAM state
- Services that take time to initialize
Root EBS Volume
(Encrypted)
Hibernate
Stopping
Hibernation
Shutdown
Stopped
Start
Running
EC2 Hibernate

<!-- Page 92 -->

# EC2 Hibernate – Good to know

- Supported Instance Families – C3, C4, C5, I3, M3, M4, R3, R4, T2, T3, …
- Instance RAM Size – must be less than 150 GB.
- Instance Size – not supported for bare metal instances.
- AMI – Amazon Linux 2, Linux AMI, Ubuntu, RHEL, CentOS & Windows…
- Root Volume – must be EBS, encrypted, not instance store, and large
- Available for On-Demand, Reserved and Spot Instances
- An instance can NOT be hibernated more than 60 days

<!-- Page 93 -->

# Amazon EC2 – Instance Storage

<!-- Page 94 -->

# What’s an EBS Volume?

- An EBS (Elastic Block Store) Volume is a network drive you can attach
to your instances while they run
- It allows your instances to persist data, even after their termination
- They can only be mounted to one instance at a time (at the CCP level)
- They are bound to a specific availability zone
- Analogy: Think of them as a “network USB stick”

<!-- Page 95 -->

# EBS Volume

- It’s a network drive (i.e. not a physical drive)
- It uses the network to communicate the instance, which means there might be a bit of
latency
- It can be detached from an EC2 instance and attached to another one quickly
- It’s locked to an Availability Zone (AZ)
- An EBS Volume in us-east-1a cannot be attached to us-east-1b
- To move a volume across, you first need to snapshot it
- Have a provisioned capacity (size in GBs, and IOPS)
- You get billed for all the provisioned capacity
- You can increase the capacity of the drive over time

<!-- Page 96 -->

# EBS Volume - Example

US-EAST-1A
EBS
(10 GB)
EBS
(100 GB)
US-EAST-1B
EBS
(50 GB)
EBS
(50 GB)
EBS
(10 GB)
unattached

<!-- Page 97 -->

# EBS – Delete on Termination attribute

- Controls the EBS behaviour when an EC2 instance terminates
- By default, the root EBS volume is deleted (attribute enabled)
- By default, any other attached EBS volume is not deleted (attribute disabled)
- This can be controlled by the AWS console / AWS CLI
- Use case: preserve root volume when instance is terminated

<!-- Page 98 -->

# EBS Snapshots

- Make a backup (snapshot) of your EBS volume at a point in time
- Not necessary to detach volume to do snapshot, but recommended
- Can copy snapshots across AZ or Region
US-EAST-1A
US-EAST-1B
EBS Snapshot
EBS
(50 GB)
snapshot
restore
EBS
(50 GB)

<!-- Page 99 -->

# EBS Snapshots Features

EBS Snapshot
Archive
EBS Snapshot
- EBS Snapshot Archive
- Move a Snapshot to an ”archive tier” that is
75% cheaper
- Takes within 24 to 72 hours for restoring the
archive
- Recycle Bin for EBS Snapshots
- Setup rules to retain deleted snapshots so you
can recover them after an accidental deletion
- Specify retention (from 1 day to 1 year)
- Fast Snapshot Restore (FSR)
- Force full initialization of snapshot to have no
latency on the first use ($$$)
archive
EBS Snapshot
Recycle Bin
delete

<!-- Page 100 -->

# AMI Overview

- AMI = Amazon Machine Image
- AMI are a customization of an EC2 instance
- You add your own software, configuration, operating system, monitoring…
- Faster boot / configuration time because all your software is pre-packaged
- AMI are built for a specific region (and can be copied across regions)
- You can launch EC2 instances from:
- A Public AMI: AWS provided
- Your own AMI: you make and maintain them yourself
- An AWS Marketplace AMI: an AMI someone else made (and potentially sells)

<!-- Page 101 -->

# AMI Process (from an EC2 instance)

- Start an EC2 instance and customize it
- Stop the instance (for data integrity)
- Build an AMI – this will also create EBS snapshots
- Launch instances from other AMIs
Custom AMI
US-EAST-1A
Create AMI
Launch
from AMI
US-EAST-1B

<!-- Page 102 -->

# EC2 Instance Store

- EBS volumes are network drives with good but “limited” performance
- If you need a high-performance hardware disk, use EC2 Instance Store
- Better I/O performance
- EC2 Instance Store lose their storage if they’re stopped (ephemeral)
- Good for buffer / cache / scratch data / temporary content
- Risk of data loss if hardware fails
- Backups and Replication are your responsibility

<!-- Page 103 -->

# Local EC2 Instance Store

Very high IOPS

<!-- Page 104 -->

# EBS Volume Types

- EBS Volumes come in 6 types
- gp2 / gp3 (SSD): General purpose SSD volume that balances price and performance for
a wide variety of workloads
- io1 / io2 Block Express (SSD): Highest-performance SSD volume for mission-critical
low-latency or high-throughput workloads
- st1 (HDD): Low cost HDD volume designed for frequently accessed, throughputintensive workloads
- sc1 (HDD): Lowest cost HDD volume designed for less frequently accessed workloads
- EBS Volumes are characterized in Size | Throughput | IOPS (I/O Ops Per Sec)
- When in doubt always consult the AWS documentation – it’s good!
- Only gp2/gp3 and io1/io2 Block Express can be used as boot volumes

<!-- Page 105 -->

# General Purpose SSD

- Cost effective storage, low-latency
- System boot volumes, Virtual desktops, Development and test environments
- 1 GiB - 16 TiB
- gp3:
- Baseline of 3,000 IOPS and throughput of 125 MiB/s
- Can increase IOPS up to 16,000 and throughput up to 1000 MiB/s independently
- gp2:
- Small gp2 volumes can burst IOPS to 3,000
- Size of the volume and IOPS are linked, max IOPS is 16,000
- 3 IOPS per GiB, means at 5,334 GiB we are at the max IOPS
EBS Volume Types Use cases

<!-- Page 106 -->

# Provisioned IOPS (PIOPS) SSD

- Critical business applications with sustained IOPS performance
- Or applications that need more than 16,000 IOPS
- Great for databases workloads (sensitive to storage perf and consistency)
- io1 (4 GiB - 16 TiB):
- Max PIOPS: 64,000 for Nitro EC2 instances & 32,000 for other
- Can increase PIOPS independently from storage size
- io2 Block Express (4 GiB – 64 TiB):
- Sub-millisecond latency
- Max PIOPS: 256,000 with an IOPS:GiB ratio of 1,000:1
- Supports EBS Multi-attach
EBS Volume Types Use cases

<!-- Page 107 -->

# Hard Disk Drives (HDD)

- Cannot be a boot volume
- 125 GiB to 16 TiB
- Throughput Optimized HDD (st1)
- Big Data, Data Warehouses, Log Processing
- Max throughput 500 MiB/s – max IOPS 500
- Cold HDD (sc1):
- For data that is infrequently accessed
- Scenarios where lowest cost is important
- Max throughput 250 MiB/s – max IOPS 250
EBS Volume Types Use cases

<!-- Page 108 -->

# EBS – Volume Types Summary

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-volume-types.html#solid-state-drives

<!-- Page 109 -->

# EBS Multi-Attach – io1/io2 family

- Attach the same EBS volume to multiple EC2
instances in the same AZ
- Each instance has full read & write permissions
to the high-performance volume
- Use case:
Availability Zone 1
- Achieve higher application availability in clustered
Linux applications (ex: Teradata)
- Applications must manage concurrent write
operations
- Up to 16 EC2 Instances at a time
- Must use a file system that’s cluster-aware (not
XFS, EXT4, etc…)
io2 volume with Multi-Attach

<!-- Page 110 -->

# EBS Encryption

- When you create an encrypted EBS volume, you get the following:
- Data at rest is encrypted inside the volume
- All the data in flight moving between the instance and the volume is encrypted
- All snapshots are encrypted
- All volumes created from the snapshot
- Encryption and decryption are handled transparently (you have nothing to
do)
- Encryption has a minimal impact on latency
- EBS Encryption leverages keys from KMS (AES-256)
- Copying an unencrypted snapshot allows encryption
- Snapshots of encrypted volumes are encrypted

<!-- Page 111 -->

# Encryption: encrypt an unencrypted EBS volume

- Create an EBS snapshot of the volume
- Encrypt the EBS snapshot ( using copy )
- Create new ebs volume from the snapshot ( the volume will also be
encrypted )
- Now you can attach the encrypted volume to the original instance

<!-- Page 112 -->

# Amazon EFS – Elastic File System

- Managed NFS (network file system) that can be mounted on many EC2
- EFS works with EC2 instances in multi-AZ
- Highly available, scalable, expensive (3x gp2), pay per use
us-east-1a
us-east-1b
us-east-1c
EC2 Instances
EC2 Instances
EC2 Instances
Security Group
EFS FileSystem

<!-- Page 113 -->

# Amazon EFS – Elastic File System

- Use cases: content management, web serving, data sharing, Wordpress
- Uses NFSv4.1 protocol
- Uses security group to control access to EFS
- Compatible with Linux based AMI (not Windows)
- Encryption at rest using KMS
- POSIX file system (~Linux) that has a standard file API
- File system scales automatically, pay-per-use, no capacity planning!

<!-- Page 114 -->

# EFS – Performance & Storage Classes

- EFS Scale
- 1000s of concurrent NFS clients, 10 GB+ /s throughput
- Grow to Petabyte-scale network file system, automatically
- Performance Mode (set at EFS creation time)
- General Purpose (default) – latency-sensitive use cases (web server, CMS, etc…)
- Max I/O – higher latency, throughput, highly parallel (big data, media processing)
- Throughput Mode
- Bursting – 1 TB = 50MiB/s + burst of up to 100MiB/s
- Provisioned – set your throughput regardless of storage size, ex: 1 GiB/s for 1 TB storage
- Elastic – automatically scales throughput up or down based on your workloads
- Up to 3GiB/s for reads and 1GiB/s for writes
- Used for unpredictable workloads

<!-- Page 115 -->

# EFS – Storage Classes

- Storage Tiers (lifecycle management feature – move
file after N days)
- Standard: for frequently accessed files
- Infrequent access (EFS-IA): cost to retrieve files, lower price
to store.
- Archive: rarely accessed data (few times each year), 50%
cheaper
- Implement lifecycle policies to move files between storage
tiers
- Availability and durability
no access
for 60 days
EFS Standard
move
Lifecycle Policy
- Standard: Multi-AZ, great for prod
- One Zone: One AZ, great for dev, backup enabled by
default, compatible with IA (EFS One Zone-IA)
- Over 90% in cost savings
EFS IA
Amazon EFS File System

<!-- Page 116 -->

# EBS vs EFS – Elastic Block Storage

- EBS volumes…
- one instance (except multi-attach io1/io2)
- are locked at the Availability Zone (AZ) level
- gp2: IO increases if the disk size increases
- gp3 & io1: can increase IO independently
Availability Zone 1
Availability Zone 2
- To migrate an EBS volume across AZ
- Take a snapshot
- Restore the snapshot to another AZ
- EBS backups use IO and you shouldn’t run
them while your application is handling a lot
of traffic
- Root EBS Volumes of instances get
terminated by default if the EC2 instance
gets terminated. (you can disable that)
EBS
snapshot
restore
EBS Snapshot
EBS

<!-- Page 117 -->

# EBS vs EFS – Elastic File System

- Mounting 100s of instances across AZ
- EFS share website files (WordPress)
- Only for Linux Instances (POSIX)
- EFS has a higher price point than EBS
- Can leverage Storage Tiers for cost savings
Availability Zone 1
Availability Zone 2
Linux
Linux
EFS
Mount
Target
EFS
Mount
Target
- Remember: EFS vs EBS vs Instance Store
EFS

<!-- Page 118 -->

# High Availability & Scalability

<!-- Page 119 -->

# Scalability & High Availability

- Scalability means that an application / system can handle greater loads
by adapting.
- There are two kinds of scalability:
- Vertical Scalability
- Horizontal Scalability (= elasticity)
- Scalability is linked but different to High Availability
- Let’s deep dive into the distinction, using a call center as an example

<!-- Page 120 -->

# Vertical Scalability

- Vertically scalability means increasing the size
of the instance
- For example, your application runs on a
t2.micro
- Scaling that application vertically means
running it on a t2.large
- Vertical scalability is very common for non
distributed systems, such as a database.
- RDS, ElastiCache are services that can scale
vertically.
- There’s usually a limit to how much you can
vertically scale (hardware limit)
junior operator
senior operator

<!-- Page 121 -->

# Horizontal Scalability

operator
operator
operator
operator
operator
operator
- Horizontal Scalability means increasing the
number of instances / systems for your
application
- Horizontal scaling implies distributed systems.
- This is very common for web applications /
modern applications
- It’s easy to horizontally scale thanks to the cloud
offerings such as Amazon EC2

<!-- Page 122 -->

# High Availability

- High Availability usually goes hand in
hand with horizontal scaling
- High availability means running your
application / system in at least 2 data
centers (== Availability Zones)
- The goal of high availability is to survive
a data center loss
- The high availability can be passive (for
RDS Multi AZ for example)
- The high availability can be active (for
horizontal scaling)
first building in New York
second building in San Francisco

<!-- Page 123 -->

# High Availability & Scalability For EC2

- Vertical Scaling: Increase instance size (= scale up / down)
- From: t2.nano - 0.5G of RAM, 1 vCPU
- To: u-12tb1.metal – 12.3 TB of RAM, 448 vCPUs
- Horizontal Scaling: Increase number of instances (= scale out / in)
- Auto Scaling Group
- Load Balancer
- High Availability: Run instances for the same application across multi AZ
- Auto Scaling Group multi AZ
- Load Balancer multi AZ

<!-- Page 124 -->

# What is load balancing?

- Load Balances are servers that forward traffic to multiple
servers (e.g., EC2 instances) downstream
Elastic Load Balancer
EC2 Instance
EC2 Instance
EC2 Instance

<!-- Page 125 -->

# Why use a load balancer?

- Spread load across multiple downstream instances
- Expose a single point of access (DNS) to your application
- Seamlessly handle failures of downstream instances
- Do regular health checks to your instances
- Provide SSL termination (HTTPS) for your websites
- Enforce stickiness with cookies
- High availability across zones
- Separate public traffic from private traffic

<!-- Page 126 -->

# Why use an Elastic Load Balancer?

- An Elastic Load Balancer is a managed load balancer
- AWS guarantees that it will be working
- AWS takes care of upgrades, maintenance, high availability
- AWS provides only a few configuration knobs
- It costs less to setup your own load balancer but it will be a lot more effort
on your end
- It is integrated with many AWS offerings / services
- EC2, EC2 Auto Scaling Groups, Amazon ECS
- AWS Certificate Manager (ACM), CloudWatch
- Route 53, AWS WAF, AWS Global Accelerator

<!-- Page 127 -->

# Health Checks

- Health Checks are crucial for Load Balancers
- They enable the load balancer to know if instances it forwards traffic to
are available to reply to requests
- The health check is done on a port and a route (/health is common)
- If the response is not 200 (OK), then the instance is unhealthy
Elastic Load Balancer
Protocol: HTTP
Port: 4567
Endpoint: /health
EC2 Instance

<!-- Page 128 -->

# Types of load balancer on AWS

- AWS has 4 kinds of managed Load Balancers
- Classic Load Balancer (v1 - old generation) – 2009 – CLB
- HTTP, HTTPS, TCP, SSL (secure TCP)
- Application Load Balancer (v2 - new generation) – 2016 – ALB
- HTTP, HTTPS, WebSocket
- Network Load Balancer (v2 - new generation) – 2017 – NLB
- TCP, TLS (secure TCP), UDP
- Gateway Load Balancer – 2020 – GWLB
- Operates at layer 3 (Network layer) – IP Protocol
- Overall, it is recommended to use the newer generation load balancers as they
provide more features
- Some load balancers can be setup as internal (private) or external (public) ELBs

<!-- Page 129 -->

# Load Balancer Security Groups

Users
HTTPS / HTTP
From anywhere
LOAD BALANCER
Load Balancer Security Group:
Application Security Group: Allow traffic only from Load Balancer
HTTP Restricted
to Load balancer
EC2

<!-- Page 130 -->

# Classic Load Balancers (v1)

- Supports TCP (Layer 4), HTTP &
HTTPS (Layer 7)
- Health checks are TCP or HTTP
based
- Fixed hostname
XXX.region.elb.amazonaws.com
listener
Client
internal
CLB
EC2

<!-- Page 131 -->

# Application Load Balancer (v2)

- Application load balancers is Layer 7 (HTTP)
- Load balancing to multiple HTTP applications across machines
(target groups)
- Load balancing to multiple applications on the same machine
(ex: containers)
- Support for HTTP/2 and WebSocket
- Support redirects (from HTTP to HTTPS for example)

<!-- Page 132 -->

# Application Load Balancer (v2)

- Routing tables to different target groups:
- Routing based on path in URL (example.com/users & example.com/posts)
- Routing based on hostname in URL (one.example.com & other.example.com)
- Routing based on Query String, Headers
(example.com/users?id=123&order=false)
- ALB are a great fit for micro services & container-based application
(example: Docker & Amazon ECS)
- Has a port mapping feature to redirect to a dynamic port in ECS
- In comparison, we’d need multiple Classic Load Balancer per application

<!-- Page 133 -->

# HTTP Based Traffic

WWW
Route /search
HTTP
Health Check
External
Application
Load Balancer
(v2)
Health Check
HTTP
Target Group
for Users
application
Route /user
Target Group
for Search
application
WWW
Application Load Balancer (v2)

<!-- Page 134 -->

# Target Groups

- EC2 instances (can be managed by an Auto Scaling Group) – HTTP
- ECS tasks (managed by ECS itself) – HTTP
- Lambda functions – HTTP request is translated into a JSON event
- IP Addresses – must be private IPs
- ALB can route to multiple target groups
- Health checks are at the target group level
Application Load Balancer (v2)

<!-- Page 135 -->

# Query Strings/Parameters Routing

WWW
Requests
?Platform=Mobile
Target Group 1
AWS – EC2 based
?Platform=Desktop
Target Group 2
On-premises – Private IP routing
External
Application
Load Balancer
(v2)
Application Load Balancer (v2)

<!-- Page 136 -->

# Good to Know

- Fixed hostname (XXX.region.elb.amazonaws.com)
- The application servers don’t see the IP of the client directly
- The true IP of the client is inserted in the header X-Forwarded-For
- We can also get Port (X-Forwarded-Port) and proto (X-Forwarded-Proto)
Client IP
12.34.56.78
Load Balancer IP
(Private IP)
Connection termination
EC2
Instance
Application Load Balancer (v2)

<!-- Page 137 -->

# Network Load Balancer (v2)

- Network load balancers (Layer 4) allow to:
- Forward TCP & UDP traffic to your instances
- Handle millions of request per seconds
- Ultra-low latency
- NLB has one static IP per AZ, and supports assigning Elastic IP
(helpful for whitelisting specific IP)
- NLB are used for extreme performance, TCP or UDP traffic

<!-- Page 138 -->

# TCP (Layer 4) Based Traffic

WWW
TCP + Rules
HTTP
Health Check
External
Network Load
Balancer (v2)
Health Check
TCP
Target Group
for Users
application
TCP + Rules
Target Group
for Search
application
WWW
Network Load Balancer (v2)

<!-- Page 139 -->

# Network Load Balancer – Target Groups

- EC2 instances
- IP Addresses – must be private IPs
- Application Load Balancer
- Health Checks support the TCP, HTTP and HTTPS Protocols
Network
Load Balancer
i-1234567890abcdef0
i-1234567890abcdef0
Target Group
(EC2 Instances)
Network
Load Balancer
192.168.1.118
10.0.4.21
Target Group
(IP Addresses)
Network
Load Balancer
Target Group
(Application Load Balancer)

<!-- Page 140 -->

# Gateway Load Balancer

- Deploy, scale, and manage a fleet of 3rd party
network virtual appliances in AWS
- Example: Firewalls, Intrusion Detection and
Prevention Systems, Deep Packet Inspection
Systems, payload manipulation, …
- Operates at Layer 3 (Network Layer) – IP
Packets
- Combines the following functions:
- Transparent Network Gateway – single entry/exit
for all traffic
- Load Balancer – distributes traffic to your virtual
appliances
Route
Table
Application
Users
(destination)
(source)
traffic
traffic
Gateway
Load Balancer
Target Group
- Uses the GENEVE protocol on port 6081
3rd Party Security
Virtual Appliances

<!-- Page 141 -->

# Gateway Load Balancer – Target Groups

- EC2 instances
- IP Addresses – must be private IPs
Gateway
Load Balancer
i-1234567890abcdef0
i-1234567890abcdef0
Target Group
(EC2 Instances)
Gateway
Load Balancer
192.168.1.118
10.0.4.21
Target Group
(IP Addresses)

<!-- Page 142 -->

# Sticky Sessions (Session Affinity)

- It is possible to implement stickiness so that the
same client is always redirected to the same
instance behind a load balancer
- This works for Classic Load Balancer, Application
Load Balancer, and Network Load Balancer
- For both CLB & ALB, the “cookie” used for
stickiness has an expiration date you control
- Use case: make sure the user doesn’t lose his
session data
- Enabling stickiness may bring imbalance to the
load over the backend EC2 instances
Client 1
EC2 Instance
Client 2
Client 3
EC2 Instance

<!-- Page 143 -->

# Sticky Sessions – Cookie Names

- Application-based Cookies
- Custom cookie
- Generated by the target
- Can include any custom attributes required by the application
- Cookie name must be specified individually for each target group
- Don’t use AWSALB, AWSALBAPP, or AWSALBTG (reserved for use by the ELB)
- Application cookie
- Generated by the load balancer
- Cookie name is AWSALBAPP
- Duration-based Cookies
- Cookie generated by the load balancer
- Cookie name is AWSALB for ALB, AWSELB for CLB

<!-- Page 144 -->

# Cross-Zone Load Balancing

With Cross Zone Load Balancing:
each load balancer instance distributes evenly
across all registered instances in all AZ
50
50
10
10
Availability Zone 1
Without Cross Zone Load Balancing:
Requests are distributed in the instances of the
node of the Elastic Load Balancer
10
10
10
10
10
10
10
10
Availability Zone 2
50
50
25
25
Availability Zone 1
6.25
6.25
6.25
6.25
6.25
6.25
6.25
6.25
Availability Zone 2

<!-- Page 145 -->

# Cross-Zone Load Balancing

- Application Load Balancer
- Enabled by default (can be disabled at the Target Group level)
- No charges for inter AZ data
- Network Load Balancer & Gateway Load Balancer
- Disabled by default
- You pay charges ($) for inter AZ data if enabled
- Classic Load Balancer
- Disabled by default
- No charges for inter AZ data if enabled

<!-- Page 146 -->

# SSL/TLS - Basics

- An SSL Certificate allows traffic between your clients and your load balancer
to be encrypted in transit (in-flight encryption)
- SSL refers to Secure Sockets Layer, used to encrypt connections
- TLS refers to Transport Layer Security, which is a newer version
- Nowadays, TLS certificates are mainly used, but people still refer as SSL
- Public SSL certificates are issued by Certificate Authorities (CA)
- Comodo, Symantec, GoDaddy, GlobalSign, Digicert, Letsencrypt, etc…
- SSL certificates have an expiration date (you set) and must be renewed

<!-- Page 147 -->

# Load Balancer - SSL Certificates

Users
HTTPS (encrypted)
Over www
LOAD BALANCER
HTTP
Over private VPC
EC2
Instance
- The load balancer uses an X.509 certificate (SSL/TLS server certificate)
- You can manage certificates using ACM (AWS Certificate Manager)
- You can create upload your own certificates alternatively
- HTTPS listener:
- You must specify a default certificate
- You can add an optional list of certs to support multiple domains
- Clients can use SNI (Server Name Indication) to specify the hostname they reach
- Ability to specify a security policy to support older versions of SSL / TLS (legacy clients)

<!-- Page 148 -->

# SSL – Server Name Indication (SNI)

- SNI solves the problem of loading multiple SSL
certificates onto one web server (to serve
multiple websites)
- It’s a “newer” protocol, and requires the client
to indicate the hostname of the target server
in the initial SSL handshake
- The server will then find the correct
certificate, or return the default one
Note:
- Only works for ALB & NLB (newer
generation), CloudFront
- Does not work for CLB (older gen)
Target group for
www.mycorp.com
Target group for
Domain1.example.com
I would like
www.mycorp.com
ALB
Client
SSL Cert:
Domain1.example.com
Use the correct
SSL cert
SSL Cert:
www.mycorp.com
….

<!-- Page 149 -->

# Elastic Load Balancers – SSL Certificates

- Classic Load Balancer (v1)
- Support only one SSL certificate
- Must use multiple CLB for multiple hostname with multiple SSL certificates
- Application Load Balancer (v2)
- Supports multiple listeners with multiple SSL certificates
- Uses Server Name Indication (SNI) to make it work
- Network Load Balancer (v2)
- Supports multiple listeners with multiple SSL certificates
- Uses Server Name Indication (SNI) to make it work

<!-- Page 150 -->

# Connection Draining

- Feature naming
- Connection Draining – for CLB
- Deregistration Delay – for ALB & NLB
- Time to complete “in-flight requests” while the
instance is de-registering or unhealthy
- Stops sending new requests to the EC2
instance which is de-registering
- Between 1 to 3600 seconds (default: 300
seconds)
- Can be disabled (set value to 0)
- Set to a low value if your requests are short
waiting for existing
connections to complete
EC2 Instance
DRAINING
Users
ELB
new connections
established to all other instances
EC2 Instance
EC2 Instance

<!-- Page 151 -->

# What’s an Auto Scaling Group?

- In real-life, the load on your websites and application can change
- In the cloud, you can create and get rid of servers very quickly
- The goal of an Auto Scaling Group (ASG) is to:
- Scale out (add EC2 instances) to match an increased load
- Scale in (remove EC2 instances) to match a decreased load
- Ensure we have a minimum and a maximum number of EC2 instances running
- Automatically register new instances to a load balancer
- Re-create an EC2 instance in case a previous one is terminated (ex: if unhealthy)
- ASG are free (you only pay for the underlying EC2 instances)

<!-- Page 152 -->

# Auto Scaling Group in AWS

Auto Scaling Group
EC2
Instance
EC2
Instance
EC2
Instance
EC2
Instance
Minimum Capacity
EC2
Instance
EC2
Instance
Scale Out as Needed
Desired Capacity
Maximum Capacity
EC2
Instance

<!-- Page 153 -->

# Auto Scaling Group in AWS With Load Balancer

Users
Elastic Load Balancer
ELB can check the health of your EC2 instances!
Auto Scaling Group
EC2
Instance
EC2
Instance
EC2
Instance
EC2
Instance
EC2
Instance
EC2
Instance
EC2
Instance

<!-- Page 154 -->

# Auto Scaling Group Attributes

- A Launch Template (older “Launch Configurations” are deprecated)
- AMI + Instance Type
- EC2 User Data
- EBS Volumes
- Security Groups
- SSH Key Pair
- IAM Roles for your EC2 Instances
- Network + Subnets Information
- Load Balancer Information
- Min Size / Max Size / Initial Capacity
- Scaling Policies
ASG Launch Template
AMI
Security
Groups
Instance
Type
EBS Volumes
SSH Key Pair
IAM Role
…
VPC + Subnets
Load
Balancer

<!-- Page 155 -->

# Auto Scaling - CloudWatch Alarms & Scaling

- It is possible to scale an ASG based on CloudWatch alarms
- An alarm monitors a metric (such as Average CPU, or a custom metric)
- Metrics such as Average CPU are computed for the overall ASG instances
- Based on the alarm:
- We can create scale-out policies (increase the number of instances)
- We can create scale-in policies (decrease the number of instances)
Auto Scaling Group
EC2
Instance
EC2
Instance
EC2
Instance
trigger Scaling
EC2
Instance
EC2
Instance
CloudWatch
Alarm

<!-- Page 156 -->

# Auto Scaling Groups – Scaling Policies

- Dynamic Scaling
- Target Tracking Scaling
- Simple to set-up
- Example: I want the average ASG CPU to stay at around 40%
- Simple / Step Scaling
- When a CloudWatch alarm is triggered (example CPU > 70%), then add 2 units
- When a CloudWatch alarm is triggered (example CPU < 30%), then remove 1
- Scheduled Scaling
- Anticipate a scaling based on known usage patterns
- Example: increase the min capacity to 10 at 5 pm on Fridays

<!-- Page 157 -->

# Auto Scaling Groups – Scaling Policies

- Predictive scaling: continuously forecast load and schedule scaling ahead

<!-- Page 158 -->

# Good metrics to scale on

- CPUUtilization: Average CPU utilization
across your instances
- RequestCountPerTarget: to make sure
the number of requests per EC2
instances is stable
- Average Network In / Out (if you’re
application is network bound)
- Any custom metric (that you push
using CloudWatch)
Users
Application
Load Balancer
RequestCountPerTarget
Target Value: 3
Auto Scaling group

<!-- Page 159 -->

# Auto Scaling Groups - Scaling Cooldowns

- After a scaling activity happens, you are in
the cooldown period (default 300
seconds)
- During the cooldown period, the ASG will
not launch or terminate additional
instances (to allow for metrics to stabilize)
- Advice: Use a ready-to-use AMI to reduce
configuration time in order to be serving
request fasters and reduce the cooldown
period
Scaling Action
Occurs
Launch or
Teminate Instance
No
Default
Cooldown
in effect?
Yes
Ignore Action

<!-- Page 160 -->

# RDS, Aurora & ElastiCache

<!-- Page 161 -->

# Amazon RDS Overview

- RDS stands for Relational Database Service
- It’s a managed DB service for DB that use SQL as a query language.
- It allows you to create databases in the cloud that are managed by AWS
- Postgres
- MySQL
- MariaDB
- Oracle
- Microsoft SQL Server
- IBM DB2
- Aurora (AWS Proprietary database)

<!-- Page 162 -->

# DB on EC2

- RDS is a managed service:
- Automated provisioning, OS patching
- Continuous backups and restore to specific timestamp (Point in Time Restore)!
- Monitoring dashboards
- Read replicas for improved read performance
- Multi AZ setup for DR (Disaster Recovery)
- Maintenance windows for upgrades
- Scaling capability (vertical and horizontal)
- Storage backed by EBS
- BUT you can’t SSH into your instances
Advantage over using RDS versus deploying

<!-- Page 163 -->

# RDS – Storage Auto Scaling

- Helps you increase storage on your RDS DB instance
dynamically
- When RDS detects you are running out of free database
storage, it scales automatically
- Avoid manually scaling your database storage
- You have to set Maximum Storage Threshold (maximum limit
for DB storage)
- Automatically modify storage if:
- Free storage is less than 10% of allocated storage
- Low-storage lasts at least 5 minutes
- 6 hours have passed since last modification
- Useful for applications with unpredictable workloads
- Supports all RDS database engines
Application
Read/Write
Storage

<!-- Page 164 -->

# RDS Read Replicas for read scalability

- Up to 15 Read Replicas
- Within AZ, Cross AZ or
Cross Region
- Replication is ASYNC,
so reads are eventually
consistent
- Replicas can be
promoted to their own
DB
- Applications must
update the connection
string to leverage read
replicas
Application
reads
RDS DB
instance read
replica
writes
reads
reads
ASYNC
replication
ASYNC
replication
RDS DB
instance
RDS DB
instance read
replica

<!-- Page 165 -->

# RDS Read Replicas – Use Cases

- You have a production database
that is taking on normal load
- You want to run a reporting
application to run some analytics
- You create a Read Replica to run
the new workload there
- The production application is
unaffected
- Read replicas are used for SELECT
(=read) only kind of statements
(not INSERT, UPDATE, DELETE)
Production
Application
reads
reads
ASYNC
replication
RDS DB
instance
Reporting
Application
RDS DB
instance read
replica

<!-- Page 166 -->

# RDS Read Replicas – Network Cost

- In AWS there’s a network cost when data goes from one AZ to another
- For RDS Read Replicas within the same region, you don’t pay that fee
Same Region / Different AZ
us-east-1a
us-east-1b
VS
ASYNC
Replication
RDS DB
instance
Same Region
Free
Region/AZ
us-east-1a
RDS DB
instance read
replica
Region/AZ
eu-west-1b
ASYNC
Replication
RDS DB
instance
Cross-Region
$$$
RDS DB
instance read
replica

<!-- Page 167 -->

# RDS Multi AZ (Disaster Recovery)

- SYNC replication
- One DNS name – automatic app
failover to standby
- Increase availability
- Failover in case of loss of AZ, loss of
network, instance or storage failure
- No manual intervention in apps
- Not used for scaling
- Note: The Read Replicas be setup as
Multi AZ for Disaster Recovery (DR)
Application
writes
reads
One DNS name – automatic failover
RDS DB
instance standby
(AZ B)
SYNC
replication
RDS Master DB
instance (AZ A)

<!-- Page 168 -->

# RDS – From Single-AZ to Multi-AZ

- Zero downtime operation (no
need to stop the DB)
- Just click on “modify” for the
database
- The following happens internally:
- A snapshot is taken
- A new DB is restored from the
snapshot in a new AZ
- Synchronization is established
between the two databases
RDS DB
instance
Standby DB
SYNC
Replication
restore
snapshot
DB snapshot

<!-- Page 169 -->

# Mode disabled

- Managed Oracle and Microsoft SQL Server Database with OS and
database customization
- RDS: Automates setup, operation, and scaling of database in AWS
- Custom: access to the underlying database and OS so you can
User
apply
cstomizations
- Configure settings
- Install patches
- Enable native features
- Access the underlying EC2 Instance using SSH or SSM Session Manager
- De-activate Automation Mode to perform your customization, better
to take a DB snapshot before
- RDS vs. RDS Custom
- RDS: entire database and the OS to be managed by AWS
- RDS Custom: full admin access to the underlying OS and the database
SSH
EC2 Instance
RDS Custom
Automation

<!-- Page 170 -->

# Amazon Aurora

- Aurora is a proprietary technology from AWS (not open sourced)
- Postgres and MySQL are both supported as Aurora DB (that means your
drivers will work as if Aurora was a Postgres or MySQL database)
- Aurora is “AWS cloud optimized” and claims 5x performance improvement
over MySQL on RDS, over 3x the performance of Postgres on RDS
- Aurora storage automatically grows in increments of 10GB, up to 256 TB.
- Aurora can have up to 15 replicas and the replication process is faster than
MySQL (sub 10 ms replica lag)
- Failover in Aurora is instantaneous. It’s HA (High Availability) native.
- Aurora costs more than RDS (20% more) – but is more efficient

<!-- Page 171 -->

# Aurora High Availability and Read Scaling

- 6 copies of your data across 3 AZ:
- 4 copies out of 6 needed for writes
- 3 copies out of 6 need for reads
- Self healing with peer-to-peer replication
- Storage is striped across 100s of volumes
- One Aurora Instance takes writes (master)
- Automated failover for master in less than
30 seconds
- Master + up to 15 Aurora Read Replicas
serve reads
- Support for Cross Region Replication
AZ 2
AZ 1
W
R
R
AZ 3
R
R
R
Shared storage Volume
Replication + Self Healing + Auto Expanding

<!-- Page 172 -->

# Aurora DB Cluster

client
Writer Endpoint
Pointing to the master
Reader Endpoint
Connection Load Balancing
Auto Scaling
W
R
R
R
R
Shared storage Volume
Auto Expanding from 10G to 256 TB
R

<!-- Page 173 -->

# Features of Aurora

- Automatic fail-over
- Backup and Recovery
- Isolation and security
- Industry compliance
- Push-button scaling
- Automated Patching with Zero Downtime
- Advanced Monitoring
- Routine Maintenance
- Backtrack: restore data at any point of time without using backups

<!-- Page 174 -->

# Aurora Replicas - Auto Scaling

Client
Writer Endpoint
Many Requests
Reader Endpoint
CPU
Usage
Endpoint Extended
CPU
Usage
Replicas Auto Scaling
W
R
Shared Storage Volume
R
R
R

<!-- Page 175 -->

# Aurora – Custom Endpoints

- Define a subset of Aurora Instances as a Custom Endpoint
- Example: Run analytical queries on specific replicas
- The Reader Endpoint is generally not used after defining Custom Endpoints
Queries
Client
Writer Endpoint
W
Custom Endpoint
Reader Endpoint
db.r3.large
db.r3.large
db.r5.2xlarge
db.r5.2xlarge
R
R
R
R
Shared Storage Volume
Analytical Queries

<!-- Page 176 -->

# Aurora Serverless

- Automated database
instantiation and autoscaling based on actual
usage
- Good for infrequent,
intermittent or
unpredictable workloads
- No capacity planning
needed
- Pay per second, can be
more cost-effective
Client
Proxy Fleet
(managed by Aurora)
Shared storage Volume

<!-- Page 177 -->

# Global Aurora

us-east-1 - PRIMARY region
- Aurora Cross Region Read Replicas:
- Useful for disaster recovery
- Simple to put in place
- Aurora Global Database (recommended):
- 1 Primary Region (read / write)
- Up to 10 secondary (read-only) regions, replication lag is less
than 1 second
- Up to 16 Read Replicas per secondary region
- Helps for decreasing latency
- Promoting another region (for disaster recovery) has an RTO
of < 1 minute
- Typical cross-region replication takes less than 1 second
Applications
Read / Write
replication
eu-west-1 - SECONDARY region
Applications
Read Only

<!-- Page 178 -->

# Aurora Machine Learning

- Enables you to add ML-based predictions to
your applications via SQL
- Simple, optimized, and secure integration
between Aurora and AWS ML services
- Supported services
- Amazon SageMaker (use with any ML model)
- Amazon Comprehend (for sentiment analysis)
Application
SQL query
(Recommended products?)
query results
(red shirt, blue …)
Amazon Aurora
data
(user’s profile,
shopping history, …)
predictions
(red shirt,
blue pants, …)
Amazon
SageMaker
Amazon
Comprehend
- You don’t need to have ML experience
- Use cases: fraud detection, ads targeting,
sentiment analysis, product recommendations

<!-- Page 179 -->

# Babelfish for Aurora PostgreSQL

- Allows Aurora PostgreSQL to
understand commands targeted for
MS SQL Server
(e.g., T-SQL)
- Therefore Microsoft SQL Server
based applications can work on
Aurora PostgreSQL
- Requires no to little code changes
(using the same MS SQL Server client
driver)
- The same applications can be used
after a migration of your database
(using AWS SCT and DMS)
Application
Application
SQL Server Client Driver
PostgreSQL Driver
T-SQL
PL/pgSQL
T-SQL
Babelfish
PostgreSQL
migrate
Aurora PostgreSQL

<!-- Page 180 -->

# RDS Backups

- Automated backups:
- Daily full backup of the database (during the backup window)
- Transaction logs are backed-up by RDS every 5 minutes
- => ability to restore to any point in time (from oldest backup to 5 minutes ago)
- 1 to 35 days of retention, set 0 to disable automated backups
- Manual DB Snapshots
- Manually triggered by the user
- Retention of backup for as long as you want
- Trick: in a stopped RDS database, you will still pay for storage. If you plan on
stopping it for a long time, you should snapshot & restore instead

<!-- Page 181 -->

# Aurora Backups

- Automated backups
- 1 to 35 days (cannot be disabled)
- point-in-time recovery in that timeframe
- Manual DB Snapshots
- Manually triggered by the user
- Retention of backup for as long as you want

<!-- Page 182 -->

# RDS & Aurora Restore options

- Restoring a RDS / Aurora backup or a snapshot creates a new database
- Restoring MySQL RDS database from S3
- Create a backup of your on-premises database
- Store it on Amazon S3 (object storage)
- Restore the backup file onto a new RDS instance running MySQL
- Restoring MySQL Aurora cluster from S3
- Create a backup of your on-premises database using Percona XtraBackup
- Store the backup file on Amazon S3
- Restore the backup file onto a new Aurora cluster running MySQL

<!-- Page 183 -->

# Aurora Database Cloning

- Create a new Aurora DB Cluster from an
existing one
- Faster than snapshot & restore
- Uses copy-on-write protocol
- Initially, the new DB cluster uses the same data
volume as the original DB cluster (fast and efficient
– no copying is needed)
- When updates are made to the new DB cluster
data, then additional storage is allocated and data is
copied to be separated
- Very fast & cost-effective
- Useful to create a “staging” database from a
“production” database without impacting the
production database
clone
Production Aurora
Staging Aurora

<!-- Page 184 -->

# RDS & Aurora Security

- At-rest encryption:
- Database master & replicas encryption using AWS KMS – must be defined as launch time
- If the master is not encrypted, the read replicas cannot be encrypted
- To encrypt an un-encrypted database, go through a DB snapshot & restore as encrypted
- In-flight encryption: TLS-ready by default, use the AWS TLS root certificates client-side
- IAM Authentication: IAM roles to connect to your database (instead of username/pw)
- Security Groups: Control Network access to your RDS / Aurora DB
- No SSH available except on RDS Custom
- Audit Logs can be enabled and sent to CloudWatch Logs for longer retention

<!-- Page 185 -->

# Amazon RDS Proxy

- Fully managed database proxy for RDS
- Allows apps to pool and share DB connections
established with the database
- Improving database efficiency by reducing the stress
on database resources (e.g., CPU, RAM) and minimize
open connections (and timeouts)
- Serverless, autoscaling, highly available (multi-AZ)
- Reduced RDS & Aurora failover time by up 66%
- Supports RDS (MySQL, PostgreSQL, MariaDB, MS
SQL Server) and Aurora (MySQL, PostgreSQL)
- No code changes required for most apps
- Enforce IAM Authentication for DB, and securely
store credentials in AWS Secrets Manager
- RDS Proxy is never publicly accessible (must be
accessed from VPC)
VPC
Lambda functions
…
IAM
Authentication
Private subnet
RDS Proxy
RDS DB
Instance

<!-- Page 186 -->

# Amazon ElastiCache Overview

- The same way RDS is to get managed Relational Databases…
- ElastiCache is to get managed Redis or Memcached
- Caches are in-memory databases with really high performance, low
latency
- Helps reduce load off of databases for read intensive workloads
- Helps make your application stateless
- AWS takes care of OS maintenance / patching, optimizations, setup,
configuration, monitoring, failure recovery and backups
- Using ElastiCache involves heavy application code changes

<!-- Page 187 -->

# Solution Architecture - DB Cache

- Applications queries
ElastiCache, if not
available, get from RDS
and store in ElastiCache.
- Helps relieve load in RDS
- Cache must have an
invalidation strategy to
make sure only the most
current data is used in
there.
Amazon
ElastiCache
Cache hit
application
Cache miss
Read from DB
Write to cache
Amazon
RDS
ElastiCache

<!-- Page 188 -->

# Solution Architecture – User Session Store

Write session
application
Amazon
ElastiCache
User
- User logs into any of the
application
- The application writes
the session data into
ElastiCache
- The user hits another
instance of our
application
- The instance retrieves the
data and the user is
already logged in
Retrieve session
application
application
ElastiCache

<!-- Page 189 -->

# ElastiCache – Redis vs Memcached

REDIS
- Multi AZ with Auto-Failover
- Read Replicas to scale reads and
have high availability
- Data Durability using AOF
persistence
- Backup and restore features
- Supports Sets and Sorted Sets
Replication
MEMCACHED
- Multi-node for partitioning of
data (sharding)
- No high availability (replication)
- Non persistent
- Backup and restore (Serverless)
- Multi-threaded architecture
+
sharding

<!-- Page 190 -->

# ElastiCache – Cache Security

- ElastiCache supports IAM Authentication for Redis
- IAM policies on ElastiCache are only used for
AWS API-level security
- Redis AUTH
- You can set a “password/token” when you create a
Redis cluster
- This is an extra level of security for your cache (on top
of security groups)
- Support SSL in flight encryption
- Memcached
- Supports SASL-based authentication (advanced)
EC2 Security group
EC2
Client
SSL encryption
Redis AUTH
Redis Security group

<!-- Page 191 -->

# Patterns for ElastiCache

- Lazy Loading: all the read data is
cached, data can become stale in
cache
- Write Through: Adds or update
data in the cache when written
to a DB (no stale data)
- Session Store: store temporary
session data in a cache (using
TTL features)
Amazon
ElastiCache
Cache hit
application
Cache miss
Read from DB
- Quote: There are only two hard
things in Computer Science: cache
invalidation and naming things
Write to cache
Lazy Loading illustrated
Amazon
RDS

<!-- Page 192 -->

# ElastiCache – Redis Use Case

- Gaming Leaderboards are computationally complex
- Redis Sorted sets guarantee both uniqueness and element ordering
- Each time a new element added, it’s ranked in real time, then added in
correct order
ElastiCache
for Redis
1
2
ElastiCache
for Redis
Clients
ElastiCache
for Redis
3
Real-time Leaderboard

<!-- Page 193 -->

# Amazon Route 53

<!-- Page 194 -->

# What is DNS?

- Domain Name System which translates the human friendly hostnames
into the machine IP addresses
- www.google.com => 172.217.18.36
- DNS is the backbone of the Internet
- DNS uses hierarchical naming structure
.com
example.com
www.example.com
api.example.com

<!-- Page 195 -->

# DNS Terminologies

- Domain Registrar: Amazon Route 53, GoDaddy, …
- DNS Records: A, AAAA, CNAME, NS, …
- Zone File: contains DNS records
- Name Server: resolves DNS queries (Authoritative or Non-Authoritative)
- Top Level Domain (TLD): .com, .us, .in, .gov, .org, …
- Second Level Domain (SLD): amazon.com, google.com, …
URL
http://api.www.example.com.
TLD
Protocol
SLD
Sub Domain
FQDN (Fully Qualified Domain Name)
Root

<!-- Page 196 -->

# How DNS Works

Web Server
(example.com)
(IP: 9.10.11.12)
pl
exam
.2
NS 1
m
o
.c
example.com?
TTL
Web Browser
You want to access
example.com
Managed by ICANN
?
e.com
Root DNS Server
.3.4
example.com?
9.10.11.12
TTL
example.com NS 5.6.7.8
Local DNS Server
Assigned and Managed by
your company or assigned by
your ISP dynamically
Managed by IANA
(Branch of ICANN)
exam
p
exam
p
le.co
m IP
TLD DNS Server
(.com)
le.co
9.10
m?
.11.
1
2
Managed by Domain Registrar
(e.g., Amazon Registrar, Inc.)
SLD DNS Server
(example.com)

<!-- Page 197 -->

# Amazon Route 53

- A highly available, scalable, fully
managed and Authoritative DNS
- Authoritative = the customer (you)
can update the DNS records
- Route 53 is also a Domain Registrar
- Ability to check the health of your
resources
- The only AWS service which
provides 100% availability SLA
- Why Route 53? 53 is a reference to
the traditional DNS port
example.com?
Client
54.22.33.44
AWS Cloud
Public IP
54.22.33.44
EC2 Instance
Amazon
Route 53

<!-- Page 198 -->

# Route 53 – Records

- How you want to route traffic for a domain
- Each record contains:
- Domain/subdomain Name – e.g., example.com
- Record Type – e.g., A or AAAA
- Value – e.g., 12.34.56.78
- Routing Policy – how Route 53 responds to queries
- TTL – amount of time the record cached at DNS Resolvers
- Route 53 supports the following DNS record types:
- (must know) A / AAAA / CNAME / NS
- (advanced) CAA / DS / MX / NAPTR / PTR / SOA / TXT / SPF / SRV

<!-- Page 199 -->

# Route 53 – Record Types

- A – maps a hostname to IPv4
- AAAA – maps a hostname to IPv6
- CNAME – maps a hostname to another hostname
- The target is a domain name which must have an A or AAAA record
- Can’t create a CNAME record for the top node of a DNS namespace (Zone
Apex)
- Example: you can’t create for example.com, but you can create for
www.example.com
- NS – Name Servers for the Hosted Zone
- Control how traffic is routed for a domain

<!-- Page 200 -->

# Route 53 – Hosted Zones

- A container for records that define how to route traffic to a domain and
its subdomains
- Public Hosted Zones – contains records that specify how to route traffic
on the Internet (public domain names)
application1.mypublicdomain.com
- Private Hosted Zones – contain records that specify how you route
traffic within one or more VPCs (private domain names)
application1.company.internal
- You pay $0.50 per month per hosted zone

<!-- Page 201 -->

# Route 53 – Public vs. Private Hosted Zones

Public Hosted Zone
Private Hosted Zone
example.com?
VPC
S3 Bucket
EC2 Instance
Amazon
(Public IP)
CloudFront
Application
Load Balancer
EC2 Instance
10.0.0.35
.10
10
.0.
0
.int
ern
i.ex
am
pl e
ap
VPC
db.example.internal?
Private Hosted Zone
Public Hosted Zone
al?
Client
54.22.33.44
EC2 Instance
(webapp.example.internal) (api.example.internal)
(Private IP)
(Private IP)
DB Instance
(db.example.internal)
(Private IP)

<!-- Page 202 -->

# Route 53 – Records TTL (Time To Live)

- High TTL – e.g., 24 hr
- Less traffic on Route 53
- Possibly outdated records
equest
?
D NS R
le.com
p
m
a
x
e
.
myapp
- Low TTL – e.g., 60 sec.
- More traffic on Route 53 ($$)
- Records are outdated for less
time
- Easy to change records
- Except for Alias records, TTL
is mandatory for each DNS
record
A 12.34.56.78
(with TTL)
TTL
Client
Will cache the result for
The TTL of the record
HT T
HT T
P Re
P Re
Amazon
Route 53
ques
t
spon
se
Web Server

<!-- Page 203 -->

# CNAME vs Alias

- AWS Resources (Load Balancer, CloudFront...) expose an AWS hostname:
- lb1-1234.us-east-2.elb.amazonaws.com and you want myapp.mydomain.com
- CNAME:
- Points a hostname to any other hostname. (app.mydomain.com => blabla.anything.com)
- ONLY FOR NON ROOT DOMAIN (aka. something.mydomain.com)
- Alias:
- Points a hostname to an AWS Resource (app.mydomain.com => blabla.amazonaws.com)
- Works for ROOT DOMAIN and NON ROOT DOMAIN (aka mydomain.com)
- Free of charge
- Native health check

<!-- Page 204 -->

# Load Balancer

Amazon
Route 53
Route 53 – Alias Records
- Maps a hostname to an AWS resource
- An extension to DNS functionality
Alias Record (Enabled)
- Automatically recognizes changes in the
Record Name Type
Value
resource’s IP addresses
example.com A
MyALB-123456789.useast• Unlike CNAME, it can be used for the top node
1.elb.amazonaws.com
of a DNS namespace (Zone Apex), e.g.:
example.com
- Alias Record is always of type A/AAAA for
MyALB-123456789.us-east-1.elb.amazonaws.com
AWS resources (IPv4 / IPv6)
AWS-Managed
(IP Addresses might change)
- You can’t set the TTL
Application

<!-- Page 205 -->

# Route 53 – Alias Records Targets

- Elastic Load Balancers
- CloudFront Distributions
Elastic
Load Balancer
- API Gateway
- Elastic Beanstalk environments
- S3 Websites
Elastic Beanstalk
- VPC Interface Endpoints
- Global Accelerator accelerator
- Route 53 record in the same hosted zone
Amazon
CloudFront
Amazon
API Gateway
S3 Websites
VPC Interface
Endpoints
Global Accelerator
- You cannot set an ALIAS record for an EC2 DNS name
Route 53 Record
(same Hosted Zone)

<!-- Page 206 -->

# Route 53 – Routing Policies

- Define how Route 53 responds to DNS queries
- Don’t get confused by the word “Routing”
- It’s not the same as Load balancer routing which routes the traffic
- DNS does not route any traffic, it only responds to the DNS queries
- Route 53 Supports the following Routing Policies
- Simple
- Weighted
- Failover
- Latency based
- Geolocation
- Multi-Value Answer
- Geoproximity (using Route 53 Traffic Flow feature)

<!-- Page 207 -->

# Routing Policies – Simple

- Typically, route traffic to a single
resource
- Can specify multiple values in the
same record
- If multiple values are returned, a
random one is chosen by the client
- When Alias enabled, specify only
one AWS resource
- Can’t be associated with Health
Checks
Single Value
foo.example.com
A 11.22.33.44
Client
Multiple Value
foo.example.com
Client
chooses
a random value
Amazon
Route 53
A 11.22.33.44
A 55.66.77.88
A 99.11.22.33
Amazon
Route 53

<!-- Page 208 -->

# Routing Policies – Weighted

- Control the % of the requests that go to each
specific resource
- Assign each record a relative weight:
- 𝑡𝑟𝑎𝑓𝑓𝑖𝑐 (%) =
Weight: 70
%
70
!"#$%& ()* + ,-".#(#. *".)*/
012 )( +33 &%" 4"#$%&, ()* +33 *".)*/,
- Weights don’t need to sum up to 100
- DNS records must have the same name and type
- Can be associated with Health Checks
- Use cases: load balancing between regions, testing
new application versions…
- Assign a weight of 0 to a record to stop sending
traffic to a resource
- If all records have weight of 0, then all records will
be returned equally
20%
Amazon
Route 53
Weight: 20
10
%
Weight: 10

<!-- Page 209 -->

# Routing Policies – Latency-based

- Redirect to the resource that
has the least latency close to us
- Super helpful when latency for
users is a priority
- Latency is based on traffic
between users and AWS
Regions
- Germany users may be
directed to the US (if that’s the
lowest latency)
- Can be associated with Health
Checks (has a failover
capability)
ALB
(us-east-1)
ALB
(ap-southeast-1)

<!-- Page 210 -->

# EC2 Instance

- HTTP Health Checks are only for public
resources
- Health Check => Automated DNS Failover:
1. Health checks that monitor an endpoint
(application, server, other AWS resource)
2. Health checks that monitor other health
checks (Calculated Health Checks)
3. Health checks that monitor CloudWatch
Alarms (full control !!) – e.g., throttles of
DynamoDB, alarms on RDS, custom metrics,
… (helpful for private resources)
- Health Checks are integrated with CW
metrics
Amazon Route 53
Route 53 – Health Checks
DNS Record
(latency, geoproximity, …)
Health Check
Health Check
us-east-1
eu-west-1
ALB
ALB
Auto Scaling group
Auto Scaling group

<!-- Page 211 -->

- About 15 global health checkers will check the
endpoint health
Health Checker
(sa-east-1)
st
ue
req
TP lth
HT /hea
to
Health Checker
(us-west-1)
e
od
0c
- Health Checks pass only when the endpoint
responds with the 2xx and 3xx status codes
- Health Checks can be setup to pass / fail based on
the text in the first 5120 bytes of the response
- Configure you router/firewall to allow incoming
requests from Route 53 Health Checkers
20
- Healthy/Unhealthy Threshold – 3 (default)
- Interval – 30 sec (can set to 10 sec – higher cost)
- Supported protocol: HTTP, HTTPS and TCP
- If > 18% of health checkers report the endpoint is
healthy, Route 53 considers it Healthy. Otherwise, it’s
Unhealthy
- Ability to choose which locations you want Route 53 to
use
Health Checker
(us-east-1)
eu-west-1
ALB
Health Checks – Monitor an Endpoint
Must allow incoming
requests from Route 53
Health Checkers IP
address range
Auto Scaling group
EC2 Instance
https://ip-ranges.amazonaws.com/ip-ranges.json

<!-- Page 212 -->

# Route 53 – Calculated Health Checks

- Combine the results of multiple Health
Checks into a single Health Check
- You can use OR, AND, or NOT
- Can monitor up to 256 Child Health Checks
- Specify how many of the health checks need
to pass to make the parent pass
- Usage: perform maintenance to your website
without causing all health checks to fail
Amazon Route 53
Health Check
(Parent)
Health Check Health Check Health Check
(Child)
(Child)
(Child)
monitor
EC2 Instance
monitor
EC2 Instance
monitor
EC2 Instance

<!-- Page 213 -->

# Health Checks – Private Hosted Zones

- Route 53 health checkers are outside the
VPC
- They can’t access private endpoints
(private VPC or on-premises resource)
- You can create a CloudWatch Metric and
associate a CloudWatch Alarm, then
create a Health Check that checks the
alarm itself
VPC
Private subnet
Health Checker
(us-east-1)
monitor
monitor
CloudWatch
Alarm

<!-- Page 214 -->

# Routing Policies – Failover (Active-Passive)

Health Check
(mandatory)
DNS Requests
EC2 Instance
(Primary)
Failover
Client
Amazon
Route 53
EC2 Instance
(Secondary – Disaster Recovery)

<!-- Page 215 -->

# Routing Policies – Geolocation

A 11.22.33.44
- Different from Latency-based!
- This routing is based on user location
- Specify location by Continent, Country
or by US State (if there’s overlapping,
most precise location selected)
- Should create a “Default” record (in
case there’s no match on location)
- Use cases: website localization, restrict
content distribution, load balancing, …
- Can be associated with Health Checks
Default
A 99.11.22.33
A 55.66.77.88

<!-- Page 216 -->

# Routing Policies – Geoproximity

- Route traffic to your resources based on the geographic location of users and
resources
- Ability to shift more traffic to resources based on the defined bias
- To change the size of the geographic region, specify bias values:
- To expand (1 to 99) – more traffic to the resource
- To shrink (-1 to -99) – less traffic to the resource
- Resources can be:
- AWS resources (specify AWS region)
- Non-AWS resources (specify Latitude and Longitude)
- You must use Route 53 Traffic Flow to use this feature

<!-- Page 217 -->

# Routing Policies – Geoproximity

us-west-1
Bias: 0
us-east-1
Bias: 0

<!-- Page 218 -->

# Routing Policies – Geoproximity

us-west-1
Bias: 0
us-east-1
Bias: 50
Higher bias in us-east-1

<!-- Page 219 -->

# (1.2.3.4)

- Routing is based on clients’ IP addresses
- You provide a list of CIDRs for your clients
and the corresponding endpoints/locations
(user-IP-to-endpoint mappings)
- Use cases: Optimize performance, reduce
network costs…
- Example: route end users from a particular
ISP to a specific endpoint
User B
(200.5.4.100)
(203.0.113.56)
Route 53
CIDR Collection
Locations
CIDR blocks
location-1
203.0.113.0/24
location-2
200.5.4.0/24
Records
Record Name
Value
IP-based
example.com
1.2.3.4
location-1
example.com
5.6.7.8
location-2
EC2 Instance
(5.6.7.8)
User A
Routing Policies – IP-based Routing
EC2 Instance

<!-- Page 220 -->

# Routing Policies – Multi-Value

- Use when routing traffic to multiple resources
- Route 53 return multiple values/resources
- Can be associated with Health Checks (return only values for healthy resources)
- Up to 8 healthy records are returned for each Multi-Value query
- Multi-Value is not a substitute for having an ELB

<!-- Page 221 -->

# Domain Registar vs. DNS Service

- You buy or register your domain name with a Domain Registrar typically by
paying annual charges (e.g., GoDaddy, Amazon Registrar Inc., …)
- The Domain Registrar usually provides you with a DNS service to manage
your DNS records
- But you can use another DNS service to manage your DNS records
- Example: purchase the domain from GoDaddy and use Route 53 to manage
your DNS records
purchase
example.com
manage DNS records
User
Amazon
Route 53

<!-- Page 222 -->

# GoDaddy as Registrar & Route 53 as DNS Service

Amazon
Route 53
Public Hosted Zone
stephanetheteacher.com

<!-- Page 223 -->

# 3rd Party Registrar with Amazon Route 53

- If you buy your domain on a 3rd party registrar, you can still use Route 53
as the DNS Service provider
1. Create a Hosted Zone in Route 53
2. Update NS Records on 3rd party website to use Route 53 Name
Servers
- Domain Registrar != DNS Service
- But every Domain Registrar usually comes with some DNS features

<!-- Page 224 -->

# Route 53 – Hybrid DNS

- By default, Route 53 Resolver
automatically answers DNS queries for:
- Local domain names for EC2 instances
- Records in Private Hosted Zones
- Records in public Name Servers
- Hybrid DNS – resolving DNS queries
between VPC (Route 53 Resolver) and
your networks (other DNS Resolvers)
- Networks can be:
- VPC itself / Peered VPC
- On-premises Network (connected through
Direct Connect or AWS VPN)
Public Name Server
Region
VPC
Private Hosted Zone
Route 53
Resolver
EC2 Instance
(ec2-192-0-2-44.compute-1.amazonaws.com)

<!-- Page 225 -->

# Route 53 – Resolver Endpoints

- Inbound Endpoint – allows your DNS Resolvers to resolve domain names for
AWS resources (e.g., EC2 instances) and records in Private Hosted Zones
us-east-1
On-Premises Data Center
Private Hosted Zone
(aws.private)
VPC
up
look
DNS Query
rivate?
app.aws.p
Private Subnet
DNS Resolvers
Route 53
Resolver
EC2 Instance
(onpremise.private)
Resolver
Inbound Endpoint
DNS Query
app.aws.private?
(app.aws.private)
VPN or DX connection
Server
(web.onpremise.private)

<!-- Page 226 -->

# Route 53 – Resolver Endpoints

- Outbound Endpoint
- Route 53 Resolver forwards DNS queries to your DNS Resolvers
us-east-1
VPC
Private Subnet
uer y
Q
S
N
D
ivate?
r
p
.
e
s
i
nprem
o
.
b
e
w
Route 53
Resolver
On-Premises Data Center
Private Hosted Zone
(aws.private)
EC2 Instance
(app.aws.private)
uer y
DNS Q .private?
i se
nprem
o
.
b
e
w
DNS Resolvers
(onpremise.private)
Resolver
Outbound Endpoint
VPN or DX connection
Server
(web.onpremise.private)

<!-- Page 227 -->

# Classic Solutions Architecture

<!-- Page 228 -->

# Section Introduction

- These solutions architectures are the best part of this course
- Let’s understand how all the technologies we’ve seen work together
- This is a section you need to be 100% comfortable with
- We’ll see the progression of a Solution’s architect mindset through many
sample case studies:
- WhatIsTheTime.Com
- MyClothes.Com
- MyWordPress.Com
- Instantiating applications quickly
- Beanstalk

<!-- Page 229 -->

# Stateless Web App: WhatIsTheTime.com

- WhatIsTheTime.com allows people to know what time it is
- We don’t need a database
- We want to start small and can accept downtime
- We want to fully scale vertically and horizontally, no downtime
- Let’s go through the Solutions Architect journey for this app
- Let’s see how we can proceed!

<!-- Page 230 -->

# Starting simple

Elastic IP Address
What time is it?
User
5:30 pm!
Public EC2
Stateless web app: What time is it?

<!-- Page 231 -->

# Scaling vertically

What time is it?
Elastic IP Address
7:30 pm!
What time is it?
Downtime while upgrading to M5
User
5:30 pm!
What time is it?
6:30 pm!
Public EC2
Stateless web app: What time is it?

<!-- Page 232 -->

# Scaling horizontally

What time is it?
7:30 pm!
What time is it?
User
5:30 pm!
What time is it?
6:30 pm!
Stateless web app: What time is it?

<!-- Page 233 -->

# Scaling horizontally

DNS Query
For api.whatisthetime.com
A Record
TTL 1 hour
What time is it?
7:30 pm!
What time is it?
5:30 pm!
What time is it?
6:30 pm!
Public EC2 instance,
No Elastic IP
Stateless web app: What time is it?

<!-- Page 234 -->

# Scaling horizontally, adding and removing instances

DNS Query
For api.whatisthetime.com
A Record
TTL 1 hour
What time is it?
INSTANCE IS GONE!
7:30 pm!
What time is it?
5:30 pm!
What time is it?
6:30 pm!
Public EC2 instance,
No Elastic IP
Stateless web app: What time is it?

<!-- Page 235 -->

# Scaling horizontally, with a load balancer

What time is it?
Availability zone 1
Availability zone 1
Restricted
Security groups rules
DNS Query
For api.whatisthetime.com
Alias Record
ELB +
Health Checks
Private
EC2 instances
Stateless web app: What time is it?

<!-- Page 236 -->

# Scaling horizontally, with an auto-scaling group

What time is it?
Availability zone 1
Availability zone 1
Auto Scaling group
DNS Query
For api.whatisthetime.com
Alias Record
ELB +
Health Checks
Private
EC2 instances
Stateless web app: What time is it?

<!-- Page 237 -->

# Making our app multi-AZ

Auto Scaling group
What time is it?
Availability zone 1 to 3
Availability zone 1
Availability zone 2
DNS Query
For api.whatisthetime.com
Alias Record
ELB +
Health Checks
+ Multi AZ
Availability zone 3
Stateless web app: What time is it?

<!-- Page 238 -->

# Minimum 2 AZ => Let’s reserve capacity

Auto Scaling group
Availability zone 1 to 3
Availability zone 1
Availability zone 2
DNS Query
For api.whatisthetime.com
Alias Record
ELB +
Health Checks
+ Multi AZ
Minimum capacity
= reserved instances
= cost savings

<!-- Page 239 -->

# In this lecture we’ve discussed…

- Public vs Private IP and EC2 instances
- Elastic IP vs Route 53 vs Load Balancers
- Route 53 TTL, A records and Alias Records
- Maintaining EC2 instances manually vs Auto Scaling Groups
- Multi AZ to survive disasters
- ELB Health Checks
- Security Group Rules
- Reservation of capacity for costing savings when possible

<!-- Page 240 -->

# Stateful Web App: MyClothes.com

- MyClothes.com allows people to buy clothes online.
- There’s a shopping cart
- Our website is having hundreds of users at the same time
- We need to scale, maintain horizontal scalability and keep our web
application as stateless as possible
- Users should not lose their shopping cart
- Users should have their details (address, etc) in a database
- Let’s see how we can proceed!

<!-- Page 241 -->

# Stateful Web App: MyClothes.com

Auto Scaling group
Availability zone 1
Multi AZ
Availability zone 2
Availability zone 3

<!-- Page 242 -->

# Introduce Stickiness (Session Affinity)

Auto Scaling group
Availability zone 1
Multi AZ
Availability zone 2
ELB Stickiness
Availability zone 3
Stateful Web App: MyClothes.com

<!-- Page 243 -->

# Introduce User Cookies

Auto Scaling group
Availability zone 1
Multi AZ
Send shopping cart
content in Web Cookies
Availability zone 2
Availability zone 3
Stateless
HTTP requests are heavier
Security risk
(cookies can be altered)
Cookies must be validated
Cookies must be less than 4KB
Stateful Web App: MyClothes.com

<!-- Page 244 -->

# Introduce Server Session

Auto Scaling group
ElastiCache
Availability zone 1
Multi AZ
Send session_id in
Web Cookies
Availability zone 2
Store / retrieve
session data
Availability zone 3
Amazon DynamoDB
(alternative)
Stateful Web App: MyClothes.com

<!-- Page 245 -->

# Storing User Data in a database

Auto Scaling group
ElastiCache
Availability zone 1
Multi AZ
Availability zone 2
Store / retrieve user data
(address, name, etc)
Availability zone 3
Amazon RDS
Stateful Web App: MyClothes.com

<!-- Page 246 -->

# Scaling Reads

Auto Scaling group
ElastiCache
Availability zone 1
Multi AZ
Availability zone 2
RDS
Master
(writes)
replication
Availability zone 3
RDS
Read Replicas
Stateful Web App: MyClothes.com

<!-- Page 247 -->

# Scaling Reads (Alternative) – Lazy Loading

ElastiCache
Auto Scaling group
Availability zone 1
cache
Multi AZ
hit
Availability zone 2
Read from cache
Read/write
Availability zone 3
RDS
Stateful Web App: MyClothes.com

<!-- Page 248 -->

# Multi AZ – Survive disasters

Auto Scaling group
ElastiCache
Multi AZ
Availability zone 1
Multi AZ
Availability zone 2
Availability zone 3
RDS
Multi AZ
Stateful Web App: MyClothes.com

<!-- Page 249 -->

# Security Groups

Auto Scaling group
Availability zone 1
Restrict traffic to ElastiCache
Security group from the
EC2 security group
ElastiCache
Multi AZ
Open HTTP / HTTPS
to 0.0.0.0/0
Availability zone 2
RDS
Availability zone 3
Restrict traffic to EC2
Security group from the LB
Restrict traffic to RDS
Security group from the
EC2 security group
Stateful Web App: MyClothes.com

<!-- Page 250 -->

# 3-tier architectures for web applications

- ELB sticky sessions
- Web clients for storing cookies and making our web app stateless
- ElastiCache
- For storing sessions (alternative: DynamoDB)
- For caching data from RDS
- Multi AZ
- RDS
- For storing user data
- Read replicas for scaling reads
- Multi AZ for disaster recovery
- Tight Security with security groups referencing each other
In this lecture we’ve discussed…

<!-- Page 251 -->

# Stateful Web App: MyWordPress.com

- We are trying to create a fully scalable WordPress website
- We want that website to access and correctly display picture uploads
- Our user data, and the blog content should be stored in a MySQL database.
- Let’s see how we can achieve this!

<!-- Page 252 -->

# RDS layer

Auto Scaling group
Availability zone 1
Multi AZ
Availability zone 2
Availability zone 3
RDS
Multi AZ
Stateful Web App: MyWordPress.com

<!-- Page 253 -->

# Scaling with Aurora: Multi AZ & Read Replicas

Auto Scaling group
Availability zone 1
Multi AZ
Availability zone 2
Availability zone 3
Aurora MySQL
Multi AZ
Read Replicas
Stateful Web App: MyWordPress.com

<!-- Page 254 -->

# Storing images with EBS

Multi AZ
Availability zone 1
Send image
Amazon EBS
Volume
Stateful Web App: MyWordPress.com

<!-- Page 255 -->

# Storing images with EBS

Availability zone 1
Multi AZ
Amazon EBS
Volume
Send image
Availability zone 2
Amazon EBS
Volume
Stateful Web App: MyWordPress.com

<!-- Page 256 -->

# Storing images with EFS

Availability zone 1
Multi AZ
ENI
Send image
Availability zone 2
EFS
ENI
Stateful Web App: MyWordPress.com

<!-- Page 257 -->

# In this lecture we’ve discussed…

- Aurora Database to have easy Multi-AZ and Read-Replicas
- Storing data in EBS (single instance application)
- Vs Storing data in EFS (distributed application)

<!-- Page 258 -->

# Instantiating Applications quickly

- When launching a full stack (EC2, EBS, RDS), it can take time to:
- Install applications
- Insert initial (or recovery) data
- Configure everything
- Launch the application
- We can take advantage of the cloud to speed that up!

<!-- Page 259 -->

# Instantiating Applications quickly

- EC2 Instances:
- Use a Golden AMI: Install your applications, OS dependencies etc.. beforehand
and launch your EC2 instance from the Golden AMI
- Bootstrap using User Data: For dynamic configuration, use User Data scripts
- Hybrid: mix Golden AMI and User Data (Elastic Beanstalk)
- RDS Databases:
- Restore from a snapshot: the database will have schemas and data ready!
- EBS Volumes:
- Restore from a snapshot: the disk will already be formatted and have data!

<!-- Page 260 -->

# Typical architecture: Web App 3-tier

Route 53
Auto Scaling group
ElastiCache
Availability zone 1
Multi AZ
Availability zone 2
Store / retrieve
session data
+ Cached data
ELB
Availability zone 3
Amazon RDS
Read / write data
PUBLIC SUBNET
PRIVATE SUBNET
DATA SUBNET

<!-- Page 261 -->

# Developer problems on AWS

- Managing infrastructure
- Deploying Code
- Configuring all the databases, load balancers, etc
- Scaling concerns
- Most web apps have the same architecture (ALB + ASG)
- All the developers want is for their code to run!
- Possibly, consistently across different applications and environments

<!-- Page 262 -->

# Elastic Beanstalk – Overview

- Elastic Beanstalk is a developer centric view of deploying an application
on AWS
- It uses all the component’s we’ve seen before: EC2, ASG, ELB, RDS, …
- Managed service
- Automatically handles capacity provisioning, load balancing, scaling, application
health monitoring, instance configuration, …
- Just the application code is the responsibility of the developer
- We still have full control over the configuration
- Beanstalk is free but you pay for the underlying instances

<!-- Page 263 -->

# Elastic Beanstalk – Components

- Application: collection of Elastic Beanstalk components (environments,
versions, configurations, …)
- Application Version: an iteration of your application code
- Environment
- Collection of AWS resources running an application version (only one application
version at a time)
- Tiers: Web Server Environment Tier & Worker Environment Tier
- You can create multiple environments (dev, test, prod, …)
update version
Create
Application
Upload
Version
Launch
Environment
deploy new version
Manage
Environment

<!-- Page 264 -->

# Elastic Beanstalk – Supported Platforms

- Go
- Java SE
- Java with Tomcat
- .NET Core on Linux
- .NET on Windows Server
- Node.js
- PHP
- Python
- Ruby
- Packer Builder
- Single Container Docker
- Multi-container Docker
- Preconfigured Docker

<!-- Page 265 -->

# Web Server Tier vs. Worker Tier

Web Environment
(myapp.us-east-1.elasticbeanstalk.com)
Availability Zone 1
ELB
Worker Environment
SQS Queue
Availability Zone 2
Availability Zone 1
SQS message
Security Group
pull
messages
SQS message
Security Group
Auto Scaling group
EC2 Instance
(Web Server)
Availability Zone 2
Auto Scaling group
EC2 Instance
(Web Server)
EC2 Instance
(Worker)
EC2 Instance
(Worker)
- Scale based on the number of SQS messages
- Can push messages to SQS queue from
another Web Server Tier

<!-- Page 266 -->

# Elastic Beanstalk Deployment Modes

Single Instance
Great for dev
High Availability with Load Balancer
Great for prod
Availability Zone 1
Availability Zone 1
Elastic IP
ALB
Availability Zone 2
Auto Scaling Group
EC2 Instance
EC2 Instance
EC2 Instance
RDS Master
RDS Master
RDS Standby

<!-- Page 267 -->

# Amazon S3

<!-- Page 268 -->

# Section introduction

- Amazon S3 is one of the main building blocks of AWS
- It’s advertised as ”infinitely scaling” storage
- Many websites use Amazon S3 as a backbone
- Many AWS services use Amazon S3 as an integration as well
- We’ll have a step-by-step approach to S3

<!-- Page 269 -->

# Amazon S3 Use cases

- Backup and storage
- Disaster Recovery
- Archive
- Hybrid Cloud storage
- Application hosting
- Media hosting
- Data lakes & big data analytics
- Software delivery
- Static website
Nasdaq stores 7 years of
data into S3 Glacier
Sysco runs analytics on
its data and gain business
insights

<!-- Page 270 -->

# Amazon S3 - Buckets

- Amazon S3 allows people to store objects (files) in “buckets” (directories)
- Buckets are defined at the region level
- S3 looks like a global service but buckets are created in a region
- Naming:
- Shared Global Namespace – have a globally unique name (across all regions all accounts)
- Account Regional Namespace – allows for “reuse” of the same bucket name across regions
- Naming constraints:
- No uppercase, No underscore
- Not an IP
- Must start with lowercase letter or number
- Must NOT start with the prefix xn-• Must NOT end with the suffix -s3alias
S3 Bucket

<!-- Page 271 -->

# Amazon S3 - Objects

- Objects (files) have a Key
- The key is the FULL path:
- s3://my-bucket/my_file.txt
- s3://my-bucket/my_folder1/another_folder/my_file.txt
- The key is composed of prefix + object name
Object
- s3://my-bucket/my_folder1/another_folder/my_file.txt
- There’s no concept of “directories” within buckets
(although the UI will trick you to think otherwise)
- Just keys with very long names that contain slashes (“/”)
S3 Bucket
with Objects

<!-- Page 272 -->

# Amazon S3 – Objects (cont.)

- Object values are the content of the body:
- Max. Object Size is 50TB (50,000GB)
- If uploading more than 5GB, must use “multi-part upload”
- Metadata (list of text key / value pairs – system or user metadata)
- Tags (Unicode key / value pair – up to 10) – useful for security / lifecycle
- Version ID (if versioning is enabled)

<!-- Page 273 -->

# Amazon S3 – Security

- User-Based
- IAM Policies – which API calls should be allowed for a specific user from IAM
- Resource-Based
- Bucket Policies – bucket wide rules from the S3 console - allows cross account
- Object Access Control List (ACL) – finer grain (can be disabled)
- Bucket Access Control List (ACL) – less common (can be disabled)
- Note: an IAM principal can access an S3 object if
- The user IAM permissions ALLOW it OR the resource policy ALLOWS it
- AND there’s no explicit DENY
- Encryption: encrypt objects in Amazon S3 using encryption keys

<!-- Page 274 -->

# S3 Bucket Policies

- JSON based policies
- Resources: buckets and objects
- Effect: Allow / Deny
- Actions: Set of API to Allow or Deny
- Principal: The account or user to apply the
policy to
- Use S3 bucket for policy to:
- Grant public access to the bucket
- Force objects to be encrypted at upload
- Grant access to another account (Cross
Account)

<!-- Page 275 -->

# Example: Public Access - Use Bucket Policy

S3 Bucket Policy
Allows Public Access
Anonymous www website visitor
S3 Bucket

<!-- Page 276 -->

# Example: User Access to S3 – IAM permissions

IAM Policy
IAM User
S3 Bucket

<!-- Page 277 -->

# Example: EC2 instance access - Use IAM Roles

EC2 Instance Role
IAM permissions
EC2 Instance
S3 Bucket

<!-- Page 278 -->

# Use Bucket Policy

S3 Bucket Policy
Allows Cross-Account
IAM User
Other AWS account
S3 Bucket
Advanced: Cross-Account Access –

<!-- Page 279 -->

# Bucket settings for Block Public Access

- These settings were created to prevent company data leaks
- If you know your bucket should never be public, leave these on
- Can be set at the account level

<!-- Page 280 -->

# (demo-bucket)

User
- S3 can host static websites and have them accessible on
the Internet
Amazon S3 – Static Website Hosting
http://demo-bucket.s3-website-us-west-2.amazonaws.com
http://demo-bucket.s3-website.us-west-2.amazonaws.com
- The website URL will be (depending on the region)
- http://bucket-name.s3-website-aws-region.amazonaws.com
OR
- http://bucket-name.s3-website.aws-region.amazonaws.com
- If you get a 403 Forbidden error, make sure the bucket
policy allows public reads!
us-west-2
S3 Bucket

<!-- Page 281 -->

# Amazon S3 - Versioning

- You can version your files in Amazon S3
- It is enabled at the bucket level
- Same key overwrite will change the “version”: 1, 2, 3….
- It is best practice to version your buckets
- Protect against unintended deletes (ability to restore a version)
- Easy roll back to previous version
- Notes:
- Any file that is not versioned prior to enabling versioning will
have version “null”
- Suspending versioning does not delete the previous versions
User
upload
S3 Bucket
(my-bucket)
Version 1
Version 2
Version 3
s3://my-bucket/my-file.docx

<!-- Page 282 -->

# Amazon S3 – Replication (CRR & SRR)

- Must enable Versioning in source and destination buckets
- Cross-Region Replication (CRR)
- Same-Region Replication (SRR)
- Buckets can be in different AWS accounts
- Copying is asynchronous
- Must give proper IAM permissions to S3
S3 Bucket
(eu-west-1)
asynchronous
replication
- Use cases:
- CRR – compliance, lower latency access, replication across accounts
- SRR – log aggregation, live replication between production and test
accounts
S3 Bucket
(us-east-2)

<!-- Page 283 -->

# Amazon S3 – Replication (Notes)

- After you enable Replication, only new objects are replicated
- Optionally, you can replicate existing objects using S3 Batch Replication
- Replicates existing objects and objects that failed replication
- For DELETE operations
- Can replicate delete markers from source to target (optional setting)
- Deletions with a version ID are not replicated (to avoid malicious deletes)
- There is no “chaining” of replication
- If bucket 1 has replication into bucket 2, which has replication into bucket 3
- Then objects created in bucket 1 are not replicated to bucket 3

<!-- Page 284 -->

# S3 Storage Classes

- Amazon S3 Standard - General Purpose
- Amazon S3 Standard-Infrequent Access (IA)
- Amazon S3 One Zone-Infrequent Access
- Amazon S3 Glacier Instant Retrieval
- Amazon S3 Glacier Flexible Retrieval
- Amazon S3 Glacier Deep Archive
- Amazon S3 Intelligent Tiering
- Can move between classes manually or using S3 Lifecycle configurations

<!-- Page 285 -->

# S3 Durability and Availability

- Durability:
- High durability (99.999999999%, 11 9’s) of objects across multiple AZ
- If you store 10,000,000 objects with Amazon S3, you can on average expect to
incur a loss of a single object once every 10,000 years
- Same for all storage classes
- Availability:
- Measures how readily available a service is
- Varies depending on storage class
- Example: S3 standard has 99.99% availability = not available 53 minutes a year

<!-- Page 286 -->

# S3 Standard – General Purpose

- 99.99% Availability
- Used for frequently accessed data
- Low latency and high throughput
- Sustain 2 concurrent facility failures
- Use Cases: Big Data analytics, mobile & gaming applications, content
distribution…

<!-- Page 287 -->

# S3 Storage Classes – Infrequent Access

- For data that is less frequently accessed, but requires rapid access when needed
- Lower cost than S3 Standard
- Amazon S3 Standard-Infrequent Access (S3 Standard-IA)
- 99.9% Availability
- Use cases: Disaster Recovery, backups
- Amazon S3 One Zone-Infrequent Access (S3 One Zone-IA)
- High durability (99.999999999%) in a single AZ; data lost when AZ is destroyed
- 99.5% Availability
- Use Cases: Storing secondary backup copies of on-premises data, or data you can recreate

<!-- Page 288 -->

# Amazon S3 Glacier Storage Classes

- Low-cost object storage meant for archiving / backup
- Pricing: price for storage + object retrieval cost
- Amazon S3 Glacier Instant Retrieval
- Millisecond retrieval, great for data accessed once a quarter
- Minimum storage duration of 90 days
- Amazon S3 Glacier Flexible Retrieval (formerly Amazon S3 Glacier):
- Expedited (1 to 5 minutes), Standard (3 to 5 hours), Bulk (5 to 12 hours) – free
- Minimum storage duration of 90 days
- Amazon S3 Glacier Deep Archive – for long term storage:
- Standard (12 hours), Bulk (48 hours)
- Minimum storage duration of 180 days

<!-- Page 289 -->

# S3 Intelligent-Tiering

- Small monthly monitoring and auto-tiering fee
- Moves objects automatically between Access Tiers based on usage
- There are no retrieval charges in S3 Intelligent-Tiering
- Frequent Access tier (automatic): default tier
- Infrequent Access tier (automatic): objects not accessed for 30 days
- Archive Instant Access tier (automatic): objects not accessed for 90 days
- Archive Access tier (optional): configurable from 90 days to 700+ days
- Deep Archive Access tier (optional): config. from 180 days to 700+ days

<!-- Page 290 -->

# S3 Storage Classes Comparison

Standard
IntelligentTiering
Standard-IA
Durability
One Zone-IA
Glacier Instant
Retrieval
Glacier Flexible
Retrieval
Glacier Deep
Archive
99.999999999% == (11 9’s)
Availability
99.99%
99.9%
99.9%
99.5%
99.9%
99.99%
99.99%
Availability SLA
99.9%
99%
99%
99%
99%
99.9%
99.9%
Availability
Zones
>= 3
>= 3
>= 3
1
>= 3
>= 3
>= 3
Min. Storage
Duration Charge
None
None
30 Days
30 Days
90 Days
90 Days
180 Days
Min. Billable
Object Size
None
None
128 KB
128 KB
128 KB
40 KB
40 KB
Retrieval Fee
None
None
Per GB retrieved
Per GB retrieved
Per GB retrieved
Per GB retrieved
Per GB retrieved
https://aws.amazon.com/s3/storage-classes/

<!-- Page 291 -->

# Example: us-east-1

Storage Cost
(per GB per month)
Retrieval Cost
(per 1000 request)
Standard
Intelligent-Tiering
Standard-IA
One Zone-IA
Glacier Instant
Retrieval
Glacier Flexible
Retrieval
Glacier Deep
Archive
$0.023
$0.0025 - $0.023
$0.0125
$0.01
$0.004
$0.0036
$0.00099
GET: $0.0004
POST: $0.005
GET: $0.0004
POST: $0.005
Retrieval Time
Monitoring Cost
(per 1000 objects)
GET: $0.001
POST: $0.01
Instantaneous
GET: $0.001
POST: $0.01
GET: $0.01
POST: $0.02
GET: $0.0004
POST: $0.03
GET: $0.0004
POST: $0.05
Expedited: $10
Standard: $0.05
Bulk: free
Standard: $0.10
Bulk: $0.025
Expedited (1 – 5 mins)
Standard (3 – 5 hours)
Bulk (5 – 12 hours)
Standard (12 hours)
Bulk (48 hours)
$0.0025
https://aws.amazon.com/s3/pricing/
S3 Storage Classes – Price Comparison

<!-- Page 292 -->

# S3 Express One Zone

- High performance, single Availability Zone storage class
- Objects stored in a Directory Bucket (bucket in a single AZ)
- Handle 100,000s requests per second with single-digit
millisecond latency
- Up to 10x better performance than S3 Standard (50%
lower costs)
- High Durability (99.999999999%) and Availability (99.95%)
- Co-locate your storage and compute resources in the same
AZ (reduces latency)
- Use cases: latency-sensitive apps, data-intensive apps, AI &
ML training, financial modeling, media processing, HPC…
- Best integrated with SageMaker Model Training, Athena,
EMR, Glue…
Region (us-east-1)
Availability Zone (AZ 4)
stephane--use1-az4--x-s3

<!-- Page 293 -->

# Amazon S3 – Advanced

<!-- Page 294 -->

# Amazon S3 – Moving between Storage Classes

- You can transition objects between
storage classes
- For infrequently accessed object,
move them to Standard IA
- For archive objects that you don’t
need fast access to, move them to
Glacier or Glacier Deep Archive
- Moving objects can be automated
using a Lifecycle Rules
Standard
Standard IA
Intelligent Tiering
One-Zone IA
Glacier Instant Retrieval
Glacier Flexible Retrieval
Glacier Deep Archive

<!-- Page 295 -->

# Amazon S3 – Lifecycle Rules

- Transition Actions – configure objects to transition to another storage class
- Move objects to Standard IA class 60 days after creation
- Move to Glacier for archiving after 6 months
- Expiration actions – configure objects to expire (delete) after some time
- Access log files can be set to delete after a 365 days
- Can be used to delete old versions of files (if versioning is enabled)
- Can be used to delete incomplete Multi-Part uploads
- Rules can be created for a certain prefix (example: s3://mybucket/mp3/*)
- Rules can be created for certain objects Tags (example: Department: Finance)

<!-- Page 296 -->

# Amazon S3 – Lifecycle Rules (Scenario 1)

- Your application on EC2 creates images thumbnails after profile photos
are uploaded to Amazon S3. These thumbnails can be easily recreated,
and only need to be kept for 60 days. The source images should be able
to be immediately retrieved for these 60 days, and afterwards, the user
can wait up to 6 hours. How would you design this?
- S3 source images can be on Standard, with a lifecycle configuration to
transition them to Glacier after 60 days
- S3 thumbnails can be on One-Zone IA, with a lifecycle configuration to
expire them (delete them) after 60 days

<!-- Page 297 -->

# Amazon S3 – Lifecycle Rules (Scenario 2)

- A rule in your company states that you should be able to recover your
deleted S3 objects immediately for 30 days, although this may happen
rarely. After this time, and for up to 365 days, deleted objects should be
recoverable within 48 hours.
- Enable S3 Versioning in order to have object versions, so that “deleted
objects” are in fact hidden by a “delete marker” and can be recovered
- Transition the “noncurrent versions” of the object to Standard IA
- Transition afterwards the “noncurrent versions” to Glacier Deep Archive

<!-- Page 298 -->

# Amazon S3 Analytics – Storage Class Analysis

- Help you decide when to transition objects to
the right storage class
- Recommendations for Standard and Standard IA
S3 Bucket
- Does NOT work for One-Zone IA or Glacier
- Report is updated daily
- 24 to 48 hours to start seeing data analysis
S3 Analytics
.csv report
- Good first step to put together Lifecycle Rules
(or improve them)!
Date
StorageClass
ObjectAge
8/22/2022
STANDARD
000-014
8/25/2022
STANDARD
030-044
9/6/2022
STANDARD
120-149

<!-- Page 299 -->

# S3 – Requester Pays

Standard Bucket
- In general, bucket owners pay for all
Amazon S3 storage and data transfer
costs associated with their bucket
- With Requester Pays buckets, the
requester instead of the bucket owner
pays the cost of the request and the
data download from the bucket
- Helpful when you want to share large
datasets with other accounts
- The requester must be authenticated
in AWS (cannot be anonymous)
Owner
$$ Storage Cost
Owner
$$ Networking Cost
Requester
download
Requester Pays Bucket
Owner
$$ Storage Cost
Requester
$$ Networking Cost
download

<!-- Page 300 -->

# S3 Event Notifications

- S3:ObjectCreated, S3:ObjectRemoved,
S3:ObjectRestore, S3:Replication…
- Object name filtering possible (*.jpg)
- Use case: generate thumbnails of images
uploaded to S3
- Can create as many “S3 events” as desired
SNS
events
Amazon S3
- S3 event notifications typically deliver events
in seconds but can sometimes take a minute
or longer
SQS
Lambda Function

<!-- Page 301 -->

# S3 Event Notifications – IAM Permissions

SNS
SNS Resource (Access) Policy
events
Amazon S3
SQS
SQS Resource (Access) Policy
Lambda Function
Lambda Resource Policy

<!-- Page 302 -->

# with Amazon EventBridge

events
All events
Amazon S3
bucket
rules
Amazon
EventBridge
Over 18
AWS services
as destinations
- Advanced filtering options with JSON rules (metadata, object size, name...)
- Multiple Destinations – ex Step Functions, Kinesis Streams / Firehose…
- EventBridge Capabilities – Archive, Replay Events, Reliable delivery
S3 Event Notifications

<!-- Page 303 -->

# S3 – Baseline Performance

- Amazon S3 automatically scales to high request rates, latency 100-200 ms
- Your application can achieve at least 3,500 PUT/COPY/POST/DELETE or
5,500 GET/HEAD requests per second per prefix in a bucket.
- There are no limits to the number of prefixes in a bucket.
- Example (object path => prefix):
- bucket/folder1/sub1/file => /folder1/sub1/
- bucket/folder1/sub2/file => /folder1/sub2/
- bucket/1/file
=> /1/
- bucket/2/file
=> /2/
- If you spread reads across all four prefixes evenly, you can achieve 22,000
requests per second for GET and HEAD

<!-- Page 304 -->

# S3 Performance

- Multi-Part upload:
- S3 Transfer Acceleration
- recommended for files > 100MB,
must use for files > 5GB
- Can help parallelize uploads (speed
up transfers)
Divide
In parts
- Increase transfer speed by transferring
file to an AWS edge location which will
forward the data to the S3 bucket in the
target region
- Compatible with multi-part upload
Parallel uploads
Fast
(public www)
BIG file
Amazon S3
File in USA
Fast
(private AWS)
Edge Location
USA
S3 Bucket
Australia

<!-- Page 305 -->

# S3 Performance – S3 Byte-Range Fetches

- Parallelize GETs by requesting specific
byte ranges
- Better resilience in case of failures
Can be used to speed up downloads
Can be used to retrieve only partial
data (for example the head of a file)
File in S3
File in S3
Byte-range request for header
(first XX bytes)
Part 1
Part 2
…
Part N
Requests in parallel
header

<!-- Page 306 -->

# S3 Batch Operations

S3 Inventory
- Perform bulk operations on existing S3 objects with a
single request, example:
- Modify object metadata & properties
- Copy objects between S3 buckets
- Encrypt un-encrypted objects
- Modify ACLs, tags
- Restore objects from S3 Glacier
- Invoke Lambda function to perform custom action on
each object
- A job consists of a list of objects, the action to
perform, and optional parameters
- S3 Batch Operations manages retries, tracks progress,
sends completion notifications, generate reports …
- You can use S3 Inventory to get object list and use
Athena to query and filter your objects
Objects List Report
Athena
operation
+
parameters
User
filter
filtered list
S3 Batch
Operations
Processed Objects
…

<!-- Page 307 -->

# S3 – Storage Lens

- Understand, analyze, and optimize storage across entire AWS Organization
- Discover anomalies, identify cost efficiencies, and apply data protection best
practices across entire AWS Organization (30 days usage & activity metrics)
- Aggregate data for Organization, specific accounts, regions, buckets, or prefixes
- Default dashboard or create your own dashboards
- Can be configured to export metrics daily to an S3 bucket (CSV, Parquet)
Organization
Summary Insights
Accounts
Data Protection
S3 Storage Lens
Regions
Cost Efficiency
Buckets
Configure
Aggregate
Analyze
(Dashboard)
Optimize

<!-- Page 308 -->

# Storage Lens – Default Dashboard

- Visualize summarized insights and trends for both free and advanced metrics
- Default dashboard shows Multi-Region and Multi-Account data
- Preconfigured by Amazon S3
- Can’t be deleted, but can be disabled
https://aws.amazon.com/blogs/aws/s3-storage-lens/
https://aws.amazon.com/blogs/aws/s3-storage-lens/

<!-- Page 309 -->

# Storage Lens – Metrics

- Summary Metrics
- General insights about your S3 storage
- StorageBytes, ObjectCount…
- Use cases: identify the fastest-growing (or not used) buckets and prefixes
- Cost-Optimization Metrics
- Provide insights to manage and optimize your storage costs
- NonCurrentVersionStorageBytes, IncompleteMultipartUploadStorageBytes…
- Use cases: identify buckets with incomplete multipart uploaded older than 7
days, Identify which objects could be transitioned to lower-cost storage class

<!-- Page 310 -->

# Storage Lens – Metrics

- Data-Protection Metrics
- Provide insights for data protection features
- VersioningEnabledBucketCount, MFADeleteEnabledBucketCount, SSEKMSEnabledBucketCount,
CrossRegionReplicationRuleCount…
- Use cases: identify buckets that aren’t following data-protection best practices
- Access-management Metrics
- Provide insights for S3 Object Ownership
- ObjectOwnershipBucketOwnerEnforcedBucketCount…
- Use cases: identify which Object Ownership settings your buckets use
- Event Metrics
- Provide insights for S3 Event Notifications
- EventNotificationEnabledBucketCount (identify which buckets have S3 Event Notifications
configured)

<!-- Page 311 -->

# Storage Lens – Metrics

- Performance Metrics
- Provide insights for S3 Transfer Acceleration
- TransferAccelerationEnabledBucketCount (identify which buckets have S3 Transfer
Acceleration enabled)
- Activity Metrics
- Provide insights about how your storage is requested
- AllRequests, GetRequests, PutRequests, ListRequests, BytesDownloaded…
- Detailed Status Code Metrics
- Provide insights for HTTP status codes
- 200OKStatusCount, 403ForbiddenErrorCount, 404NotFoundErrorCount…

<!-- Page 312 -->

# Storage Lens – Free vs. Paid

- Free Metrics
- Automatically available for all customers
- Contains around 28 usage metrics
- Data is available for queries for 14 days
- Advanced Metrics and Recommendations
- Additional paid metrics and features
- Advanced Metrics – Activity, Advanced Cost
Optimization, Advanced Data Protection, Status
Code
- CloudWatch Publishing – Access metrics in
CloudWatch without additional charges
- Prefix Aggregation – Collect metrics at the prefix
level
- Data is available for queries for 15 months

<!-- Page 313 -->

# Amazon S3 – Security

<!-- Page 314 -->

# Amazon S3 – Object Encryption

- You can encrypt objects in S3 buckets using one of 4 methods
- Server-Side Encryption (SSE)
- Server-Side Encryption with Amazon S3-Managed Keys (SSE-S3) – Enabled by Default
- Encrypts S3 objects using keys handled, managed, and owned by AWS
- Server-Side Encryption with KMS Keys stored in AWS KMS (SSE-KMS)
- Leverage AWS Key Management Service (AWS KMS) to manage encryption keys
- Server-Side Encryption with Customer-Provided Keys (SSE-C)
- When you want to manage your own encryption keys
- Client-Side Encryption
- It’s important to understand which ones are for which situation for the exam

<!-- Page 315 -->

# Amazon S3 Encryption – SSE-S3

- Encryption using keys handled, managed, and owned by AWS
- Object is encrypted server-side
- Encryption type is AES-256
- Must set header "x-amz-server-side-encryption": "AES256"
- Enabled by default for new buckets & new objects
Object
upload
User
HTTP(S) + Header
Amazon S3
+
S3 Bucket
S3 Owned Key
Encryption

<!-- Page 316 -->

# Amazon S3 Encryption – SSE-KMS

- Encryption using keys handled and managed by AWS KMS (Key Management Service)
- KMS advantages: user control + audit key usage using CloudTrail
- Object is encrypted server side
- Must set header "x-amz-server-side-encryption": "aws:kms"
Amazon S3
Object
upload
User
HTTP(S) + Header
+
Encryption
S3 Bucket
KMS Key
AWS KMS

<!-- Page 317 -->

# SSE-KMS Limitation

- If you use SSE-KMS, you may be impacted
by the KMS limits
- When you upload, it calls the
GenerateDataKey KMS API
- When you download, it calls the Decrypt
KMS API
- Count towards the KMS quota per second
(5500, 10000, 30000 req/s based on region)
- You can request a quota increase using the
Service Quotas Console
S3 Bucket
KMS Key
API call
Upload / download
SSE-KMS
Users

<!-- Page 318 -->

# Amazon S3 Encryption – SSE-C

- Server-Side Encryption using keys fully managed by the customer outside of AWS
- Amazon S3 does NOT store the encryption key you provide
- HTTPS must be used
- Encryption key must provided in HTTP headers, for every HTTP request made
+
upload
User
HTTPS ONLY
+ Key in Header
Object
+
Amazon S3
Encryption
S3 Bucket
Client-Provided Key

<!-- Page 319 -->

# Amazon S3 Encryption – Client-Side Encryption

- Use client libraries such as Amazon S3 Client-Side Encryption Library
- Clients must encrypt data themselves before sending to Amazon S3
- Clients must decrypt data themselves when retrieving from Amazon S3
- Customer fully manages the keys and encryption cycle
Amazon S3
File
+
upload
Encryption
File
Client Key
(encrypted)
HTTP(S)
S3 Bucket

<!-- Page 320 -->

# Amazon S3 – Encryption in transit (SSL/TLS)

- Encryption in flight is also called SSL/TLS
- Amazon S3 exposes two endpoints:
- HTTP Endpoint – non encrypted
- HTTPS Endpoint – encryption in flight
- HTTPS is recommended
- HTTPS is mandatory for SSE-C
- Most clients would use the HTTPS endpoint by default

<!-- Page 321 -->

# aws:SecureTransport

User
Account B
http
S3 Bucket
https
User
(my-bucket)
Bucket Policy
Amazon S3 – Force Encryption in Transit

<!-- Page 322 -->

# Amazon S3 – Default Encryption vs. Bucket Policies

- SSE-S3 encryption is automatically applied to new objects stored in S3 bucket
- Optionally, you can “force encryption” using a bucket policy and refuse any API call
to PUT an S3 object without encryption headers (SSE-KMS or SSE-C)
- Note: Bucket Policies are evaluated before “Default Encryption”

<!-- Page 323 -->

# What is CORS?

- Cross-Origin Resource Sharing (CORS)
- Origin = scheme (protocol) + host (domain) + port
- example: https://www.example.com (implied port is 443 for HTTPS, 80 for HTTP)
- Web Browser based mechanism to allow requests to other origins while
visiting the main origin
- Same origin: http://example.com/app1 & http://example.com/app2
- Different origins: http://www.example.com & http://other.example.com
- The requests won’t be fulfilled unless the other origin allows for the
requests, using CORS Headers (example: Access-Control-Allow-Origin)

<!-- Page 324 -->

# What is CORS?

OPTIONS /
Host: www.other.com
Origin: https://www.example.com
Preflight Request
HTTPS Request
Access-Control-Allow-Origin: https://www.example.com
Access-Control-Allow-Methods: GET, PUT, DELETE
Preflight Response
Web Server
(Origin)
https://www.example.com
Web Browser
Web Server
GET /
Host: www.other.com
Origin: https://www.example.com
CORS Headers received already by the Origin
The Web Browser can make requests
(Cross-Origin)
https://www.other.com

<!-- Page 325 -->

# Amazon S3 – CORS

- If a client makes a cross-origin request on our S3 bucket, we need to enable
the correct CORS headers
- It’s a popular exam question
- You can allow for a specific origin or for * (all origins)
GET /index.html
Host: http://my-bucket-html.s3-website.us-west-2.amazonaws.com
index.html
Web Browser
GET /images/coffee.jpg
Host: http://my-bucket-assets.s3-website.us-west-2.amazonaws.com
Origin: http://my-bucket-html.s3-website.us-west-2.amazonaws.com
Access-Control-Allow-Origin: http://my-bucket-html.s3-website.us-west-2.amazonaws.com
S3 Bucket
(my-bucket-html)
(Static Website Enabled)
S3 Bucket
(my-bucket-assets)
(Static Website Enabled)

<!-- Page 326 -->

# Amazon S3 – MFA Delete

- MFA (Multi-Factor Authentication) – force users to generate a code on a
device (usually a mobile phone or hardware) before doing important
operations on S3
- MFA will be required to:
- Permanently delete an object version
- Suspend Versioning on the bucket
Google Authenticator
- MFA won’t be required to:
- Enable Versioning
- List deleted versions
MFA Hardware Device
- To use MFA Delete, Versioning must be enabled on the bucket
- Only the bucket owner (root account) can enable/disable MFA Delete

<!-- Page 327 -->

# S3 Access Logs

- For audit purpose, you may want to log all access to S3 buckets
- Any request made to S3, from any account, authorized or denied,
will be logged into another S3 bucket
- That data can be analyzed using data analysis tools…
- The target logging bucket must be in the same AWS region
- The log format is at:
https://docs.aws.amazon.com/AmazonS3/latest/dev/LogFormat.html
requests
My-bucket
Log all
requests
Logging Bucket

<!-- Page 328 -->

# S3 Access Logs: Warning

- Do not set your logging bucket to be the monitored bucket
- It will create a logging loop, and your bucket will grow exponentially
Logging loop
PutObject
App Bucket &
Logging Bucket
Do not try this at home J

<!-- Page 329 -->

# Amazon S3 – Pre-Signed URLs

Owner
- S3 Console – 1 min up to 720 mins (12 hours)
- AWS CLI – configure expiration with --expires-in parameter in seconds
(default 3600 secs, max. 604800 secs ~ 168 hours)
- Users given a pre-signed URL inherit the permissions of the user
that generated the URL for GET / PUT
- Examples:
- Allow only logged-in users to download a premium video from your S3
bucket
- Allow an ever-changing list of users to download files by generating URLs
dynamically
- Allow temporarily a user to upload a file to a precise location in your S3
bucket
URL
generate
pre-signed URL
- Generate pre-signed URLs using the S3 Console, AWS CLI or SDK
- URL Expiration
URL
S3 Bucket
(Private)
URL
User

<!-- Page 330 -->

# S3 Glacier Vault Lock

- Adopt a WORM (Write Once Read
Many) model
- Create a Vault Lock Policy
- Lock the policy for future edits
(can no longer be changed or deleted)
- Helpful for compliance and data
retention
Object
Vault Lock Policy
Object can’t be deleted

<!-- Page 331 -->

# S3 Object Lock (versioning must be enabled)

- Adopt a WORM (Write Once Read Many) model
- Block an object version deletion for a specified amount of time
- Retention mode - Compliance:
- Object versions can't be overwritten or deleted by any user, including the root user
- Objects retention modes can't be changed, and retention periods can't be shortened
- Retention mode - Governance:
- Most users can't overwrite or delete an object version or alter its lock settings
- Some users have special permissions to change the retention or delete the object
- Retention Period: protect the object for a fixed period, it can be extended
- Legal Hold:
- protect the object indefinitely, independent from retention period
- can be freely placed and removed using the s3:PutObjectLegalHold IAM permission

<!-- Page 332 -->

# S3 – Access Points

Policy
Users
(Finance)
Users
(Sales)
Users
(Analytics)
Grant R/W to
/finance prefix
Policy
Grant R/W to
/sales prefix
Policy
Grant R to
entire bucket
Finance
Access Point
Sales
Access Point
S3 Bucket
/finance/…
Simple Bucket
Policy
/sales/…
Analytics
Access Point
- Access Points simplify security management for S3 Buckets
- Each Access Point has:
- its own DNS name (Internet Origin or VPC Origin)
- an access point policy (similar to bucket policy) – manage security at scale

<!-- Page 333 -->

# S3 – Access Points – VPC Origin

- We can define the access
point to be accessible
only from within the VPC
- You must create a VPC
Endpoint to access the
Access Point (Gateway
or Interface Endpoint)
- The VPC Endpoint Policy
must allow access to the
target bucket and Access
Point
VPC
EC2 Instance
VPC Endpoint
Endpoint
Policy
Access Point
VPC Origin
Access Point
Policy
S3 Bucket
Bucket
Policy

<!-- Page 334 -->

# S3 Object Lambda

- Use AWS Lambda Functions to
change the object before it is
retrieved by the caller application
- Only one S3 bucket is needed, on
top of which we create S3 Access
Point and S3 Object Lambda Access
Points.
- Use Cases:
- Redacting personally identifiable
information for analytics or nonproduction environments.
- Converting across data formats, such
as converting XML to JSON.
- Resizing and watermarking images on
the fly using caller-specific details, such
as the user who requested the object.
AWS Cloud
Original
Object
S3 Bucket
E-Commerce
Application
Access Point
Redacting
Lambda Function
Access Point
Enriching
Lambda Function
Supporting
S3 Access Point
Redacted
Object
Analytics
Application
Enriched
Object
Marketing
Application
Customer Loyalty
Database

<!-- Page 335 -->

# CloudFront & Global Accelerator

<!-- Page 336 -->

# Amazon CloudFront

- Content Delivery Network (CDN)
- Improves read performance, content
is cached at the edge
- Improves users experience
- Hundreds of Points of Presence
globally (edge locations, caches)
- DDoS protection (because
worldwide), integration with Shield,
AWS Web Application Firewall
Source: https://aws.amazon.com/cloudfront/features/?nc=sn&loc=2

<!-- Page 337 -->

# CloudFront – Origins

- S3 bucket
- For distributing files and caching them at the edge
- For uploading files to S3 through CloudFront
- Secured using Origin Access Control (OAC)
- VPC Origin
- For applications hosted in VPC private subnets
- Private Application Load Balancer / Network Load Balancer / EC2 Instances
- Custom Origin (HTTP)
- S3 website (must first enable the bucket as a static S3 website)
- Any public HTTP backend you want (example: Public ALB)

<!-- Page 338 -->

# CloudFront at a high level

GET /beach.jpg?size=300x300 HTTP/1.1
User-Agent: Mozilla/4.0 (compatible; MSIE5.01; Windows NT)
Host: www.example.com
Accept-Encoding: gzip, deflate
Forward Request
to your Origin
Client
CloudFront Edge Location
Origin
S3
or
HTTP
Local Cache

<!-- Page 339 -->

# CloudFront – S3 as an Origin

AWS Cloud
Public www
Private AWS
Edge
Los Angeles
Private AWS
Edge
Mumbai
Private AWS
Private AWS
Origin (S3 bucket)
OAC
Public www
Edge
Melbourne
Edge
São Paulo
Origin Access Control
+ S3 bucket policy

<!-- Page 340 -->

# CloudFront vs S3 Cross Region Replication

- CloudFront:
- Global Edge network
- Files are cached for a TTL (maybe a day)
- Great for static content that must be available everywhere
- S3 Cross Region Replication:
- Must be setup for each region you want replication to happen
- Files are updated in near real-time
- Read only
- Great for dynamic content that needs to be available at low-latency in few regions

<!-- Page 341 -->

# Using VPC Origins

- Allows you to deliver content from your applications hosted in your
VPC private subnets (no need to expose them on the Internet)
- Deliver traffic to private:
- Application Load Balancer
- Network Load Balancer
- EC2 Instances
VPC
Private Subnet
Application Load Balancer
Users
CloudFront
VPC
Origin
Network Load Balancer
EC2 Instance
Edge Location
CloudFront – ALB or EC2 as an origin

<!-- Page 342 -->

# Using Public Network

Security group
Allow Public IP of Edge Locations
http://d7uri8nf7uskq.cloudfront.net/tools/list-cloudfront-ips
EC2 Instances
Must be Public
Edge Location
Allow Public IP of
Edge Locations
Edge Location
Public IPs
Security group
Application Load Balancer
Must be Public
Allow Security Group
of Load Balancer
Security group
EC2 Instances
Can be Private
CloudFront – ALB or EC2 as an origin

<!-- Page 343 -->

# CloudFront Geo Restriction

- You can restrict who can access your distribution
- Allowlist: Allow your users to access your content only if they're in one of the
countries on a list of approved countries.
- Blocklist: Prevent your users from accessing your content if they're in one of the
countries on a list of banned countries.
- The “country” is determined using a 3rd party Geo-IP database
- Use case: Copyright Laws to control access to content

<!-- Page 344 -->

# CloudFront – Cache Invalidations

- In case you update the back-end
origin, CloudFront doesn’t know
about it and will only get the
refreshed content after the TTL has
expired
- However, you can force an entire or
partial cache refresh (thus bypassing
the TTL) by performing a CloudFront
Invalidation
- You can invalidate all files (*) or a
special path (/images/*)
GET /index.html
CloudFront
invalidate
Edge Location
Edge Location
Cache
Cache
index.html
update files
Invalidate
- /index.html
- /images/*
/images/
S3 Bucket
(origin)
index.html
/images/

<!-- Page 345 -->

# Global users for our application

- You have deployed an
application and have global
users who want to access it
directly.
- They go over the public
internet, which can add a lot of
latency due to many hops
- We wish to go as fast as
possible through AWS network
to minimize latency
America
hops
Europe
Public ALB
Australia
India

<!-- Page 346 -->

# Unicast IP vs Anycast IP

Client
- Unicast IP: one server holds one IP
address
12.34.56.78
- Anycast IP: all servers hold the same
IP address and the client is routed to
the nearest one
Client
12.34.56.78
98.76.54.32
12.34.56.78

<!-- Page 347 -->

# AWS Global Accelerator

- Leverage the AWS internal
network to route to your
application
- 2 Anycast IP are created for your
application
- The Anycast IP send traffic directly
to Edge Locations
- The Edge locations send the traffic
to your application
America
Edge location
Europe
Private AWS
Australia
Public ALB
India

<!-- Page 348 -->

# AWS Global Accelerator

- Works with Elastic IP, EC2 instances, ALB, NLB, public or private
- Consistent Performance
- Intelligent routing to lowest latency and fast regional failover
- No issue with client cache (because the IP doesn’t change)
- Internal AWS network
- Health Checks
- Global Accelerator performs a health check of your applications
- Helps make your application global (failover less than 1 minute for unhealthy)
- Great for disaster recovery (thanks to the health checks)
- Security
- only 2 external IP need to be whitelisted
- DDoS protection thanks to AWS Shield

<!-- Page 349 -->

# AWS Global Accelerator vs CloudFront

- They both use the AWS global network and its edge locations around the world
- Both services integrate with AWS Shield for DDoS protection.
- CloudFront
- Improves performance for both cacheable content (such as images and videos)
- Dynamic content (such as API acceleration and dynamic site delivery)
- Content is served at the edge
- Global Accelerator
- Improves performance for a wide range of applications over TCP or UDP
- Proxying packets at the edge to applications running in one or more AWS Regions.
- Good fit for non-HTTP use cases, such as gaming (UDP), IoT (MQTT), or Voice over IP
- Good for HTTP use cases that require static IP addresses
- Good for HTTP use cases that required deterministic, fast regional failover

<!-- Page 350 -->

# AWS Storage Extras

<!-- Page 351 -->

# AWS Snowball

- Highly-secure, portable devices to collect and process data at the edge,
and migrate data into and out of AWS
- Helps migrate up to Petabytes of data
Device
Compute
Memory
Storage (SSD)
Snowball Edge Storage Optimized
104 vCPUs
416 GB
210 TB
Snowball Edge Compute Optimized
104 vCPUs
416 GB
28 TB
Snowball Edge

<!-- Page 352 -->

# Data Migrations with Snowball

Time to Transfer
100 Mbps
1Gbps
10Gbps
10 TB
12 days
30 hours
3 hours
100 TB
124 days
12 days
30 hours
1 PB
3 years
124 days
12 days
Challenges:
- Limited connectivity
- Limited bandwidth
- High network cost
- Shared bandwidth (can’t
maximize the line)
- Connection stability
AWS Snowball: offline devices to perform data migrations
If it takes more than a week to transfer over the network, use Snowball devices!

<!-- Page 353 -->

# Diagrams

- Direct upload to S3:
www: 10Gbit/s
Amazon S3
bucket
client
- With Snowball:
ship
client
AWS
Snowball
AWS
Snowball
import/
export
Amazon S3
bucket

<!-- Page 354 -->

# What is Edge Computing?

- Process data while it’s being created on an edge location
- A truck on the road, a ship on the sea, a mining station underground...
- These locations may have limited internet and no access to computing power
- We setup a Snowball Edge device to do edge computing
- Snowball Edge Compute Optimized (dedicated for that use case) & Storage Optimized
- Run EC2 Instances or Lambda functions at the edge
- Use cases: preprocess data, machine learning, transcoding media

<!-- Page 355 -->

# Solution Architecture: Snowball into Glacier

- Snowball cannot import to Glacier directly
- You must use Amazon S3 first, in combination with an S3 lifecycle policy
import
Snowball
S3 lifecycle policy
Amazon S3
Amazon Glacier

<!-- Page 356 -->

# Amazon FSx – Overview

- Launch 3rd party high-performance file systems on AWS
- Fully managed service
FSx for Lustre
FSx for
NetApp ONTAP
FSx for Windows
File Server
FSx for
OpenZFS

<!-- Page 357 -->

# Amazon FSx for Windows (File Server)

- FSx for Windows is a fully managed Windows file system share drive
- Supports SMB protocol & Windows NTFS
- Microsoft Active Directory integration, ACLs, user quotas
- Can be mounted on Linux EC2 instances
- Supports Microsoft's Distributed File System (DFS) Namespaces (group files across multiple FS)
- Scale up to 10s of GB/s, millions of IOPS, 100s PB of data
- Storage Options:
- SSD – latency sensitive workloads (databases, media processing, data analytics, …)
- HDD – broad spectrum of workloads (home directory, CMS, …)
- Can be accessed from your on-premises infrastructure (VPN or Direct Connect)
- Can be configured to be Multi-AZ (high availability)
- Data is backed-up daily to S3

<!-- Page 358 -->

# Amazon FSx for Lustre

- Lustre is a type of parallel distributed file system, for large-scale computing
- The name Lustre is derived from “Linux” and “cluster
- Machine Learning, High Performance Computing (HPC)
- Video Processing, Financial Modeling, Electronic Design Automation
- Scales up to 100s GB/s, millions of IOPS, sub-ms latencies
- Storage Options:
- SSD – low-latency, IOPS intensive workloads, small & random file operations
- HDD – throughput-intensive workloads, large & sequential file operations
- Seamless integration with S3
- Can “read S3” as a file system (through FSx)
- Can write the output of the computations back to S3 (through FSx)
- Can be used from on-premises servers (VPN or Direct Connect)

<!-- Page 359 -->

# FSx Lustre - File System Deployment Options

- Scratch File System
- Temporary storage
- Data is not replicated (doesn’t persist if file
server fails)
- High burst (6x faster, 200MBps per TiB)
- Usage: short-term processing, optimize
costs
- Persistent File System
- Long-term storage
- Data is replicated within same AZ
- Replace failed files within minutes
- Usage: long-term processing, sensitive data
Region
Availability Zone 1
Compute
instances
ENI
FSx For Lustre
(Scratch file system)
Availability Zone 2
Compute
instances
S3 bucket
(optional data repository)
Region
Availability Zone 1
Compute
instances
FSx For Lustre
(Persistent file system)
ENI
Availability Zone 2
Compute
instances
S3 bucket
(optional data repository)

<!-- Page 360 -->

# Amazon FSx for NetApp ONTAP

- Managed NetApp ONTAP on AWS
- File System compatible with NFS, SMB, iSCSI protocol
- Move workloads running on ONTAP or NAS to AWS
- Works with:
- 
- 
- 
- 
- 
- 
Linux
Windows
MacOS
VMware Cloud on AWS
Amazon Workspaces & AppStream 2.0
Amazon EC2, ECS and EKS
- Storage shrinks or grows automatically
- Snapshots, replication, low-cost, compression and data
de-duplication
- Point-in-time instantaneous cloning (helpful for testing
new workloads)
Amazon FSx for
NetApp ONTAP FS
NFS, SMB, iSCSI
EC2
ECS
VMware Cloud
on AWS
Amazon
AppStream 2.0
EKS
Amazon
On-premises
WorkSpaces
Server

<!-- Page 361 -->

# Amazon FSx for OpenZFS

- Managed OpenZFS file system on AWS
- File System compatible with NFS (v3, v4, v4.1, v4.2)
- Move workloads running on ZFS to AWS
- Works with:
Amazon FSx
for OpenZFS
- Linux
- Windows
- MacOS
- VMware Cloud on AWS
- Amazon Workspaces & AppStream 2.0
- Amazon EC2, ECS and EKS
- Up to 1,000,000 IOPS with < 0.5ms latency
- Snapshots, compression and low-cost
- Point-in-time instantaneous cloning (helpful for
testing new workloads)
NFS (v3, v4, v4.1, v4.2)
EC2
ECS
VMware Cloud
on AWS
Amazon
AppStream 2.0
EKS
Amazon
On-premises
WorkSpaces
Server

<!-- Page 362 -->

# Hybrid Cloud for Storage

- AWS is pushing for ”hybrid cloud”
- Part of your infrastructure is on the cloud
- Part of your infrastructure is on-premises
- This can be due to
- Long cloud migrations
- Security requirements
- Compliance requirements
- IT strategy
- S3 is a proprietary storage technology (unlike EFS / NFS), so how do
you expose the S3 data on-premises?
- AWS Storage Gateway!

<!-- Page 363 -->

# AWS Storage Cloud Native Options

Block
Amazon EBS
EC2 Instance
Store
File
Amazon EFS
Object
Amazon FSx
Amazon S3
Amazon Glacier

<!-- Page 364 -->

# AWS Storage Gateway

- Bridge between on-premises data and cloud data
- Use cases:
- disaster recovery
- backup & restore
- tiered storage
- on-premises cache & low-latency files access
- Types of Storage Gateway:
- S3 File Gateway
- Volume Gateway
- Tape Gateway
Storage Gateway

<!-- Page 365 -->

# Amazon S3 File Gateway

- Configured S3 buckets are accessible using the NFS and SMB protocol
- Most recently used data is cached in the file gateway
- Supports S3 Standard, S3 Standard IA, S3 One Zone A, S3 Intelligent Tiering
- Transition to S3 Glacier using a Lifecycle Policy
- Bucket access using IAM roles for each File Gateway
- SMB Protocol has integration with Active Directory (AD) for user authentication
AWS Cloud
Corporate
Data Center
HTTPS
NFS or SMB
Application
Server
S3 File
Gateway
S3 Standard
S3 Standard-IA
S3 One Zone-IA
S3 Intelligent-Tiering
Lifecycle
policy
S3 Glacier
.

<!-- Page 366 -->

# Volume Gateway

- Block storage using iSCSI protocol backed by S3
- Backed by EBS snapshots which can help restore on-premises volumes!
- Cached volumes: low latency access to most recent data
- Stored volumes: entire dataset is on premise, scheduled backups to S3
Corporate
Data Center
AWS Cloud
Region
HTTPS
iSCSI
Application
Server
S3 Bucket
Amazon EBS
Snapshots

<!-- Page 367 -->

# Tape Gateway

- Some companies have backup processes using physical tapes (!)
- With Tape Gateway, companies use the same processes but, in the cloud
- Virtual Tape Library (VTL) backed by Amazon S3 and Glacier
- Back up data using existing tape-based processes (and iSCSI interface)
- Works with leading backup software vendors
Corporate
Data Center
iSCSI
Backup
Server
AWS Cloud
Region
Media
Changer
Tape
Drive
HTTPS
Tape
Gateway
Virtual Tapes
stored in
Amazon S3
Archived Tapes
stored in
Amazon Glacier

<!-- Page 368 -->

# AWS Storage Gateway

On-Premises
AWS Cloud
NFS/SMB
File Gateway
User/group file shares
local cache
iSCSI
Any S3 Storage Class
Including Glacier
Amazon S3
AWS EBS
Encryption in Transit
Internet or Direct Connect
Volume Gateway
Application Server
Amazon S3
excluding Glacier &
Glacier Deep Archive
local cache
Storage Gateway
Eject from backup application
iSCSI VTL
Backup Application
Tape Gateway
local cache
Gateway Deployment Options
VM(VMware, Hyper-V, KVM)
Amazon S3
Tape Library
Tape Archive
Glacier &
Glacier Deep Archive

<!-- Page 369 -->

# AWS Transfer Family

- A fully-managed service for file transfers into and out of Amazon S3 or
Amazon EFS using the FTP protocol
- Supported Protocols
- AWS Transfer for FTP (File Transfer Protocol (FTP))
- AWS Transfer for FTPS (File Transfer Protocol over SSL (FTPS))
- AWS Transfer for SFTP (Secure File Transfer Protocol (SFTP))
- Managed infrastructure, Scalable, Reliable, Highly Available (multi-AZ)
- Pay per provisioned endpoint per hour + data transfers in GB
- Store and manage users’ credentials within the service
- Integrate with existing authentication systems (Microsoft Active Directory,
LDAP, Okta, Amazon Cognito, custom)
- Usage: sharing files, public datasets, CRM, ERP, …

<!-- Page 370 -->

# AWS Transfer Family

MS Active Directory
LDAP
…
authenticate
AWS Transfer for SFTP
Amazon S3
AWS Transfer for FTPS
Users
(FTP client)
Route 53
(optional)
AWS Transfer for FTP
(only within VPC)
IAM Role
Amazon EFS

<!-- Page 371 -->

# AWS DataSync

- Move large amount of data to and from
- On-premises / other cloud to AWS (NFS, SMB, HDFS, S3 API…) – needs agent
- AWS to AWS (different storage services) – no agent needed
- Can synchronize to:
- Amazon S3 (any storage classes – including Glacier)
- Amazon EFS
- Amazon FSx (Windows, Lustre, NetApp, OpenZFS...)
- Replication tasks can be scheduled hourly, daily, weekly
- File permissions and metadata are preserved (NFS POSIX, SMB…)
- One agent task can use 10 Gbps, can setup a bandwidth limit

<!-- Page 372 -->

# NFS / SMB to AWS (S3, EFS, FSx…)

Region
On-Premises
AWS Storage Resources
NFS or SMB
NFS or SMB
Server
TLS
AWS DataSync
Agent
AWS
DataSync
S3 Standard
S3 IntelligentTiering
S3 Standard-IA
S3 One
Zone-IA
S3 Glacier
S3 Glacier
Deep Archive
AWS EFS
Amazon FSx
AWS DataSync

<!-- Page 373 -->

# Transfer between AWS storage services

Amazon S3
Amazon S3
Amazon EFS
Amazon EFS
AWS DataSync
copy data and metadata
between AWS Storage Services
Amazon FSx
Amazon FSx
AWS DataSync

<!-- Page 374 -->

# Storage Comparison

- S3: Object Storage
- S3 Glacier: Object Archival
- EBS volumes: Network storage for one EC2 instance at a time
- Instance Storage: Physical storage for your EC2 instance (high IOPS)
- EFS: Network File System for Linux instances, POSIX filesystem
- FSx for Windows: Network File System for Windows servers
- FSx for Lustre: High Performance Computing Linux file system
- FSx for NetApp ONTAP: High OS Compatibility
- FSx for OpenZFS: Managed ZFS file system
- Storage Gateway: S3 & FSx File Gateway, Volume Gateway (cache & stored), Tape Gateway
- Transfer Family: FTP, FTPS, SFTP interface on top of Amazon S3 or Amazon EFS
- DataSync: Schedule data sync from on-premises to AWS, or AWS to AWS
- Snowcone / Snowball / Snowmobile: to move large amount of data to the cloud, physically
- Database: for specific workloads, usually with indexing and querying

<!-- Page 375 -->

# AWS Integration & Messaging

SQS, SNS & Kinesis

<!-- Page 376 -->

# Section Introduction

- When we start deploying multiple applications, they will inevitably need
to communicate with one another
- There are two patterns of application communication
1) Synchronous communications
(application to application)
Buying
Service
Shipping
Service
2) Asynchronous / Event based
(application to queue to application)
Buying
Service
Queue
Shipping
Service

<!-- Page 377 -->

# Section Introduction

- Synchronous between applications can be problematic if there are
sudden spikes of traffic
- What if you need to suddenly encode 1000 videos but usually it’s 10?
- In that case, it’s better to decouple your applications,
- using SQS: queue model
- using SNS: pub/sub model
- using Kinesis: real-time streaming model
- These services can scale independently from our application!

<!-- Page 378 -->

# What’s a queue?

Consumer
Producer
Consumer
Producer
Send messages
Poll messages
Consumer
Producer
SQS Queue
Consumer
Amazon SQS

<!-- Page 379 -->

# Amazon SQS – Standard Queue

- Oldest offering (over 10 years old)
- Fully managed service, used to decouple applications
- Attributes:
- Unlimited throughput, unlimited number of messages in queue
- Default retention of messages: 4 days, maximum of 14 days
- Low latency (<10 ms on publish and receive)
- Limitation of 1,024 KB per message sent
- Can have duplicate messages (at least once delivery, occasionally)
- Can have out of order messages (best effort ordering)

<!-- Page 380 -->

# SQS – Producing Messages

- Produced to SQS using the SDK (SendMessage API)
- The message is persisted in SQS until a consumer deletes it
- Message retention: default 4 days, up to 14 days
- Example: send an order to be processed
- Order id
- Customer id
- Any attributes you want
- SQS standard: unlimited throughput
Sent to SQS
Message
Up to 1024 KB

<!-- Page 381 -->

# SQS – Consuming Messages

- Consumers (running on EC2 instances, servers, or AWS Lambda)…
- Poll SQS for messages (receive up to 10 messages at a time)
- Process the messages (example: insert the message into an RDS database)
- Delete the messages using the DeleteMessage API
Poll / Receive
messages
DeleteMessage
Consumer
insert

<!-- Page 382 -->

# SQS – Multiple EC2 Instances Consumers

SQS Queue
poll
- Consumers receive and process
messages in parallel
- At least once delivery
- Best-effort message ordering
- Consumers delete messages
after processing them
- We can scale consumers
horizontally to improve
throughput of processing

<!-- Page 383 -->

# SQS with Auto Scaling Group (ASG)

Poll for messages
EC2 Instances
SQS Queue
Auto Scaling Group
scale
Alarm for breach
CloudWatch Metric – Queue Length
ApproximateNumberOfMessages
CloudWatch Alarm

<!-- Page 384 -->

# SQS to decouple between application tiers

Back-end processing
application
Front-end web app
requests
SendMessage
ReceiveMessages
SQS Queue
(infinitely scalable)
Auto-Scaling
Auto-Scaling

<!-- Page 385 -->

# Amazon SQS - Security

- Encryption:
- In-flight encryption using HTTPS API
- At-rest encryption using KMS keys
- Client-side encryption if the client wants to perform encryption/decryption itself
- Access Controls: IAM policies to regulate access to the SQS API
- SQS Access Policies (similar to S3 bucket policies)
- Useful for cross-account access to SQS queues
- Useful for allowing other services (SNS, S3…) to write to an SQS queue

<!-- Page 386 -->

# SQS – Message Visibility Timeout

- After a message is polled by a consumer, it becomes invisible to other consumers
- By default, the “message visibility timeout” is 30 seconds
- That means the message has 30 seconds to be processed
- After the message visibility timeout is over, the message is “visible” in SQS
ReceiveMessage
Request
ReceiveMessage
Request
ReceiveMessage
Request
ReceiveMessage
Request
Visibility timeout
Not returned
Message returned
Not returned
Time
Message returned (again)

<!-- Page 387 -->

# SQS – Message Visibility Timeout

ReceiveMessage
Request
ReceiveMessage
Request
ReceiveMessage
Request
ReceiveMessage
Request
Visibility timeout
Not returned
Message returned
Not returned
Time
Message returned (again)
- If a message is not processed within the visibility timeout, it will be processed twice
- A consumer could call the ChangeMessageVisibility API to get more time
- If visibility timeout is high (hours), and consumer crashes, re-processing will take time
- If visibility timeout is too low (seconds), we may get duplicates

<!-- Page 388 -->

# Amazon SQS - Long Polling

- When a consumer requests messages from the
queue, it can optionally “wait” for messages to
arrive if there are none in the queue
- This is called Long Polling
- LongPolling decreases the number of API calls
made to SQS while increasing the efficiency and
reducing latency of your application
- The wait time can be between 1 sec to 20 sec
(20 sec preferable)
- Long Polling is preferable to Short Polling
- Long polling can be enabled at the queue level
or at the API level using WaitTimeSeconds
message
SQS Queue
poll
Consumer

<!-- Page 389 -->

# – mandatory parameter

- FIFO = First In First Out (ordering of messages in the queue)
Producer
Poll messages
Send messages
4
3
2
1
4
3
2
1
Consumer
Amazon SQS – FIFO Queue
- Limited throughput: 300 msg/s without batching, 3000 msg/s with
- Exactly-once send capability (by removing duplicates using Deduplication ID)
- Messages are processed in order by the consumer
- Ordering by Message Group ID (all messages in the same group are ordered)

<!-- Page 390 -->

# SQS with Auto Scaling Group (ASG)

Poll for messages
EC2 Instances
SQS Queue
Auto Scaling Group
scale
Alarm for breach
CloudWatch Metric – Queue Length
ApproximateNumberOfMessages
CloudWatch Alarm

<!-- Page 391 -->

# some transactions may be lost

Amazon RDS
Application
Insert
transactions
requests
Amazon Aurora
Auto-Scaling
Amazon DynamoDB
If the load is too big,

<!-- Page 392 -->

# SQS as a buffer to database writes

Dequeue message
Enqueue message
requests
SendMessage
ReceiveMessages
insert
SQS Queue
(infinitely scalable)
Auto-Scaling
Auto-Scaling

<!-- Page 393 -->

# SQS to decouple between application tiers

Back-end processing
application
Front-end web app
requests
SendMessage
ReceiveMessages
SQS Queue
(infinitely scalable)
Auto-Scaling
Auto-Scaling

<!-- Page 394 -->

# Amazon SNS

- What if you want to send one message to many receivers?
Direct
integration
Buying
Service
Email
notification
Fraud
Service
Shipping
Service
SQS Queue
Pub / Sub
Email
notification
Fraud
Service
Buying
Service
SNS Topic
Shipping
Service
SQS Queue

<!-- Page 395 -->

# Amazon SNS

- The “event producer” only sends message to one SNS topic
- As many “event receivers” (subscriptions) as we want to listen to the SNS topic notifications
- Each subscriber to the topic will get all the messages (note: new feature to filter messages)
- Up to 12,500,000 subscriptions per topic
- 100,000 topics limit
Subscribers
publish
SQS
Lambda
Kinesis Data
Firehose
Emails
SMS &
Mobile Notifications
HTTP(S)
Endpoints
SNS

<!-- Page 396 -->

# SNS integrates with a lot of AWS services

- Many AWS services can send data directly to SNS for notifications
…
CloudWatch Alarms AWS Budgets
Lambda
…
Auto Scaling Group
(Notifications)
S3 Bucket
(Events)
DynamoDB
…
CloudFormation
(State Changes)
AWS DMS
(New Replic)
publish
RDS Events
SNS

<!-- Page 397 -->

# Amazon SNS – How to publish

- Topic Publish (using the SDK)
- Create a topic
- Create a subscription (or many)
- Publish to the topic
- Direct Publish (for mobile apps SDK)
- Create a platform application
- Create a platform endpoint
- Publish to the platform endpoint
- Works with Google GCM, Apple APNS, Amazon ADM…

<!-- Page 398 -->

# Amazon SNS – Security

- Encryption:
- In-flight encryption using HTTPS API
- At-rest encryption using KMS keys
- Client-side encryption if the client wants to perform encryption/decryption itself
- Access Controls: IAM policies to regulate access to the SNS API
- SNS Access Policies (similar to S3 bucket policies)
- Useful for cross-account access to SNS topics
- Useful for allowing other services ( S3…) to write to an SNS topic

<!-- Page 399 -->

# SNS + SQS: Fan Out

SQS Queue
Fraud
Service
Buying
Service
Shipping
Service
SNS Topic
SQS Queue
- Push once in SNS, receive in all SQS queues that are subscribers
- Fully decoupled, no data loss
- SQS allows for: data persistence, delayed processing and retries of work
- Ability to add more SQS subscribers over time
- Make sure your SQS queue access policy allows for SNS to write
- Cross-Region Delivery: works with SQS Queues in other regions

<!-- Page 400 -->

# Application: S3 Events to multiple queues

- For the same combination of: event type (e.g. object create) and prefix
(e.g. images/) you can only have one S3 Event rule
- If you want to send the same S3 event to many SQS queues, use fan-out
SQS Queues
Fan-out
S3 Object
created…
events
Amazon S3
SNS Topic
Lambda Function

<!-- Page 401 -->

# Kinesis Data Firehose

- SNS can send to Kinesis and therefore we can have the following
solutions architecture:
Buying
Service
Amazon S3
SNS Topic
Kinesis Data
Firehose
Any supported KDF
Destination
Application: SNS to Amazon S3 through

<!-- Page 402 -->

# Amazon SNS – FIFO Topic

- FIFO = First In First Out (ordering of messages in the topic)
Producer
Receive messages
Send messages
4
3
2
1
4
3
2
1
Subscribers
SQS FIFO
- Similar features as SQS FIFO:
- Ordering by Message Group ID (all messages in the same group are ordered)
- Deduplication using a Deduplication ID or Content Based Deduplication
- Can have SQS Standard and FIFO queues as subscribers
- Limited throughput (same throughput as SQS FIFO)

<!-- Page 403 -->

# SNS FIFO + SQS FIFO: Fan Out

- In case you need fan out + ordering + deduplication
SQS FIFO Queue
Fraud
Service
Buying
Service
Shipping
Service
SNS FIFO Topic
SQS FIFO Queue

<!-- Page 404 -->

# SNS – Message Filtering

- JSON policy used to filter messages sent to SNS topic’s subscriptions
- If a subscription doesn’t have a filter policy, it receives every message
Filter Policy
State: Placed
Buying
Service
Filter Policy
New transaction
Order: 1036
Product: Pencil
Qty: 4
State: Placed
State: Cancelled
SNS Topic
SQS Queue
(Placed orders)
SQS Queue
(Cancelled orders)
Email Subscription
(Cancelled orders)
Filter Policy
State: Declined
SQS Queue
(Declined orders)
SQS Queue
(All)

<!-- Page 405 -->

# Amazon Kinesis Data Streams

- Collect and store streaming data in real-time
Real-time data
Click Streams
Consumers
Producers
Applications
IoT devices
Kinesis Agent
Metrics & Logs
Amazon Kinesis
Data Streams
Application
Lambda
Amazon
Data Firehose
Managed
Service for
Apache Flink

<!-- Page 406 -->

# Kinesis Data Streams

- Retention between up to 365 days
- Ability to reprocess (replay) data by consumers
- Data can’t be deleted from Kinesis (until it expires)
- Data up to 10MiB (typical use case is lot of “small” real-time data)
- Data ordering guarantee for data with the same “Partition ID”
- At-rest KMS encryption, in-flight HTTPS encryption
- Kinesis Producer Library (KPL) to write an optimized producer application
- Kinesis Client Library (KCL) to write an optimized consumer application

<!-- Page 407 -->

# Kinesis Data Streams – Capacity Modes

- Provisioned mode:
- Choose number of shards
- Each shard gets 1MB/s in (or 1000 records per second)
- Each shard gets 2MB/s out
- Scale manually to increase or decrease the number of shards
- You pay per shard provisioned per hour
- On-demand mode:
- No need to provision or manage the capacity
- Default capacity provisioned (4 MB/s in or 4000 records per second)
- Scales automatically based on observed throughput peak during the last 30 days
- Pay per stream per hour & data in/out per GB

<!-- Page 408 -->

# Amazon Data Firehose

3rd-party Partner Destinations
Lambda
function
Applications
Kinesis
Data Streams
Client
Datadog
Data
transformation
Amazon S3
Record
AWS Destinations
Up to 1 MB
Amazon
CloudWatch
(Logs & Events)
SDK
Kinesis Agent
Producers
AWS IoT
Batch writes
Amazon
Data Firehose
All or Failed data
Amazon Redshift
Amazon OpenSearch
Custom Destinations
S3 backup bucket
HTTP Endpoint

<!-- Page 409 -->

# Amazon Data Firehose

- Note: used to be called “Kinesis Data Firehose”
- Fully Managed Service
- Amazon Redshift / Amazon S3 / Amazon OpenSearch Service
- 3rd party: Splunk / MongoDB / Datadog / NewRelic / …
- Custom HTTP Endpoint
- Automatic scaling, serverless, pay for what you use
- Near Real-Time with buffering capability based on size / time
- Supports CSV, JSON, Parquet, Avro, Raw Text, Binary data
- Conversions to Parquet / ORC, compressions with gzip / snappy
- Custom data transformations using AWS Lambda (ex: CSV to JSON)

<!-- Page 410 -->

# Kinesis Data Streams vs Amazon Data Firehose

Kinesis Data Streams
- Streaming data collection
- Producer & Consumer code
- Real-time
- Provisioned / On-Demand mode
- Data storage up to 365 days
- Replay Capability
Amazon Data Firehose
- Load streaming data into S3 / Redshift /
OpenSearch / 3rd party / custom HTTP
- Fully managed
- Near real-time
- Automatic scaling
- No data storage
- Doesn’t support replay capability

<!-- Page 411 -->

# SQS vs SNS vs Kinesis

SQS:
- Consumer “pull data”
- Data is deleted after being
consumed
- Can have as many workers
(consumers) as we want
- No need to provision
throughput
- Ordering guarantees only on
FIFO queues
- Individual message delay
capability
SNS:
- Push data to many
subscribers
- Up to 12,500,000 subscribers
- Data is not persisted (lost if
not delivered)
- Pub/Sub
- Up to 100,000 topics
- No need to provision
throughput
- Integrates with SQS for fanout architecture pattern
- FIFO capability for SQS FIFO
Kinesis:
- Standard: pull data
- 2 MB per shard
- Enhanced-fan out: push data
- 2 MB per shard per consumer
- Possibility to replay data
- Meant for real-time big data,
analytics and ETL
- Ordering at the shard level
- Data expires after X days
- Provisioned mode or ondemand capacity mode

<!-- Page 412 -->

# Amazon MQ

- SQS, SNS are “cloud-native” services: proprietary protocols from AWS
- Traditional applications running from on-premises may use open protocols
such as: MQTT, AMQP, STOMP, Openwire, WSS
- When migrating to the cloud, instead of re-engineering the application to use
SQS and SNS, we can use Amazon MQ
- Amazon MQ is a managed message broker service for
- Amazon MQ doesn’t “scale” as much as SQS / SNS
- Amazon MQ runs on servers, can run in Multi-AZ with failover
- Amazon MQ has both queue feature (~SQS) and topic features (~SNS)

<!-- Page 413 -->

# Amazon MQ – High Availability

Region
(us-east-1)
ACTIVE
Availability Zone
(us-east-1a)
Amazon MQ Broker
Client
failover
STANDBY
Availability Zone
(us-east-1b)
Amazon MQ Broker
Amazon EFS
(storage)

<!-- Page 414 -->

# Containers on AWS

<!-- Page 415 -->

# What is Docker?

- Docker is a software development platform to deploy apps
- Apps are packaged in containers that can be run on any OS
- Apps run the same, regardless of where they’re run
- Any machine
- No compatibility issues
- Predictable behavior
- Less work
- Easier to maintain and deploy
- Works with any language, any OS, any technology
- Use cases: microservices architecture, lift-and-shift apps from onpremises to the AWS cloud, …

<!-- Page 416 -->

# Docker on an OS

Server (e.g., EC2 instance)

<!-- Page 417 -->

# Where are Docker images stored?

- Docker images are stored in Docker Repositories
- Docker Hub (https://hub.docker.com)
- Public repository
- Find base images for many technologies or OS (e.g., Ubuntu, MySQL, …)
- Amazon ECR (Amazon Elastic Container Registry)
- Private repository
- Public repository (Amazon ECR Public Gallery https://gallery.ecr.aws)

<!-- Page 418 -->

# Docker vs. Virtual Machines

- Docker is ”sort of ” a virtualization technology, but not exactly
- Resources are shared with the host => many containers on one server
Apps
Apps
Apps
Guest OS
(VM)
Guest OS
(VM)
Guest OS
(VM)
Hypervisor
Docker Daemon
Host OS
Host OS (EC2 Instance)
Infrastructure
Infrastructure

<!-- Page 419 -->

# Getting Started with Docker

Build
Run
Dockerfile
image
Push
Pull
Docker Repository
Amazon
ECR
container

<!-- Page 420 -->

# Docker Containers Management on AWS

- Amazon Elastic Container Service (Amazon ECS)
- Amazon’s own container platform
- Amazon Elastic Kubernetes Service (Amazon EKS)
- Amazon’s managed Kubernetes (open source)
- AWS Fargate
- Amazon’s own Serverless container platform
- Works with ECS and with EKS
- Amazon ECR:
- Store container images
Amazon ECS
Amazon EKS
AWS Fargate
Amazon ECR

<!-- Page 421 -->

# Amazon ECS - EC2 Launch Type

- ECS = Elastic Container Service
- Launch Docker containers on AWS =
Launch ECS Tasks on ECS Clusters
- EC2 Launch Type: you must provision
& maintain the infrastructure (the EC2
instances)
- Each EC2 Instance must run the ECS
Agent to register in the ECS Cluster
- AWS takes care of starting / stopping
containers
Amazon ECS / ECS Cluster
New Docker
Container
EC2 Instance
EC2 Instance
EC2 Instance
ECS Agent
ECS Agent
ECS Agent

<!-- Page 422 -->

# Amazon ECS – Fargate Launch Type

- Launch Docker containers on AWS
- You do not provision the infrastructure
(no EC2 instances to manage)
- It’s all Serverless!
- You just create task definitions
- AWS just runs ECS Tasks for you based
on the CPU / RAM you need
- To scale, just increase the number of
tasks. Simple - no more EC2 instances
New Docker
Container
AWS Fargate / ECS Cluster

<!-- Page 423 -->

# DynamoDB

EC2 Instance Profile
- EC2 Instance Profile (EC2 Launch Type only):
- Used by the ECS agent
- Makes API calls to ECS service
- Send container logs to CloudWatch Logs
- Pull Docker image from ECR
- Reference sensitive data in Secrets Manager or
SSM Parameter Store
- ECS Task Role:
- Allows each task to have a specific role
- Use different roles for the different ECS Services
you run
- Task Role is defined in the task definition
ECS
EC2 Instance
ECR
ECS Agent
CloudWatch
Logs
ECS Task A Role
S3
Task A
ECS Task B Role
Task B
Amazon ECS – IAM Roles for ECS

<!-- Page 424 -->

# Amazon ECS – Load Balancer Integrations

- Application Load Balancer supported
and works for most use cases
- Network Load Balancer recommended
only for high throughput / high
performance use cases, or to pair it with
AWS Private Link
- Classic Load Balancer supported but not
recommended (no advanced features –
no Fargate)
EC2 Instance
ECS Task
ECS Task
80/443
Users
EC2 Instance
Application
Load Balancer
ECS Task
ECS Task
ECS Cluster

<!-- Page 425 -->

# Amazon ECS – Data Volumes (EFS)

- Mount EFS file systems onto ECS tasks
- Works for both EC2 and Fargate launch types
- Tasks running in any AZ will share the same data
in the EFS file system
- Fargate + EFS = Serverless
- Use cases: persistent multi-AZ shared storage for
your containers
- Note:
- Amazon S3 cannot be mounted as a file system
EC2 Instance
mount
mount
ECS Cluster
Amazon EFS
File System
Fargate

<!-- Page 426 -->

# ECS Service Auto Scaling

- Automatically increase/decrease the desired number of ECS tasks
- Amazon ECS Auto Scaling uses AWS Application Auto Scaling
- ECS Service Average CPU Utilization
- ECS Service Average Memory Utilization - Scale on RAM
- ALB Request Count Per Target – metric coming from the ALB
- Target Tracking – scale based on target value for a specific CloudWatch metric
- Step Scaling – scale based on a specified CloudWatch Alarm
- Scheduled Scaling – scale based on a specified date/time (predictable changes)
- ECS Service Auto Scaling (task level) ≠ EC2 Auto Scaling (EC2 instance level)
- Fargate Auto Scaling is much easier to setup (because Serverless)

<!-- Page 427 -->

# EC2 Launch Type – Auto Scaling EC2 Instances

- Accommodate ECS Service Scaling by adding underlying EC2 Instances
- Auto Scaling Group Scaling
- Scale your ASG based on CPU Utilization
- Add EC2 instances over time
- ECS Cluster Capacity Provider
- Used to automatically provision and scale the infrastructure for your ECS Tasks
- Capacity Provider paired with an Auto Scaling Group
- Add EC2 Instances when you’re missing capacity (CPU, RAM…)

<!-- Page 428 -->

# ECS Scaling – Service CPU Usage Example

CPU
Usage
Task 1
Task 3
(new)
Task 2
Service A
Auto Scaling
Auto Scaling Group
CloudWatch Metric
(ECS Service CPU Usage)
Trigger
CloudWatch Alarm
le
a
c
S
Scale ECS Capacity Providers
(optional)

<!-- Page 429 -->

# ECS tasks invoked by Event Bridge

Region
VPC
Upload object
Client
Ge
to
bje
S3 Bucket
Event
Rul
Amazon
EventBridge
e
n
: Ru
ECS
AWS Fargate
ct
k
Tas
Task
(new)
ECS Task Role
(Access S3 & DynamoDB)
Save result
Amazon
DynamoDB
Amazon ECS Cluster

<!-- Page 430 -->

# ECS tasks invoked by Event Bridge Schedule

AWS Fargate
Every 1 hour
Amazon
EventBridge
Task
(new)
Rule: Run ECS Task
ECS Task Role
Access S3
Batch Processing
Amazon S3
Amazon ECS Cluster

<!-- Page 431 -->

# ECS – SQS Queue Example

Messages
Poll for messages
Task 1
Task 3
SQS Queue
Task 2
Service A
ECS Service Auto Scaling

<!-- Page 432 -->

# ECS – Intercept Stopped Tasks using EventBridge

ECS Task
event
email
trigger
exited
Containers
EventBridge
Event Pattern
SNS
Administrator

<!-- Page 433 -->

# Amazon ECR

- ECR = Elastic Container Registry
- Store and manage Docker images on AWS
- Private and Public repository (Amazon ECR
Public Gallery https://gallery.ecr.aws)
- Fully integrated with ECS, backed by Amazon S3
- Access is controlled through IAM (permission
errors => policy)
- Supports image vulnerability scanning, versioning,
image tags, image lifecycle, …
ECR Repository
Docker
Image A
Docker
Image B
pull
pull
IAM Role
EC2 Instance
ECS Cluster

<!-- Page 434 -->

# Amazon EKS Overview

- Amazon EKS = Amazon Elastic Kubernetes Service
- It is a way to launch managed Kubernetes clusters on AWS
- Kubernetes is an open-source system for automatic deployment, scaling and
management of containerized (usually Docker) application
- It’s an alternative to ECS, similar goal but different API
- EKS supports EC2 if you want to deploy worker nodes or Fargate to deploy
serverless containers
- Use case: if your company is already using Kubernetes on-premises or in
another cloud, and wants to migrate to AWS using Kubernetes
- Kubernetes is cloud-agnostic (can be used in any cloud – Azure, GCP…)
- For multiple regions, deploy one EKS cluster per region
- Collect logs and metrics using CloudWatch Container Insights

<!-- Page 435 -->

# Amazon EKS - Diagram

AWS Cloud
Availability Zone 1
VPC
Availability Zone 2
Public subnet 1
ELB
Public subnet 2
NGW
EKS
Public
Service LB
Public subnet 3
NGW
ELB
Private subnet 2
Private subnet 1
EKS node
Availability Zone 3
ELB
Private subnet 3 EKS
Private
Service LB
EKS node
EKS node
EKS Pods
EKS Pods
Auto Scaling Group
EKS Pods
EKS Worker Nodes
NGW
ELB

<!-- Page 436 -->

# Amazon EKS – Node Types

- Managed Node Groups
- Creates and manages Nodes (EC2 instances) for you
- Nodes are part of an ASG managed by EKS
- Supports On-Demand or Spot Instances
- Self-Managed Nodes
- Nodes created by you and registered to the EKS cluster and managed by an ASG
- You can use prebuilt AMI - Amazon EKS Optimized AMI
- Supports On-Demand or Spot Instances
- AWS Fargate
- No maintenance required; no nodes managed

<!-- Page 437 -->

# Amazon EKS – Data Volumes

- Need to specify StorageClass manifest on your EKS cluster
- Leverages a Container Storage Interface (CSI) compliant driver
- Support for…
- Amazon EBS
- Amazon EFS (works with Fargate)
- Amazon FSx for Lustre
- Amazon FSx for NetApp ONTAP

<!-- Page 438 -->

# Serverless Overview

<!-- Page 439 -->

# What’s serverless?

- Serverless is a new paradigm in which the developers don’t have to
manage servers anymore…
- They just deploy code
- They just deploy… functions !
- Initially... Serverless == FaaS (Function as a Service)
- Serverless was pioneered by AWS Lambda but now also includes
anything that’s managed: “databases, messaging, storage, etc.”
- Serverless does not mean there are no servers…
it means you just don’t manage / provision / see them

<!-- Page 440 -->

# Serverless in AWS

co
nt
en
t
Sta
tic
REST API
Lambda
DynamoDB
n
API Gateway
gi
S3 bucket
Lo
- AWS Lambda
- DynamoDB
- AWS Cognito
- AWS API Gateway
- Amazon S3
- AWS SNS & SQS
- AWS Kinesis Data Firehose
- Aurora Serverless
- Step Functions
- Fargate
Users
Cognito

<!-- Page 441 -->

# Why AWS Lambda

Amazon EC2
Amazon Lambda
- Virtual Servers in the Cloud
- Limited by RAM and CPU
- Continuously running
- Scaling means intervention to add / remove servers
- Virtual functions – no servers to manage!
- Limited by time - short executions
- Run on-demand
- Scaling is automated!

<!-- Page 442 -->

# Benefits of AWS Lambda

- Easy Pricing:
- Pay per request and compute time
- Free tier of 1,000,000 AWS Lambda requests and 400,000 GBs of compute time
- Integrated with the whole AWS suite of services
- Integrated with many programming languages
- Easy monitoring through AWS CloudWatch
- Easy to get more resources per functions (up to 10GB of RAM!)
- Increasing RAM will also improve CPU and network!

<!-- Page 443 -->

# AWS Lambda language support

- Node.js (JavaScript)
- Python
- Java
- C# (.NET Core) / Powershell
- Ruby
- Custom Runtime API (community supported, example Rust or Golang)
- Lambda Container Image
- The container image must implement the Lambda Runtime API
- ECS / Fargate is preferred for running arbitrary Docker images

<!-- Page 444 -->

# Main ones

API Gateway
Kinesis
DynamoDB
S3
CloudFront
CloudWatch Events
EventBridge
CloudWatch Logs
SNS
SQS
Cognito
AWS Lambda Integrations

<!-- Page 445 -->

# Example: Serverless Thumbnail creation

sh
u
p
New thumbnail in S3
trigger
pu
AWS Lambda Function
Creates a Thumbnail
sh
New image in S3
Image name
Image size
Creation date
etc…
Metadata in DynamoDB

<!-- Page 446 -->

# Example: Serverless CRON Job

Trigger
Every 1 hour
CloudWatch Events
EventBridge
AWS Lambda Function
Perform a task

<!-- Page 447 -->

# AWS Lambda Pricing: example

- You can find overall pricing information here:
https://aws.amazon.com/lambda/pricing/
- Pay per calls:
- First 1,000,000 requests are free
- $0.20 per 1 million requests thereafter ($0.0000002 per request)
- Pay per duration: (in increment of 1 ms)
- 400,000 GB-seconds of compute time per month for FREE
- == 400,000 seconds if function is 1GB RAM
- == 3,200,000 seconds if function is 128 MB RAM
- After that $1.00 for 600,000 GB-seconds
- It is usually very cheap to run AWS Lambda so it’s very popular

<!-- Page 448 -->

# AWS Lambda Limits to Know - per region

- Execution:
- Memory allocation: 128 MB – 10GB (1 MB increments)
- Maximum execution time: 900 seconds (15 minutes)
- Environment variables (4 KB)
- Disk capacity in the “function container” (in /tmp): 512 MB to 10GB
- Concurrency executions: 1000 (can be increased)
- Deployment:
- Lambda function deployment size (compressed .zip): 50 MB
- Size of uncompressed deployment (code + dependencies): 250 MB
- Can use the /tmp directory to load other files at startup
- Size of environment variables: 4 KB

<!-- Page 449 -->

# Lambda Concurrency and Throttling

- Concurrency limit: up to 1000 concurrent executions
- Can set a “reserved concurrency” at the function level (=limit)
- Each invocation over the concurrency limit will trigger a “Throttle”
- Throttle behavior:
- If synchronous invocation => return ThrottleError - 429
- If asynchronous invocation => retry automatically and then go to DLQ
- If you need a higher limit, open a support ticket

<!-- Page 450 -->

# Lambda Concurrency Issue

- If you don’t reserve (=limit) concurrency, the following can happen:
1000 concurrent
executions
Many users
Application Load Balancer
THROTTLE!
Few users
SDK / CLI
API Gateway
THROTTLE!

<!-- Page 451 -->

# Concurrency and Asynchronous Invocations

New file event
New file event
S3 bucket
New file event
- If the function doesn't have enough
concurrency available to process all
events, additional requests are
throttled.
- For throttling errors (429) and
system errors (500-series), Lambda
returns the event to the queue and
attempts to run the function again
for up to 6 hours.
- The retry interval increases
exponentially from 1 second after
the first attempt to a maximum of
5 minutes.

<!-- Page 452 -->

# Cold Starts & Provisioned Concurrency

- Cold Start:
- New instance => code is loaded and code outside the handler run (init)
- If the init is large (code, dependencies, SDK…) this process can take some time.
- First request served by new instances has higher latency than the rest
- Provisioned Concurrency:
- Concurrency is allocated before the function is invoked (in advance)
- So the cold start never happens and all invocations have low latency
- Application Auto Scaling can manage concurrency (schedule or target utilization)
- Note:
- Note: cold starts in VPC have been dramatically reduced in Oct & Nov 2019
- https://aws.amazon.com/blogs/compute/announcing-improved-vpc-networking-for-aws-lambda-functions/

<!-- Page 453 -->

# Reserved and Provisioned Concurrency

https://docs.aws.amazon.com/lambda/latest/dg/configuration-concurrency.html

<!-- Page 454 -->

# Lambda SnapStart

- Improves your Lambda functions performance
up to 10x at no extra cost for Java, Python & .NET
- When enabled, function is invoked from a preinitialized state (no function initialization from
scratch)
- When you publish a new version:
- Lambda initializes your function
- Takes a snapshot of memory and disk state of the
initialized function
- Snapshot is cached for low-latency access
SnapStart
enabled
SnapStart
disabled
invoke
invoke
Lambda
function is
pre-initialized
Lambda
Init
Invoke
Invoke
Shutdown
Shutdown
Lambda Invocation
Lifecycle Phases

<!-- Page 455 -->

# Customization At The Edge

- Many modern applications execute some form of the logic at the edge
- Edge Function:
- A code that you write and attach to CloudFront distributions
- Runs close to your users to minimize latency
- CloudFront provides two types: CloudFront Functions & Lambda@Edge
- You don’t have to manage any servers, deployed globally
- Use case: customize the CDN content
- Pay only for what you use
- Fully serverless

<!-- Page 456 -->

# Use Cases

- Website Security and Privacy
- Dynamic Web Application at the Edge
- Search Engine Optimization (SEO)
- Intelligently Route Across Origins and Data Centers
- Bot Mitigation at the Edge
- Real-time Image Transformation
- A/B Testing
- User Authentication and Authorization
- User Prioritization
- User Tracking and Analytics
CloudFront Functions & Lambda@Edge

<!-- Page 457 -->

# CloudFront Functions

- Lightweight functions written in JavaScript
- For high-scale, latency-sensitive CDN customizations
- Sub-ms startup times, millions of requests/second
- Used to change Viewer requests and responses:
- Viewer Request: after CloudFront receives a request from a
viewer
- Viewer Response: before CloudFront forwards the response
to the viewer
Client
Viewer
Request
Viewer
Response
CloudFront
Origin
Request
Origin
Response
- Native feature of CloudFront (manage code entirely
within CloudFront)
Origin

<!-- Page 458 -->

# Lambda@Edge

- Lambda functions written in NodeJS or Python
- Scales to 1000s of requests/second
- Used to change CloudFront requests and responses:
- Viewer Request – after CloudFront receives a request from a
viewer
- Origin Request – before CloudFront forwards the request to the
origin
- Origin Response – after CloudFront receives the response from the
origin
- Viewer Response – before CloudFront forwards the response to
the viewer
Client
Viewer
Request
Viewer
Response
CloudFront
Origin
Request
Origin
Response
- Author your functions in one AWS Region (us-east-1), then
CloudFront replicates to its locations
Origin

<!-- Page 459 -->

# CloudFront Functions vs. Lambda@Edge

CloudFront Functions
Lambda@Edge
Runtime Support
JavaScript
Node.js, Python
# of Requests
Millions of requests per second
Thousands of requests per second
CloudFront Triggers
-
-
Max. Execution Time
< 1 ms
5 – 10 seconds
Max. Memory
2 MB
128 MB up to 10 GB
Total Package Size
10 KB
1 MB – 50 MB
Network Access, File System Access
No
Yes
Access to the Request Body
No
Yes
Pricing
Free tier available, 1/6th price of @Edge
No free tier, charged per request & duration
Viewer Request/Response
Viewer Request/Response
Origin Request/Response

<!-- Page 460 -->

# CloudFront Functions vs. Lambda@Edge - Use Cases

CloudFront Functions
- Cache key normalization
- Transform request attributes (headers,
cookies, query strings, URL) to create an
optimal Cache Key
- Header manipulation
- Insert/modify/delete HTTP headers in the
request or response
- URL rewrites or redirects
- Request authentication & authorization
- Create and validate user-generated
tokens (e.g., JWT) to allow/deny requests
Lambda@Edge
- Longer execution time (several ms)
- Adjustable CPU or memory
- Your code depends on a 3rd
libraries (e.g., AWS SDK to access
other AWS services)
- Network access to use external
services for processing
- File system access or access to the
body of HTTP requests

<!-- Page 461 -->

# Lambda by default

- By default, your Lambda function is
launched outside your own VPC (in
an AWS-owned VPC)
- Therefore, it cannot access resources
in your VPC (RDS, ElastiCache,
internal ELB…)
Default Lambda Deployment
AWS Cloud
Public
www
works
DynamoDB
VPC & Private Subnet
Not working
Private RDS

<!-- Page 462 -->

# Lambda in VPC

- You must define the VPC ID, the
Subnets and the Security Groups
- Lambda will create an ENI (Elastic
Network Interface) in your subnets
Lambda Function
Private subnet
Lambda Security group
Elastic Network
Interface (ENI)
RDS Security group
Amazon RDS
In VPC

<!-- Page 463 -->

# Lambda with RDS Proxy

- If Lambda functions directly access your
database, they may open too many
connections under high load
- RDS Proxy
- Improve scalability by pooling and sharing DB
connections
- Improve availability by reducing by 66% the
failover time and preserving connections
- Improve security by enforcing IAM
authentication and storing credentials in
Secrets Manager
- The Lambda function must be deployed in
your VPC, because RDS Proxy is never
publicly accessible
VPC
Lambda functions
…
Private subnet
RDS Proxy
RDS DB
Instance

<!-- Page 464 -->

# Amazon SES

User
- Invoke Lambda functions from within your DB instance
- Allows you to process data events from within a database
- Supported for RDS for PostgreSQL and Aurora MySQL
- Must allow outbound traffic to your Lambda function from
within your DB instance (Public, NAT GW, VPC
Endpoints)
- DB instance must have the required permissions to invoke
the Lambda function (Lambda Resource-based Policy &
IAM Policy)
register
(INSERT)
RDS DB
Instance
Invoking Lambda from RDS & Aurora
Permissions
invoke
Lambda
function
send Email

<!-- Page 465 -->

# function

- Notifications that tells information about the DB
instance itself (created, stopped, start, …)
- You don’t have any information about the data
itself
- Subscribe to the following event categories: DB
instance, DB snapshot, DB Parameter Group, DB
Security Group, RDS Proxy, Custom Engine Version
- Near real-time events (up to 5 minutes)
- Send notifications to SNS or subscribe to events
using EventBridge
…
RDS DB
Instance
SNS
SQS
Queue
RDS Event Notifications
EventBridge
Lambda
Lambda

<!-- Page 466 -->

# Amazon DynamoDB

- Fully managed, highly available with replication across multiple AZs
- NoSQL database - not a relational database - with transaction support
- Scales to massive workloads, distributed database
- Millions of requests per seconds, trillions of row, 100s of TB of storage
- Fast and consistent in performance (single-digit millisecond)
- Integrated with IAM for security, authorization and administration
- Low cost and auto-scaling capabilities
- No maintenance or patching, always available
- Standard & Infrequent Access (IA) Table Class

<!-- Page 467 -->

# DynamoDB - Basics

- DynamoDB is made of Tables
- Each table has a Primary Key (must be decided at creation time)
- Each table can have an infinite number of items (= rows)
- Each item has attributes (can be added over time – can be null)
- Maximum size of an item is 400KB
- Data types supported are:
- Scalar Types – String, Number, Binary, Boolean, Null
- Document Types – List, Map
- Set Types – String Set, Number Set, Binary Set
- Therefore, in DynamoDB you can rapidly evolve schemas

<!-- Page 468 -->

# DynamoDB – Table example

Primary Key
Attributes
Partition Key
Sort Key
User_ID
Game_ID
Score
Result
7791a3d6-…
4421
92
Win
873e0634-…
1894
14
Lose
873e0634-…
4521
77
Win

<!-- Page 469 -->

# DynamoDB – Read/Write Capacity Modes

- Control how you manage your table’s capacity (read/write throughput)
- Provisioned Mode (default)
- You specify the number of reads/writes per second
- You need to plan capacity beforehand
- Pay for provisioned Read Capacity Units (RCU) & Write Capacity Units (WCU)
- Possibility to add auto-scaling mode for RCU & WCU
- On-Demand Mode
- Read/writes automatically scale up/down with your workloads
- No capacity planning needed
- Pay for what you use, more expensive ($$$)
- Great for unpredictable workloads, steep sudden spikes

<!-- Page 470 -->

# DynamoDB Accelerator (DAX)

Application
- Fully-managed, highly available, seamless inmemory cache for DynamoDB
- Help solve read congestion by caching
- Microseconds latency for cached data
- Doesn’t require application logic modification
(compatible with existing DynamoDB APIs)
- 5 minutes TTL for cache (default)
DAX Cluster
…
Nodes
Amazon DynamoDB
…
Tables

<!-- Page 471 -->

# DynamoDB Accelerator (DAX) vs. ElastiCache

Amazon
ElastiCache
Store Aggregation Result
Application
- Individual objects cache
- Query & Scan cache
Amazon
DynamoDB
DynamoDB Accelerator (DAX)

<!-- Page 472 -->

# DynamoDB – Stream Processing

- Ordered stream of item-level modifications (create/update/delete) in a table
- Use cases:
- React to changes in real-time (welcome email to users)
- Real-time usage analytics
- Insert into derivative tables
- Implement cross-region replication
- Invoke AWS Lambda on changes to your DynamoDB table
DynamoDB Streams
- 24 hours retention
- Limited # of consumers
- Process using AWS Lambda Triggers, or
DynamoDB Stream Kinesis adapter
Kinesis Data Streams (newer)
- 1 year retention
- High # of consumers
- Process using AWS Lambda, Kinesis Data
Analytics, Kineis Data Firehose, AWS Glue
Streaming ETL…

<!-- Page 473 -->

# DynamoDB Streams

Processing Layer
DynamoDB
KCL Adapter
filtering, transforming, …
Lambda
create/update/delete
Application
messaging, notifications
Table
DynamoDB
Streams
analytics
archiving
Kinesis Data
Streams
DDB Table
Amazon
Redshift
Amazon S3
Kinesis Data
Firehose
indexing
Amazon SNS
Amazon
OpenSearch

<!-- Page 474 -->

# DynamoDB Global Tables

GLOBAL TABLE
Table
Table
US-EAST-1
AP-SOUTHEAST-2
two-way
replication
- Make a DynamoDB table accessible with low latency in multiple-regions
- Active-Active replication
- Applications can READ and WRITE to the table in any region
- Must enable DynamoDB Streams as a pre-requisite

<!-- Page 475 -->

# DynamoDB – Time To Live (TTL)

Current Time
Friday, September 10, 2021, 11:56:11 AM
(Epoch timestamp: 1631274971)
Expiration Process
- Automatically delete items after an expiry
timestamp
- Use cases: reduce stored data by keeping only
current items, adhere to regulatory
obligations, web session handling…
scan &
expire items
SessionData (Table)
User_ID
Session_ID
ExpTime (TTL)
7791a3d6-…
74686572652
1631188571
873e0634-…
6e6f7468696
1631274971
a80f73a1-…
746f2073656
1631102171
scan &
delete items
Deletion Process

<!-- Page 476 -->

# DynamoDB – Backups for disaster recovery

- Continuous backups using point-in-time recovery (PITR)
- Optionally enabled for the last 35 days
- Point-in-time recovery to any time within the backup window
- The recovery process creates a new table
- On-demand backups
- Full backups for long-term retention, until explicitely deleted
- Doesn’t affect performance or latency
- Can be configured and managed in AWS Backup (enables cross-region copy)
- The recovery process creates a new table

<!-- Page 477 -->

# DynamoDB – Integration with Amazon S3

- Export to S3 (must enable PITR)
- Works for any point of time in the last 35 days
- Doesn’t affect the read capacity of your table
- Perform data analysis on top of DynamoDB
- Retain snapshots for auditing
- ETL on top of S3 data before importing back into
DynamoDB
- Export in DynamoDB JSON or ION format
export
DynamoDB
query
S3
Athena
- Import from S3
- Import CSV, DynamoDB JSON or ION format
- Doesn’t consume any write capacity
- Creates a new table
- Import errors are logged in CloudWatch Logs
import
S3
(.csv, .json, .ion)
DynamoDB

<!-- Page 478 -->

# Example: Building a Serverless API

REST API
Client
PROXY REQUESTS
API Gateway
CRUD
Lambda
DynamoDB

<!-- Page 479 -->

# AWS API Gateway

- AWS Lambda + API Gateway: No infrastructure to manage
- Support for the WebSocket Protocol
- Handle API versioning (v1, v2…)
- Handle different environments (dev, test, prod…)
- Handle security (Authentication and Authorization)
- Create API keys, handle request throttling
- Swagger / Open API import to quickly define APIs
- Transform and validate requests and responses
- Generate SDK and API specifications
- Cache API responses

<!-- Page 480 -->

# API Gateway – Integrations High Level

- Lambda Function
- Invoke Lambda function
- Easy way to expose REST API backed by AWS Lambda
- HTTP
- Expose HTTP endpoints in the backend
- Example: internal HTTP API on premise, Application Load Balancer…
- Why? Add rate limiting, caching, user authentications, API keys, etc…
- AWS Service
- Expose any AWS API through the API Gateway
- Example: start an AWS Step Function workflow, post a message to SQS
- Why? Add authentication, deploy publicly, rate control…

<!-- Page 481 -->

# Kinesis Data Streams example

requests
Client
records
send
API Gateway
store .json
files
Kinesis Data
Streams
Kinesis Data
Firehose
Amazon S3
API Gateway – AWS Service Integration

<!-- Page 482 -->

# API Gateway - Endpoint Types

- Edge-Optimized (default): For global clients
- Requests are routed through the CloudFront Edge locations (improves latency)
- The API Gateway still lives in only one region
- Regional:
- For clients within the same region
- Could manually combine with CloudFront (more control over the caching
strategies and the distribution)
- Private:
- Can only be accessed from your VPC using an interface VPC endpoint (ENI)
- Use a resource policy to define access

<!-- Page 483 -->

# API Gateway – Security

- User Authentication through
- IAM Roles (useful for internal applications)
- Cognito (identity for external users – example mobile users)
- Custom Authorizer (your own logic)
- Custom Domain Name HTTPS security through integration with AWS
Certificate Manager (ACM)
- If using Edge-Optimized endpoint, then the certificate must be in us-east-1
- If using Regional endpoint, the certificate must be in the API Gateway region
- Must setup CNAME or A-alias record in Route 53

<!-- Page 484 -->

# AWS Step Functions

- Build serverless visual workflow to
orchestrate your Lambda functions
- Features: sequence, parallel, conditions,
timeouts, error handling, …
- Can integrate with EC2, ECS, On-premises
servers, API Gateway, SQS queues, etc…
- Possibility of implementing human approval
feature
- Use cases: order fulfillment, data processing,
web applications, any workflow

<!-- Page 485 -->

# Amazon Cognito

- Give users an identity to interact with our web or mobile application
- Cognito User Pools:
- Sign in functionality for app users
- Integrate with API Gateway & Application Load Balancer
- Cognito Identity Pools (Federated Identity):
- Provide AWS credentials to users so they can access AWS resources directly
- Integrate with Cognito User Pools as an identity provider
- Cognito vs IAM: “hundreds of users”, ”mobile users”, “authenticate with SAML”

<!-- Page 486 -->

# Cognito User Pools (CUP) – User Features

- Create a serverless database of user for your web & mobile apps
- Simple login: Username (or email) / password combination
- Password reset
- Email & Phone Number Verification
- Multi-factor authentication (MFA)
- Federated Identities: users from Facebook, Google, SAML…

<!-- Page 487 -->

# Cognito User Pools (CUP) - Integrations

- CUP integrates with API Gateway and Application Load Balancer
Cognito User Pools
Authenticate
Retrieve token
Cognito User Pools
Authenticate
Evaluate Cognito Token
REST API +
Pass Token
API Gateway
Application Load Balancer
+ Listeners & Rules
backend
Target Group
Backend

<!-- Page 488 -->

# Cognito Identity Pools (Federated Identities)

- Get identities for “users” so they obtain temporary AWS credentials
- Users source can be Cognito User Pools, 3rd party logins, etc…
- Users can then access AWS services directly or through API Gateway
- The IAM policies applied to the credentials are defined in Cognito
- They can be customized based on the user_id for fine grained control
- Default IAM roles for authenticated and guest users

<!-- Page 489 -->

# Cognito Identity Pools – Diagram

Login and Get Token
Web & Mobile
Applications
Exchange token
for temporary
AWS credentials
Cognito Identity Pools
Social Identity Provider
validate
Direct access to AWS
Cognito
User Pools
Private S3 Bucket
DynamoDB Table

<!-- Page 490 -->

# Row Level Security in DynamoDB

Cognito Identity Pools

<!-- Page 491 -->

# Serverless Architectures

<!-- Page 492 -->

# Mobile application: MyTodoList

- We want to create a mobile application with the following requirements
- Expose as REST API with HTTPS
- Serverless architecture
- Users should be able to directly interact with their own folder in S3
- Users should authenticate through a managed serverless service
- The users can write and read to-dos, but they mostly read them
- The database should scale, and have some high read throughput

<!-- Page 493 -->

# Mobile app: REST API layer

REST HTTPS
Amazon API Gateway
Mobile
client
AWS Lambda
Verify authentication
authenticate
Amazon Cognito
query
invoke
Amazon DynamoDB

<!-- Page 494 -->

# Mobile app: giving users access to S3

Store/retrieve files
Amazon S3
Permissions
Amazon API Gateway
Mobile
client
authenticate
Amazon Cognito
AWS Lambda
Amazon DynamoDB

<!-- Page 495 -->

# Mobile app: high read throughput, static data

Store/retrieve files
Amazon S3
Permissions
REST HTTPS
Amazon API Gateway
Mobile
client
AWS Lambda
Verify authentication
authenticate
Amazon Cognito
Query / read
invoke
DAX
Caching layer
DynamoDB

<!-- Page 496 -->

# Mobile app: caching at the API Gateway

Store/retrieve files
Amazon S3
Permissions
CACHING OF RESPONSES
REST HTTPS
Amazon API Gateway
Mobile
client
AWS Lambda
Verify authentication
authenticate
Amazon Cognito
Query / read
invoke
DAX
Caching layer
DynamoDB

<!-- Page 497 -->

# In this lecture

- Serverless REST API: HTTPS, API Gateway, Lambda, DynamoDB
- Using Cognito to generate temporary credentials to access S3 bucket
with restricted policy. App users can directly access AWS resources this
way. Pattern can be applied to DynamoDB, Lambda…
- Caching the reads on DynamoDB using DAX
- Caching the REST requests at the API Gateway level
- Security for authentication and authorization with Cognito

<!-- Page 498 -->

# Serverless hosted website: MyBlog.com

- This website should scale globally
- Blogs are rarely written, but often read
- Some of the website is purely static files, the rest is a dynamic REST API
- Caching must be implement where possible
- Any new users that subscribes should receive a welcome email
- Any photo uploaded to the blog should have a thumbnail generated

<!-- Page 499 -->

# Serving static content, globally

Interaction with
edge locations
Client
Amazon CloudFront
Global distribution
Amazon S3

<!-- Page 500 -->

# Serving static content, globally, securely

OAC: Origin Access Control
Interaction with
edge locations
Client
Amazon CloudFront
Global distribution
Amazon S3
Bucket policy
Only authorize from
CloudFront Distribution

<!-- Page 501 -->

# Adding a public serverless REST API

OAC: Origin Access Control
Interaction with
edge locations
REST HTTPS
Client
Amazon S3
Amazon CloudFront
Global distribution
Amazon API Gateway
Bucket policy
Only authorize from
CloudFront Distribution
Query / read
invoke
AWS Lambda
DAX
Caching layer
DynamoDB

<!-- Page 502 -->

# Leveraging DynamoDB Global Tables

OAC: Origin Access Control
Interaction with
edge locations
REST HTTPS
Client
Amazon S3
Amazon CloudFront
Global distribution
Amazon API Gateway
Bucket policy
Only authorize from
CloudFront Distribution
Query / read
invoke
AWS Lambda
DAX
Caching layer
DynamoDB
Global Tables

<!-- Page 503 -->

# User Welcome email flow

OAC: Origin Access Control
Interaction with
edge locations
Client
Amazon S3
Amazon CloudFront
Global distribution
REST HTTPS
Bucket policy
Only authorize from
CloudFront Distribution
Query / read
invoke
AWS Lambda
Amazon API Gateway
SDK to send email
DAX
Caching layer
IAM Role
DynamoDB
Stream changes
Invoke lambda
Amazon Simple
Email Service (SES)
AWS Lambda
DynamoDB
Stream

<!-- Page 504 -->

# Thumbnail Generation flow

OAC: Origin Access Control
Interaction with
edge locations
Client
Amazon S3
Amazon CloudFront
Global distribution
REST HTTPS
Query / read
invoke
Amazon API Gateway
Bucket policy
Only authorize from
CloudFront Distribution
AWS Lambda
DAX
Caching layer
DynamoDB
optional
Upload photos
Transfer acceleration
OAC
Amazon CloudFront
Global distribution
trigger
Amazon S3
thumbnail
AWS Lambda
SQS
Amazon S3
SNS

<!-- Page 505 -->

# AWS Hosted Website Summary

- We’ve seen static content being distributed using CloudFront with S3
- The REST API was serverless, didn’t need Cognito because public
- We leveraged a Global DynamoDB table to serve the data globally
- (we could have used Aurora Global Database)
- We enabled DynamoDB streams to trigger a Lambda function
- The lambda function had an IAM role which could use SES
- SES (Simple Email Service) was used to send emails in a serverless way
- S3 can trigger SQS / SNS / Lambda to notify of events

<!-- Page 506 -->

# Micro Services architecture

- We want to switch to a micro service architecture
- Many services interact with each other directly using a REST API
- Each architecture for each micro service may vary in form and shape
- We want a micro-service architecture so we can have a leaner
development lifecycle for each service

<!-- Page 507 -->

# Micro Services Environment

service1.example.com
DNS Query
Amazon Route 53
Elastic Load Balancing
ECS
DynamoDB
service2.example.com
HTTPS
Users
Amazon API Gateway
ElastiCache
service3.example.com
Elastic Load Balancing
AWS Lambda
Amazon EC2
Auto Scaling
Amazon RDS

<!-- Page 508 -->

# Discussions on Micro Services

- You are free to design each micro-service the way you want
- Synchronous patterns: API Gateway, Load Balancers
- Asynchronous patterns: SQS, Kinesis, SNS, Lambda triggers (S3)
- Challenges with micro-services:
- repeated overhead for creating each new microservice,
- issues with optimizing server density/utilization
- complexity of running multiple versions of multiple microservices simultaneously
- proliferation of client-side code requirements to integrate with many separate services.
- Some of the challenges are solved by Serverless patterns:
- API Gateway, Lambda scale automatically and you pay per usage
- You can easily clone API, reproduce environments
- Generated client SDK through Swagger integration for the API Gateway

<!-- Page 509 -->

# Software updates offloading

- We have an application running on EC2, that distributes software
updates once in a while
- When a new software update is out, we get a lot of request and the
content is distributed in mass over the network. It’s very costly
- We don’t want to change our application, but want to optimize our cost
and CPU, how can we do it?

<!-- Page 510 -->

# Our application current state

Auto Scaling group
Availability zone 1
Availability zone 1 to 3
Availability zone 2
Amazon Elastic
File System
Availability zone 3

<!-- Page 511 -->

# Easy way to fix things!

Auto Scaling group
Availability zone 1
Availability zone 1 to 3
Availability zone 2
Amazon Elastic
File System
Amazon CloudFront
Availability zone 3

<!-- Page 512 -->

# Why CloudFront?

- No changes to architecture
- Will cache software update files at the edge
- Software update files are not dynamic, they’re static (never changing)
- Our EC2 instances aren’t serverless
- But CloudFront is, and will scale for us
- Our ASG will not scale as much, and we’ll save tremendously in EC2
- We’ll also save in availability, network bandwidth cost, etc
- Easy way to make an existing application more scalable and cheaper!

<!-- Page 513 -->

# Databases in AWS

<!-- Page 514 -->

# Choosing the Right Database

- We have a lot of managed databases on AWS to choose from
- Questions to choose the right database based on your architecture:
- Read-heavy, write-heavy, or balanced workload? Throughput needs? Will it
change, does it need to scale or fluctuate during the day?
- How much data to store and for how long? Will it grow? Average object size?
How are they accessed?
- Data durability? Source of truth for the data ?
- Latency requirements? Concurrent users?
- Data model? How will you query the data? Joins? Structured? Semi-Structured?
- Strong schema? More flexibility? Reporting? Search? RDBMS / NoSQL?
- License costs? Switch to Cloud Native DB such as Aurora?

<!-- Page 515 -->

# Database Types

- RDBMS (= SQL / OLTP): RDS, Aurora – great for joins
- NoSQL database – no joins, no SQL : DynamoDB (~JSON), ElastiCache (key / value
pairs), Neptune (graphs), DocumentDB (for MongoDB), Keyspaces (for Apache
Cassandra)
- Object Store: S3 (for big objects) / Glacier (for backups / archives)
- Data Warehouse (= SQL Analytics / BI): Redshift (OLAP), Athena, EMR
- Search: OpenSearch (JSON) – free text, unstructured searches
- Graphs: Amazon Neptune – displays relationships between data
- Ledger: Amazon Quantum Ledger Database
- Time series: Amazon Timestream
- Note: some databases are being discussed in the Data & Analytics section

<!-- Page 516 -->

# Amazon RDS – Summary

- Managed PostgreSQL / MySQL / Oracle / SQL Server / DB2 / MariaDB / Custom
- Provisioned RDS Instance Size and EBS Volume Type & Size
- Auto-scaling capability for Storage
- Support for Read Replicas and Multi AZ
- Security through IAM, Security Groups, KMS , SSL in transit
- Automated Backup with Point in time restore feature (up to 35 days)
- Manual DB Snapshot for longer-term recovery
- Managed and Scheduled maintenance (with downtime)
- Support for IAM Authentication, integration with Secrets Manager
- RDS Custom for access to and customize the underlying instance (Oracle & SQL Server)
- Use case: Store relational datasets (RDBMS / OLTP), perform SQL queries, transactions

<!-- Page 517 -->

# Amazon Aurora – Summary

- Compatible API for PostgreSQL / MySQL, separation of storage and compute
- Storage: data is stored in 6 replicas, across 3 AZ – highly available, self-healing, auto-scaling
- Compute: Cluster of DB Instance across multiple AZ, auto-scaling of Read Replicas
- Cluster: Custom endpoints for writer and reader DB instances
- Same security / monitoring / maintenance features as RDS
- Know the backup & restore options for Aurora
- Aurora Serverless – for unpredictable / intermittent workloads, no capacity planning
- Aurora Global: up to 16 DB Read Instances in each region, < 1 second storage replication
- Aurora Machine Learning: perform ML using SageMaker & Comprehend on Aurora
- Aurora Database Cloning: new cluster from existing one, faster than restoring a snapshot
- Use case: same as RDS, but with less maintenance / more flexibility / more performance / more features

<!-- Page 518 -->

# Amazon ElastiCache – Summary

- Managed Redis / Memcached (similar offering as RDS, but for caches)
- In-memory data store, sub-millisecond latency
- Select an ElastiCache instance type (e.g., cache.m6g.large)
- Support for Clustering (Redis) and Multi AZ, Read Replicas (sharding)
- Security through IAM, Security Groups, KMS, Redis Auth
- Backup / Snapshot / Point in time restore feature
- Managed and Scheduled maintenance
- Requires some application code changes to be leveraged
- Use Case: Key/Value store, Frequent reads, less writes, cache results for DB
queries, store session data for websites, cannot use SQL.

<!-- Page 519 -->

# Amazon DynamoDB – Summary

- AWS proprietary technology, managed serverless NoSQL database, millisecond latency
- Capacity modes: provisioned capacity with optional auto-scaling or on-demand capacity
- Can replace ElastiCache as a key/value store (storing session data for example, using TTL feature)
- Highly Available, Multi AZ by default, Read and Writes are decoupled, transaction capability
- DAX cluster for read cache, microsecond read latency
- Security, authentication and authorization is done through IAM
- Event Processing: DynamoDB Streams to integrate with AWS Lambda, or Kinesis Data Streams
- Global Table feature: active-active setup
- Automated backups up to 35 days with PITR (restore to new table), or on-demand backups
- Export to S3 without using RCU within the PITR window, import from S3 without using WCU
- Great to rapidly evolve schemas
- Use Case: Serverless applications development (small documents 100s KB), distributed serverless
cache

<!-- Page 520 -->

# Amazon S3 – Summary

- S3 is a… key / value store for objects
- Great for bigger objects, not so great for many small objects
- Serverless, scales infinitely, max object size is 50 TB, versioning capability
- Tiers: S3 Standard, S3 Infrequent Access, S3 Intelligent, S3 Glacier + lifecycle policy
- Features: Versioning, Encryption, Replication, MFA-Delete, Access Logs…
- Security: IAM, Bucket Policies, ACL, Access Points, Object Lambda, CORS, Object/Vault Lock
- Encryption: SSE-S3, SSE-KMS, SSE-C, client-side, TLS in transit, default encryption
- Batch operations on objects using S3 Batch, listing files using S3 Inventory
- Performance: Multi-part upload, S3 Transfer Acceleration, S3 Select
- Automation: S3 Event Notifications (SNS, SQS, Lambda, EventBridge)
- Use Cases: static files, key value store for big files, website hosting

<!-- Page 521 -->

# DocumentDB

- Aurora is an “AWS-implementation” of PostgreSQL / MySQL …
- DocumentDB is the same for MongoDB (which is a NoSQL database)
- MongoDB is used to store, query, and index JSON data
- Similar “deployment concepts” as Aurora
- Fully Managed, highly available with replication across 3 AZ
- DocumentDB storage automatically grows in increments of 10GB
- Automatically scales to workloads with millions of requests per seconds

<!-- Page 522 -->

# Amazon Neptune

- Fully managed graph database
- A popular graph dataset would be a social network
- 
- 
- 
- 
Users have friends
Posts have comments
Comments have likes from users
Users share and like posts…
- Highly available across 3 AZ, with up to 15 read replicas
- Build and run applications working with highly connected
datasets – optimized for these complex and hard queries
- Can store up to billions of relations and query the graph with
milliseconds latency
- Highly available with replications across multiple AZs
- Great for knowledge graphs (Wikipedia), fraud detection,
recommendation engines, social networking

<!-- Page 523 -->

# Amazon Neptune – Streams

- Real-time ordered sequence of every change to
your graph data
- Changes are available immediately after writing
- No duplicates, strict order
- Streams data is accessible in an HTTP REST API
- Use cases:
Neptune Cluster
writes
Neptune
Streams
Streams API
HTTP Get Request
- Send notifications when certain changes are made
- Maintain your graph data synchronized in another
data store (e.g., S3, OpenSearch, ElastiCache)
- Replicate data across regions in Neptune
Streams reader application
…
…
S3
OpenSearch ElastiCache

<!-- Page 524 -->

# Amazon Keyspaces (for Apache Cassandra)

- Apache Cassandra is an open-source NoSQL distributed database
- A managed Apache Cassandra-compatible database service
- Serverless, Scalable, highly available, fully managed by AWS
- Automatically scale tables up/down based on the application’s traffic
- Tables are replicated 3 times across multiple AZ
- Using the Cassandra Query Language (CQL)
- Single-digit millisecond latency at any scale, 1000s of requests per second
- Capacity: On-demand mode or provisioned mode with auto-scaling
- Encryption, backup, Point-In-Time Recovery (PITR) up to 35 days
- Use cases: store IoT devices info, time-series data, …

<!-- Page 525 -->

# Amazon Timestream

- Fully managed, fast, scalable, serverless time series database
- Automatically scales up/down to adjust capacity
- Store and analyze trillions of events per day
- 1000s times faster & 1/10th the cost of relational databases
- Scheduled queries, multi-measure records, SQL compatibility
- Data storage tiering: recent data kept in memory and
historical data kept in a cost-optimized storage
- Built-in time series analytics functions (helps you identify
patterns in your data in near real-time)
- Encryption in transit and at rest
- Use cases: IoT apps, operational applications, real-time
analytics, …

<!-- Page 526 -->

# Amazon Timestream – Architecture

Amazon
QuickSight
AWS IoT
Lambda
Kinesis Data
Streams
Amazon
SageMaker
Prometheus
Amazon
Timestream
Kinesis Data
Streams
Any JDBC connection
Amazon MSK
Kinesis Data Analytics
For Apache Flink

<!-- Page 527 -->

# Data & Analytics

<!-- Page 528 -->

# Amazon Athena

- Serverless query service to analyze data stored in Amazon S3
- Uses standard SQL language to query the files (built on Presto)
- Supports CSV, JSON, ORC, Avro, and Parquet
- Pricing: $5.00 per TB of data scanned
- Commonly used with Amazon Quicksight for
reporting/dashboards
- Use cases: Business intelligence / analytics / reporting, analyze &
query VPC Flow Logs, ELB Logs, CloudTrail trails, etc...
- Exam Tip: analyze data in S3 using serverless SQL, use Athena
load data
S3 Bucket
Query & Analyze
Amazon
Athena
Reporting & Dashboards
Amazon
QuickSight

<!-- Page 529 -->

# Amazon Athena – Performance Improvement

- Use columnar data for cost-savings (less scan)
- Apache Parquet or ORC is recommended
- Huge performance improvement
- Use Glue to convert your data to Parquet or ORC
- Compress data for smaller retrievals (bzip2, gzip, lz4, snappy, zlip, zstd…)
- Partition datasets in S3 for easy querying on virtual columns
- s3://yourBucket/pathToTable
/<PARTITION_COLUMN_NAME>=<VALUE>
/<PARTITION_COLUMN_NAME>=<VALUE>
/<PARTITION_COLUMN_NAME>=<VALUE>
/etc…
- Example: s3://athena-examples/flight/parquet/year=1991/month=1/day=1/
- Use larger files (> 128 MB) to minimize overhead

<!-- Page 530 -->

# Amazon Athena – Federated Query

- Allows you to run SQL queries across
data stored in relational, non-relational,
object, and custom data sources (AWS
or on-premises)
- Uses Data Source Connectors that run
on AWS Lambda to run Federated
Queries (e.g., CloudWatch Logs,
DynamoDB, RDS, …)
- Store the results back in Amazon S3
Amazon
Athena
S3 Bucket
Lambda
ElastiCache
DocumentDB
HBase in EMR
DynamoDB
Redshift
(Data Source
Connector)
Database
(On-Premises)
Aurora
SQL Server
MySQL

<!-- Page 531 -->

# Redshift Overview

- Redshift is based on PostgreSQL, but it’s not used for OLTP
- It’s OLAP – online analytical processing (analytics and data warehousing)
- 10x better performance than other data warehouses, scale to PBs of data
- Columnar storage of data (instead of row based) & parallel query engine
- Two modes: Provisioned cluster or Serverless cluster
- Has a SQL interface for performing the queries
- BI tools such as Amazon Quicksight or Tableau integrate with it
- vs Athena: faster queries / joins / aggregations thanks to indexes

<!-- Page 532 -->

# Redshift Cluster

Query
SELECT COUNT (*), …
FROM MY_TABLE
GROUP BY …
JDBC/ODBC
Amazon Redshift Cluster
Leader Node
Compute Nodes
- Leader node: for query
planning, results
aggregation
- Compute node: for
performing the queries,
send results to leader
- Provisioned mode:
- Choose instance types
in advance
- Can reserve instances
for cost savings

<!-- Page 533 -->

# Redshift – Snapshots & DR

- Redshift has “Multi-AZ” mode for some clusters
- Snapshots are point-in-time backups of a cluster,
stored internally in S3
- Snapshots are incremental (only what has
changed is saved)
- You can restore a snapshot into a new cluster
- Automated: every 8 hours, every 5 GB, or on a
schedule. Set retention between 1 to 35 days
- Manual: snapshot is retained until you delete it
Region
(us-east-1)
Take Snapshot
Redshift Cluster
(Original)
Automated
/ Manual
Copy
Region
(eu-west-1)
- You can configure Amazon Redshift to
automatically copy snapshots (automated or
manual) of a cluster to another AWS Region
Restore
Redshift Cluster
(New)
Cluster
Snapshot
Copied
Snapshot

<!-- Page 534 -->

# Large inserts are MUCH better

Amazon Kinesis
Data Firehose
S3 using COPY command
EC2 Instance
JDBC driver
Internet
Without Enhanced VPC Routing
Through VPC
Amazon Kinesis
Data Firehose
Amazon Redshift
Cluster
(through S3 copy)
S3 Bucket
(mybucket)
With Enhanced VPC Routing
Amazon Redshift EC2 Instance
Amazon Redshift
Cluster
Cluster
Better to write
Data in batches
copy customer
from 's3://mybucket/mydata’
iam_role 'arn:aws:iam::0123456789012:role/MyRedshiftRole';
Loading data into Redshift:

<!-- Page 535 -->

# Redshift Spectrum

Query
SELECT COUNT (*), …
FROM S3.EXT_TABLE
GROUP BY …
JDBC/ODBC
- Query data that is already in
S3 without loading it
- Must have a Redshift cluster
available to start the query
Compute Nodes
- The query is then submitted
to thousands of Redshift
Spectrum nodes
Amazon S3
Amazon Redshift Cluster
Leader Node
1
2
….
N

<!-- Page 536 -->

# Amazon OpenSearch Service

- Amazon OpenSearch is successor to Amazon ElasticSearch
- In DynamoDB, queries only exist by primary key or indexes…
- With OpenSearch, you can search any field, even partially matches
- It’s common to use OpenSearch as a complement to another database
- Two modes: managed cluster or serverless cluster
- Does not natively support SQL (can be enabled via a plugin)
- Ingestion from Kinesis Data Firehose, AWS IoT, and CloudWatch Logs
- Security through Cognito & IAM, KMS encryption, TLS
- Comes with OpenSearch Dashboards (visualization)

<!-- Page 537 -->

# DynamoDB

CRUD
DynamoDB Table
DynamoDB Stream
API to retrieve items
Lambda Function
Amazon OpenSearch
API to search items
OpenSearch patterns

<!-- Page 538 -->

# CloudWatch Logs

Real time
Subscription Filter
Lambda Function
(managed by AWS)
Amazon OpenSearch
Near Real Time
Subscription Filter
Kinesis Data Firehose
Amazon OpenSearch
OpenSearch patterns

<!-- Page 539 -->

# Kinesis Data Streams & Kinesis Data Firehose

Kinesis Data
Streams
Kinesis Data
Streams
Lambda
Function
Kinesis Data
Firehose
(near real time)
Amazon
OpenSearch
Lambda
Function
(real time)
data
transformation
Amazon
OpenSearch
OpenSearch patterns

<!-- Page 540 -->

# Amazon EMR

- EMR stands for “Elastic MapReduce”
- EMR helps creating Hadoop clusters (Big Data) to analyze and process
vast amount of data
- The clusters can be made of hundreds of EC2 instances
- EMR comes bundled with Apache Spark, HBase, Presto, Flink…
- EMR takes care of all the provisioning and configuration
- Auto-scaling and integrated with Spot instances
- Use cases: data processing, machine learning, web indexing, big data…

<!-- Page 541 -->

# Amazon EMR – Node types & purchasing

- Master Node: Manage the cluster, coordinate, manage health – long running
- Core Node: Run tasks and store data – long running
- Task Node (optional): Just to run tasks – usually Spot
- Purchasing options:
- On-demand: reliable, predictable, won’t be terminated
- Reserved (min 1 year): cost savings (EMR will automatically use if available)
- Spot Instances: cheaper, can be terminated, less reliable
- Can have long-running cluster, or transient (temporary) cluster

<!-- Page 542 -->

# Amazon QuickSight

- Serverless machine learning-powered business intelligence service to create
interactive dashboards
- Fast, automatically scalable, embeddable, with per-session pricing
- Use cases:
- Business analytics
- Building visualizations
- Perform ad-hoc analysis
- Get business insights using data
- Integrated with RDS, Aurora,
Athena, Redshift, S3…
- In-memory computation using SPICE
engine if data is imported into QuickSight
- Enterprise edition: Possibility to setup
Column-Level security (CLS)
https://aws.amazon.com/quicksight/

<!-- Page 543 -->

# QuickSight Integrations

Data Sources (AWS Services)
QuickSight
RDS
Aurora
Redshift
On-Premises
Databases (JDBC)
Data Sources (Imports)
Athena
S3
OpenSearch
Data Sources (SaaS)
Timestream
ELF & CLF
(Log Format)

<!-- Page 544 -->

# QuickSight – Dashboard & Analysis

- Define Users (standard versions) and Groups (enterprise version)
- These users & groups only exist within QuickSight, not IAM !!
- A dashboard…
- is a read-only snapshot of an analysis that you can share
- preserves the configuration of the analysis (filtering, parameters, controls, sort)
- You can share the analysis or the dashboard with Users or Groups
- To share a dashboard, you must first publish it
- Users who see the dashboard can also see the underlying data

<!-- Page 545 -->

# AWS Glue

- Managed extract, transform, and load (ETL) service
- Useful to prepare and transform data for analytics
- Fully serverless service
Glue ETL
S3 Bucket
Load
Extract
Amazon RDS
Transform
Redshift
Data Warehouse

<!-- Page 546 -->

# AWS Glue – Convert data into Parquet format

Glue ETL
Import CSV
S3 Put
Input
S3 Bucket
Event notifications
On S3 PUT
Analyze
Parquet
Trigger
Glue ETL Job
Output
S3 Bucket
Lambda Function
(EventBridge works as an alternative)
Amazon
Athena

<!-- Page 547 -->

# Glue Data Catalog: catalog of datasets

Glue Jobs (ETL)
Amazon Athena
Data discovery
Amazon S3
Writes Metadata
Amazon RDS
AWS Glue
Data Crawler
AWS Glue
Data Catalog
Database
Database
Amazon DynamoDB
JDBC
Tables
Tables
(Metadata) (Metadata)
Amazon
Redshift
Spectrum
Amazon EMR

<!-- Page 548 -->

# Glue – things to know at a high-level

- Glue Job Bookmarks: prevent re-processing old data
- Glue DataBrew: clean and normalize data using pre-built transformation
- Glue Studio: new GUI to create, run and monitor ETL jobs in Glue
- Glue Streaming ETL (built on Apache Spark Structured Streaming):
compatible with Kinesis Data Streaming, Kafka, MSK (managed Kafka)

<!-- Page 549 -->

# AWS Lake Formation

- Data lake = central place to have all your data for analytics purposes
- Fully managed service that makes it easy to setup a data lake in days
- Discover, cleanse, transform, and ingest data into your Data Lake
- It automates many complex manual steps (collecting, cleansing, moving,
cataloging data, …) and de-duplicate (using ML Transforms)
- Combine structured and unstructured data in the data lake
- Out-of-the-box source blueprints: S3, RDS, Relational & NoSQL DB…
- Fine-grained Access Control for your applications (row and column-level)
- Built on top of AWS Glue

<!-- Page 550 -->

# AWS Lake Formation

Data Sources
Athena
Source Crawlers
Amazon S3
ETL and Data Prep.
ingest
RDS
Aurora
Data Catalog
Data Lake
(stored in S3)
Users
Security Settings
Access Control
On-Premises
Database (SQL & NoSQL)
Redshift
EMR

<!-- Page 551 -->

# Centralized Permissions Example

Data Sources
Amazon S3
ingest
Access Control
Column-level security
AWS Lake Formation
RDS
Aurora
Data Lake
(stored in S3)
Athena
Quicksight
Users
AWS Lake Formation

<!-- Page 552 -->

# Amazon Managed Service for Apache Flink

- Previously named: Kinesis Data Analytics for Apache Flink
- Flink (Java, Scala or SQL) is a framework for processing data streams
Kinesis Data
Streams
Amazon MSK
(Apache Kafka)
Amazon Managed Service
for Apache Flink
- Run any Apache Flink application on a managed cluster on AWS
- Provisioned compute resources, parallel computation, automatic scaling
- Application backups (implemented as checkpoints and snapshots)
- Use any Apache Flink programming features to transform data
- Important: Flink does not read from Amazon Data Firehose

<!-- Page 553 -->

# Kafka (Amazon MSK)

- Alternative to Amazon Kinesis
- Fully managed Apache Kafka on AWS
- Allow you to create, update, delete clusters
- MSK creates & manages Kafka brokers nodes & Zookeeper nodes for you
- Deploy the MSK cluster in your VPC, multi-AZ (up to 3 for HA)
- Automatic recovery from common Apache Kafka failures
- Data is stored on EBS volumes for as long as you want
- MSK Serverless
- Run Apache Kafka on MSK without managing the capacity
- MSK automatically provisions resources and scales compute & storage
Amazon Managed Streaming for Apache

<!-- Page 554 -->

# Apache Kafka at a high level

EMR
MSK Cluster
S3
rep
lica
ti
on
Kinesis
IoT
Producers
(your code)
Poll from topic
Write to topic
Broker 1
SageMaker
Consumers
(your code)
ti
lica
rep
RDS
Broker 2
Kinesis
on
Etc…
RDS
Broker 3
Etc…

<!-- Page 555 -->

# Kinesis Data Streams vs. Amazon MSK

Kinesis Data Streams
- 1 MB message size limit
- Data Streams with Shards
- Shard Splitting & Merging
- TLS In-flight encryption
- KMS at-rest encryption
Amazon MSK
- 1MB default, configure for higher (ex: 10MB)
- Kafka Topics with Partitions
- Can only add partitions to a topic
- PLAINTEXT or TLS In-flight Encryption
- KMS at-rest encryption

<!-- Page 556 -->

# Amazon MSK Consumers

Kinesis Data Analytics
for Apache Flink
AWS Glue
Streaming ETL Jobs
Powered by Apache Spark Streaming
Amazon MSK
Lambda
Applications Running on
Amazon EC2
ECS
EKS

<!-- Page 557 -->

# Big Data Ingestion Pipeline

- We want the ingestion pipeline to be fully serverless
- We want to collect data in real time
- We want to transform the data
- We want to query the transformed data using SQL
- The reports created using the queries should be in S3
- We want to load that data into a warehouse and create dashboards

<!-- Page 558 -->

# Serverless

Pull data
IoT Devices
Real-time
Every 1 minute
Reporting
Bucket
Ingestion
Bucket
trigger
Amazon Kinesis Data
Streams
Amazon Kinesis Data Amazon Simple Storage Amazon Simple Queue
Firehose
Service (S3)
Service
AWS Lambda
Amazon Athena
Amazon Simple Storage
Service (S3)
(optional)
AWS Lambda
Amazon QuickSight
Big Data Ingestion Pipeline
Amazon Redshift

<!-- Page 559 -->

# Big Data Ingestion Pipeline discussion

- IoT Core allows you to harvest data from IoT devices
- Kinesis is great for real-time data collection
- Firehose helps with data delivery to S3 in near real-time (1 minute)
- Lambda can help Firehose with data transformations
- Amazon S3 can trigger notifications to SQS
- Lambda can subscribe to SQS (we could have connecter S3 to Lambda)
- Athena is a serverless SQL service and results are stored in S3
- The reporting bucket contains analyzed data and can be used by
reporting tool such as AWS QuickSight, Redshift, etc…

<!-- Page 560 -->

# Machine Learning

<!-- Page 561 -->

# Amazon Rekognition

- Find objects, people, text, scenes in images and videos using ML
- Facial analysis and facial search to do user verification, people counting
- Create a database of “familiar faces” or compare against celebrities
- Use cases:
- Labeling
- Content Moderation
- Text Detection
- Face Detection and Analysis (gender, age range, emotions…)
- Face Search and Verification
- Celebrity Recognition
- Pathing (ex: for sports game analysis)

<!-- Page 562 -->

# Amazon Rekognition – Content Moderation

- Detect content that is inappropriate, unwanted,
or offensive (image and videos)
- Used in social media, broadcast media,
advertising, and e-commerce situations to create
a safer user experience
- Set a Minimum Confidence Threshold for items
that will be flagged
- Flag sensitive content for manual review in
Amazon Augmented AI (A2I)
- Help comply with regulations
Image
Amazon
Rekognition
Confidence Level
and Threshold
Optional Manual
review in A2I

<!-- Page 563 -->

# Amazon Transcribe

- Automatically convert speech to text
- Uses a deep learning process called automatic speech recognition (ASR) to convert
speech to text quickly and accurately
- Automatically remove Personally Identifiable Information (PII) using Redaction
- Supports Automatic Language Identification for multi-lingual audio
- Use cases:
- transcribe customer service calls
- automate closed captioning and subtitling
- generate metadata for media assets to create a fully searchable archive
”Hello my name is Stéphane.
I hope you’re enjoying the course!

<!-- Page 564 -->

# Amazon Polly

- Turn text into lifelike speech using deep learning
- Allowing you to create applications that talk
Hi! My name is Stéphane
and this is a demo of Amazon Polly

<!-- Page 565 -->

# Amazon Polly – Lexicon & SSML

- Customize the pronunciation of words with Pronunciation lexicons
- Stylized words: St3ph4ne => “Stephane”
- Acronyms: AWS => “Amazon Web Services”
- Upload the lexicons and use them in the SynthesizeSpeech operation
- Generate speech from plain text or from documents marked up with Speech
Synthesis Markup Language (SSML) – enables more customization
- emphasizing specific words or phrases
- using phonetic pronunciation
- including breathing sounds, whispering
- using the Newscaster speaking style

<!-- Page 566 -->

# Amazon Translate

- Natural and accurate language translation
- Amazon Translate allows you to localize content - such as websites and
applications - for international users, and to easily translate large volumes
of text efficiently.

<!-- Page 567 -->

# Amazon Lex & Connect

- Amazon Lex: (same technology that powers Alexa)
- Automatic Speech Recognition (ASR) to convert speech to text
- Natural Language Understanding to recognize the intent of text, callers
- Helps build chatbots, call center bots
- Amazon Connect:
- Receive calls, create contact flows, cloud-based virtual contact center
- Can integrate with other CRM systems or AWS
- No upfront payments, 80% cheaper than traditional contact center solutions
Phone Call
Schedule an
Appointment
call
stream
Connect
schedule
invoke
Lex
Intent recognized
Lambda
CRM

<!-- Page 568 -->

# Amazon Comprehend

- For Natural Language Processing – NLP
- Fully managed and serverless service
- Uses machine learning to find insights and relationships in text
- Language of the text
- Extracts key phrases, places, people, brands, or events
- Understands how positive or negative the text is
- Analyzes text using tokenization and parts of speech
- Automatically organizes a collection of text files by topic
- Sample use cases:
- analyze customer interactions (emails) to find what leads to a positive or negative experience
- Create and groups articles by topics that Comprehend will uncover

<!-- Page 569 -->

# Amazon Comprehend Medical

- Amazon Comprehend Medical detects and returns useful information in
unstructured clinical text:
- Physician’s notes
- Discharge summaries
- Test results
- Case notes
- Uses NLP to detect Protected Health Information (PHI) – DetectPHI API
- Store your documents in Amazon S3, analyze real-time data with Kinesis
Data Firehose, or use Amazon Transcribe to transcribe patient narratives
into text that can be analyzed by Amazon Comprehend Medical.

<!-- Page 570 -->

# Amazon SageMaker AI

- Fully managed service for developers / data scientists to build ML models
- Typically, difficult to do all the processes in one place + provision servers
- Machine learning process (simplified): predicting your exam score
label
670
build
Train and Tune
890
934
Historical Data:
# years of experience in IT
# years of experience with AWS
Time spent on the course
…
ML model
score
Apply model
New data
Prediction
PASS WITH 906

<!-- Page 571 -->

# Amazon Kendra

- Fully managed document search service powered by Machine Learning
- Extract answers from within a document (text, pdf, HTML, PowerPoint, MS Word, FAQs…)
- Natural language search capabilities
- Learn from user interactions/feedback to promote preferred results (Incremental Learning)
- Ability to manually fine-tune search results (importance of data, freshness, custom, …)
Data Sources
Where is the IT support desk?
Amazon S3
Amazon RDS Google Drive MS SharePoint
3rd party,
MS OneDrive
APNs,
Custom
indexing
Knowledge Index
(powered by ML)
1st floor
User

<!-- Page 572 -->

# Amazon Personalize

- Fully managed ML-service to build apps with real-time personalized recommendations
- Example: personalized product recommendations/re-ranking, customized direct marketing
- Example: User bought gardening tools, provide recommendations on the next one to buy
- Same technology used by Amazon.com
- Integrates into existing websites, applications, SMS, email marketing systems, …
- Implement in days, not months (you don’t need to build, train, and deploy ML solutions)
- Use cases: retail stores, media and entertainment…
Websites & Apps
Amazon S3
Amazon Personalize API
read data from S3
Customized personalized API
Mobile Apps
real-time data integration
SMS
Emails

<!-- Page 573 -->

# Amazon Textract

- Automatically extracts text, handwriting, and data from any scanned
documents using AI and ML
{
analyze
result
}
“Document ID”: “123456789-005”,
“Name”: “”,
“SEX”: “F”,
“DOB”: “23.05.1997”,
…
- Extract data from forms and tables
- Read and process any type of document (PDFs, images, …)
- Use cases:
- Financial Services (e.g., invoices, financial reports)
- Healthcare (e.g., medical records, insurance claims)
- Public Sector (e.g., tax forms, ID documents, passports)

<!-- Page 574 -->

# AWS Machine Learning - Summary

- Rekognition: face detection, labeling, celebrity recognition
- Transcribe: audio to text (ex: subtitles)
- Polly: text to audio
- Translate: translations
- Lex: build conversational bots – chatbots
- Connect: cloud contact center
- Comprehend: natural language processing
- SageMaker: machine learning for every developer and data scientist
- Kendra: ML-powered search engine
- Personalize: real-time personalized recommendations
- Textract: detect text and data in documents

<!-- Page 575 -->

# Performance

CloudWatch, CloudTrail & AWS Config
AWS Monitoring, Audit and

<!-- Page 576 -->

# Amazon CloudWatch Metrics

- CloudWatch provides metrics for every service in AWS
- Metric is a variable to monitor (CPUUtilization, NetworkIn…)
- Metrics belong to namespaces
- Dimension is an attribute of a metric (instance id, environment, etc…).
- Up to 30 dimensions per metric
- Metrics have timestamps
- Can create CloudWatch dashboards of metrics
- Can create CloudWatch Custom Metrics (for the RAM for example)

<!-- Page 577 -->

# CloudWatch Metric Streams

CloudWatch Metrics
- Continually stream CloudWatch
metrics to a destination of your choice,
with near-real-time delivery and low
latency.
Stream near-real-time
- Amazon Kinesis Data Firehose (and then
its destinations)
- 3rd party service provider: Datadog,
Dynatrace, New Relic, Splunk, Sumo
Logic…
- Option to filter metrics to only stream
a subset of them
Kinesis Data Firehose
Amazon S3
Amazon
Redshift
Athena
Amazon
OpenSearch

<!-- Page 578 -->

# CloudWatch Logs

- Log groups: arbitrary name, usually representing an application
- Log stream: instances within application / log files / containers
- Can define log expiration policies (never expire, 1 day to 10 years…)
- CloudWatch Logs can send logs to:
- Amazon S3 (exports)
- Kinesis Data Streams
- Kinesis Data Firehose
- AWS Lambda
- OpenSearch
- Logs are encrypted by default
- Can setup KMS-based encryption with your own keys

<!-- Page 579 -->

# CloudWatch Logs - Sources

- SDK, CloudWatch Logs Agent, CloudWatch Unified Agent
- Elastic Beanstalk: collection of logs from application
- ECS: collection from containers
- AWS Lambda: collection from function logs
- VPC Flow Logs: VPC specific logs
- API Gateway
- CloudTrail based on filter
- Route53: Log DNS queries

<!-- Page 580 -->

# CloudWatch Logs Insights

https://mng.workshop.aws/operations-2022/detect/cwlogs.html

<!-- Page 581 -->

# CloudWatch Logs Insights

- Search and analyze log data stored in CloudWatch Logs
- Example: find a specific IP inside a log, count occurrences of
“ERROR” in your logs…
- Provides a purpose-built query language
- Automatically discovers fields from AWS services and JSON log
events
- Fetch desired event fields, filter based on conditions, calculate
aggregate statistics, sort events, limit number of events…
- Can save queries and add them to CloudWatch Dashboards
- Can query multiple Log Groups in different AWS accounts
- It’s a query engine, not a real-time engine

<!-- Page 582 -->

# CloudWatch Logs – S3 Export

- Log data can take up to 12 hours to
become available for export
- The API call is CreateExportTask
CloudWatch Logs
Amazon S3
- Not near-real time or real-time… use
Logs Subscriptions instead

<!-- Page 583 -->

# CloudWatch Logs Subscriptions

- Get a real-time log events from CloudWatch Logs for processing and analysis
- Send to Kinesis Data Streams, Kinesis Data Firehose, or Lambda
- Subscription Filter – filter which logs are events delivered to your destination
OpenSearch
Service
real-time
Lambda
logs
CloudWatch Logs
Subscription Filter
near
real-time
S3
Kinesis Data Firehose
…
Kinesis Data Streams
KDF
KDA
EC2 Lambda

<!-- Page 584 -->

# Multi-Account & Multi Region

ACCOUNT A
REGION 1
CloudWatch Logs
Subscription Filter
ACCOUNT B
REGION 2
CloudWatch Logs
Near
Real Time
Subscription Filter
ACCOUNT B
REGION 3
CloudWatch Logs
Subscription Filter
Kinesis Data Streams Kinesis Data Firehose
Amazon S3
CloudWatch Logs Aggregation

<!-- Page 585 -->

# CloudWatch Logs Subscriptions

- Cross-Account Subscription – send log events to resources in a different AWS
account (KDS, KDF)
IAM Role
(Cross-Account)
Account – Sender
Account – Recipient
(111111111111)
logs
CloudWatch
Logs
logs
Subscription
Filter
Can be assumed
(999999999999)
Subscription
Destination
Kinesis Data Streams
(RecipientStream)
Destination
Access Policy
Destination
Access Policy
IAM Role
allow PutRecord

<!-- Page 586 -->

# CloudWatch Logs for EC2

- By default, no logs from your EC2
machine will go to CloudWatch
- You need to run a CloudWatch
agent on EC2 to push the log files
you want
- Make sure IAM permissions are
correct
- The CloudWatch log agent can be
setup on-premises too
CloudWatch Logs
CloudWatch
Logs Agent
CloudWatch
Logs Agent
EC2 Instance
On Premise
Server

<!-- Page 587 -->

# CloudWatch Logs Agent & Unified Agent

- For virtual servers (EC2 instances, on-premises servers…)
- CloudWatch Logs Agent
- Old version of the agent
- Can only send to CloudWatch Logs
- CloudWatch Unified Agent
- Collect additional system-level metrics such as RAM, processes, etc…
- Collect logs to send to CloudWatch Logs
- Centralized configuration using SSM Parameter Store

<!-- Page 588 -->

# CloudWatch Unified Agent – Metrics

- Collected directly on your Linux server / EC2 instance
- CPU (active, guest, idle, system, user, steal)
- Disk metrics (free, used, total), Disk IO (writes, reads, bytes, iops)
- RAM (free, inactive, used, total, cached)
- Netstat (number of TCP and UDP connections, net packets, bytes)
- Processes (total, dead, bloqued, idle, running, sleep)
- Swap Space (free, used, used %)
- Reminder: out-of-the box metrics for EC2 – disk, CPU, network (high level)

<!-- Page 589 -->

# CloudWatch Alarms

- Alarms are used to trigger notifications for any metric
- Various options (sampling, %, max, min, etc…)
- Alarm States:
- OK
- INSUFFICIENT_DATA
- ALARM
- Period:
- Length of time in seconds to evaluate the metric
- High resolution custom metrics: 10 sec, 30 sec or multiples of 60 sec

<!-- Page 590 -->

# CloudWatch Alarm Targets

- Stop, Terminate, Reboot, or Recover an EC2 Instance
- Trigger Auto Scaling Action
- Send notification to SNS (from which you can do pretty much anything)
Amazon EC2
EC2 Auto Scaling
Amazon SNS

<!-- Page 591 -->

# CloudWatch Alarms – Composite Alarms

- CloudWatch Alarms are on a single metric
- Composite Alarms are monitoring the states of multiple other alarms
- AND and OR conditions
- Helpful to reduce “alarm noise” by creating complex composite alarms
Composite Alarm
ALARM
monitor CPU
CW Alarm - A
EC2 Instance
monitor IOPS
ALARM
CW Alarm - B
trigger
Amazon SNS

<!-- Page 592 -->

# EC2 Instance Recovery

- Status Check:
- Instance status = check the EC2 VM
- System status = check the underlying hardware
- Attached EBS status = check attached EBS volumes
monitor
EC2 Instance
alert
CloudWatch Alarm
StatusCheckFailed_System
SNS Topic
- Recovery: Same Private, Public, Elastic IP, metadata, placement group

<!-- Page 593 -->

# CloudWatch Alarm: good to know

- Alarms can be created based on CloudWatch Logs Metrics Filters
CloudWatch
Metric Filter
Alert
CW Logs
CW Alarm
Amazon SNS
- To test alarms and notifications, set the alarm state to Alarm using CLI
aws cloudwatch set-alarm-state --alarm-name "myalarm" --state-value
ALARM --state-reason "testing purposes"

<!-- Page 594 -->

# CloudWatch Network Synthetic Monitor

- Monitor and detects network issues between
your apps hosted on AWS and your onpremises data center
- Identify any network performance
degradation (e.g., packet loss, latency, jitter…)
- No agents required to be installed
- Tests ICMP or TCP traffic to IPv4/IPv4 onpremises destinations through Direct
Connect or S2S VPN connections
- Publishes data to CloudWatch Metrics
AWS Cloud
Private Subnet
EC2 instance
CloudWatch
Metrics
DX Connection
or
VPN Connection
Corporate Data Center
Server

<!-- Page 595 -->

# (formerly CloudWatch Events)

- Schedule: Cron jobs (scheduled scripts)
Schedule Every hour
Trigger script on Lambda function
- Event Pattern: Event rules to react to a service doing something
IAM Root User Sign in Event
SNS Topic with Email Notification
- Trigger Lambda functions, send SQS/SNS messages…
Amazon EventBridge

<!-- Page 596 -->

# Amazon EventBridge Rules

{
EC2 Instance
(ex: Start Instance)
CodeBuild
(ex: failed build)
Filter events
(optional)
S3 Event
(ex: upload object)
CloudTrail
(any API call)
Trusted Advisor
(ex: new Finding)
Schedule or Cron
(ex: every 4 hours)
"version": "0",
"id": "6a7e8feb-b491",
"detail-type": "EC2 Instance
State-change Notification",
….
}
Amazon
EventBridge
Integration
JSON
Maintenance Orchestration
Example Source
Compute
Example Destinations
Lambda
AWS Batch
ECS Task
SQS
SNS
Kinesis Data
Streams
Step
Functions
CodePipeline
CodeBuild
SSM
EC2 Actions

<!-- Page 597 -->

# Amazon EventBridge

AWS Services
Default
Event Bus
AWS SaaS
Partners
Partner
Event Bus
Custom
Apps
Custom
Event Bus
- Event buses can be accessed by other AWS accounts using Resource-based Policies
- You can archive events (all/filter) sent to an event bus (indefinitely or set period)
- Ability to replay archived events

<!-- Page 598 -->

# Amazon EventBridge – Schema Registry

- EventBridge can analyze the events in
your bus and infer the schema
- The Schema Registry allows you to
generate code for your application, that
will know in advance how data is
structured in the event bus
- Schema can be versioned

<!-- Page 599 -->

# Amazon EventBridge – Resource-based Policy

- Manage permissions for a specific Event Bus
- Example: allow/deny events from another AWS account or AWS region
- Use case: aggregate all events from your AWS Organization in a single AWS
account or AWS region
AWS Account
(123456789012)
AWS Account
(111122223333)
PutEvents
EventBridge Bus
(central-event-bus)
Allow events from another AWS account
Lambda function

<!-- Page 600 -->

# CloudWatch Container Insights

- Collect, aggregate, summarize metrics and logs
from containers
- Available for containers on…
- Amazon Elastic Container Service (Amazon ECS)
- Amazon Elastic Kubernetes Services (Amazon EKS)
- Kubernetes platforms on EC2
- Fargate (both for ECS and EKS)
- In Amazon EKS and Kubernetes, CloudWatch
Insights is using a containerized version of the
CloudWatch Agent to discover containers
ECS Container
EKS Container
Metrics and logs
CloudWatch
Container Insights

<!-- Page 601 -->

# CloudWatch Lambda Insights

- Monitoring and troubleshooting
solution for serverless applications
running on AWS Lambda
- Collects, aggregates, and summarizes
system-level metrics including CPU
time, memory, disk, and network
- Collects, aggregates, and summarizes
diagnostic information such as cold
starts and Lambda worker shutdowns
- Lambda Insights is provided as a
Lambda Layer

<!-- Page 602 -->

# CloudWatch Contributor Insights

- Analyze log data and create time series that display
contributor data.
- See metrics about the top-N contributors
- The total number of unique contributors, and their usage.
- This helps you find top talkers and understand who or
what is impacting system performance.
- Works for any AWS-generated logs (VPC, DNS, etc..)
- For example, you can find bad hosts, identify the
heaviest network users, or find the URLs that generate
the most errors.
- You can build your rules from scratch, or you can also
use sample rules that AWS has created – leverages
your CloudWatch Logs
- CloudWatch also provides built-in rules that you can
use to analyze metrics from other AWS services.
VPC Flow Logs
CloudWatch Logs
CloudWatch
Contributor Insights
Top-10 IP addresses

<!-- Page 603 -->

# CloudWatch Application Insights

- Provides automated dashboards that show potential problems with monitored
applications, to help isolate ongoing issues
- Your applications run on Amazon EC2 Instances with select technologies only
(Java, .NET, Microsoft IIS Web Server, databases…)
- And you can use other AWS resources such as Amazon EBS, RDS, ELB, ASG,
Lambda, SQS, DynamoDB, S3 bucket, ECS, EKS, SNS, API Gateway…
- Powered by SageMaker
- Enhanced visibility into your application health to reduce the time it will take
you to troubleshoot and repair your applications
- Findings and alerts are sent to Amazon EventBridge and SSM OpsCenter

<!-- Page 604 -->

# CloudWatch Insights and Operational Visibility

- CloudWatch Container Insights
- ECS, EKS, Kubernetes on EC2, Fargate, needs agent for Kubernetes
- Metrics and logs
- CloudWatch Lambda Insights
- Detailed metrics to troubleshoot serverless applications
- CloudWatch Contributors Insights
- Find “Top-N” Contributors through CloudWatch Logs
- CloudWatch Application Insights
- Automatic dashboard to troubleshoot your application and related AWS services

<!-- Page 605 -->

# AWS CloudTrail

- Provides governance, compliance and audit for your AWS Account
- CloudTrail is enabled by default!
- Get an history of events / API calls made within your AWS Account by:
- Console
- SDK
- CLI
- AWS Services
- Can put logs from CloudTrail into CloudWatch Logs or S3
- A trail can be applied to All Regions (default) or a single Region.
- If a resource is deleted in AWS, investigate CloudTrail first!

<!-- Page 606 -->

# CloudTrail Diagram

SDK
CloudTrail Console
CloudWatch Logs
CLI
Console
IAM Users &
IAM Roles
Inspect & Audit
S3 Bucket

<!-- Page 607 -->

# CloudTrail Events

- Management Events:
- Operations that are performed on resources in your AWS account
- Examples:
- Configuring security (IAM AttachRolePolicy)
- Configuring rules for routing data (Amazon EC2 CreateSubnet)
- Setting up logging (AWS CloudTrail CreateTrail)
- By default, trails are configured to log management events.
- Can separate Read Events (that don’t modify resources) from Write Events (that may modify resources)
- Data Events:
- By default, data events are not logged (because high volume operations)
- Amazon S3 object-level activity (ex: GetObject, DeleteObject, PutObject): can separate Read and Write Events
- AWS Lambda function execution activity (the Invoke API)
- CloudTrail Insights Events:
- See next slide J

<!-- Page 608 -->

# CloudTrail Insights

- Enable CloudTrail Insights to detect unusual activity in your account:
- inaccurate resource provisioning
- hitting service limits
- Bursts of AWS IAM actions
- Gaps in periodic maintenance activity
- CloudTrail Insights analyzes normal management events to create a baseline
- And then continuously analyzes write events to detect unusual patterns
- Anomalies appear in the CloudTrail console
- Event is sent to Amazon S3
- An EventBridge event is generated (for automation needs)
Management Events
Continous analysis
generate
Insights Events
CloudTrail Console
S3 Bucket
EventBridge event

<!-- Page 609 -->

# CloudTrail Events Retention

- Events are stored for 90 days in CloudTrail
- To keep events beyond this period, log them to S3 and use Athena
Management Events
CloudTrail
log
Data Events
Insights Events
Athena
90 days
retention
analyze
S3 Bucket
Long-term retention

<!-- Page 610 -->

# Amazon EventBridge – Intercept API Calls

User
DeleteTable API Call 💥
Log API call
DynamoDB
alert
event
CloudTrail
(any API call)
Amazon
EventBridge
SNS

<!-- Page 611 -->

# Amazon EventBridge + CloudTrail

API Call logs
User
IAM
AssumeRole
event
CloudTrail
AuthorizeSecurityGroupIngress
SNS
EventBridge
SNS
IAM Role
API Call logs
User
EventBridge
edit SG
Inbound Rules
EC2
Security Group
event
CloudTrail

<!-- Page 612 -->

# AWS Config

- Helps with auditing and recording compliance of your AWS resources
- Helps record configurations and changes over time
- Questions that can be solved by AWS Config:
- Is there unrestricted SSH access to my security groups?
- Do my buckets have any public access?
- How has my ALB configuration changed over time?
- You can receive alerts (SNS notifications) for any changes
- AWS Config is a per-region service
- Can be aggregated across regions and accounts
- Possibility of storing the configuration data into S3 (analyzed by Athena)

<!-- Page 613 -->

# Config Rules

- Can use AWS managed config rules (over 75)
- Can make custom config rules (must be defined in AWS Lambda)
- Ex: evaluate if each EBS disk is of type gp2
- Ex: evaluate if each EC2 instance is t2.micro
- Rules can be evaluated / triggered:
- For each config change
- And / or: at regular time intervals
- AWS Config Rules does not prevent actions from happening (no deny)
- Pricing: no free tier, $0.003 per configuration item recorded per region,
$0.001 per config rule evaluation per region

<!-- Page 614 -->

# AWS Config Resource

- View compliance of a resource over time
- View configuration of a resource over time
- View CloudTrail API calls of a resource over time

<!-- Page 615 -->

# Config Rules – Remediations

- Automate remediation of non-compliant resources using SSM Automation
Documents
- Use AWS-Managed Automation Documents or create custom Automation
Documents
- Tip: you can create custom Automation Documents that invokes Lambda function
- You can set Remediation Retries if the resource is still non-compliant after autoremediation
expired
IAM Access Key
(NON_COMPLIANT)
monitor
trigger
AWS Config
deactivate
Auto-Remediation Action
(SSM Document: AWSConfigRemediationRevokeUnusedIAMUserCredentials)
Retries: 5

<!-- Page 616 -->

# Config Rules – Notifications

- Use EventBridge to trigger notifications when AWS resources are noncompliant
monitor
AWS Resources
Security group
…
trigger
…
NON_COMPLIANT
Lambda
AWS Config
EventBridge
SNS
SQS
- Ability to send configuration changes and compliance state notifications
to SNS (all events – use SNS Filtering or filter at client-side)
monitor
AWS Resources
Security group
…
notification
trigger
All events
(configuration changes,
compliance state…)
AWS Config
SNS
Admin

<!-- Page 617 -->

# CloudWatch vs CloudTrail vs Config

- CloudWatch
- Performance monitoring (metrics, CPU, network, etc…) & dashboards
- Events & Alerting
- Log Aggregation & Analysis
- CloudTrail
- Record API calls made within your Account by everyone
- Can define trails for specific resources
- Global Service
- Config
- Record configuration changes
- Evaluate resources against compliance rules
- Get timeline of changes and compliance

<!-- Page 618 -->

# For an Elastic Load Balancer

- CloudWatch:
- Monitoring Incoming connections metric
- Visualize error codes as % over time
- Make a dashboard to get an idea of your load balancer performance
- Config:
- Track security group rules for the Load Balancer
- Track configuration changes for the Load Balancer
- Ensure an SSL certificate is always assigned to the Load Balancer (compliance)
- CloudTrail:
- Track who made any changes to the Load Balancer with API calls

<!-- Page 619 -->

# Advanced Identity in AWS

<!-- Page 620 -->

# AWS Organizations

- Global service
- Allows to manage multiple AWS accounts
- The main account is the management account
- Other accounts are member accounts
- Member accounts can only be part of one organization
- Consolidated Billing across all accounts - single payment method
- Pricing benefits from aggregated usage (volume discount for EC2, S3…)
- Shared reserved instances and Savings Plans discounts across accounts
- API is available to automate AWS account creation

<!-- Page 621 -->

# AWS Organizations

Root Organizational Unit (OU)
Management Account
OU (Dev)
OU (Prod)
OU (HR)
Member Accounts
OU (Finance)

<!-- Page 622 -->

# Organizational Units (OU) - Examples

Business Unit
Sales OU
Management
Account
Retail OU
Sales
Account 1
Prod OU
Sales
Account 2
Retail
Account 1
Retail
Account 2
Finance
OU
Environmental Lifecycle
Finance
Account 1
Finance
Account 2
Management
Account
Dev OU
Prod
Account 1
Project 1
OU
Prod
Account 2
Dev
Account 1
Dev
Account 2
Test OU
Project-Based
Test
Account 1
Test
Account 2
Management
Account
Project 2
OU
Project 3
OU
Project 1
Account 1
Project 1
Account 2
Project 2
Account 1
Project 2
Account 2
Project 3
Account 1
Project 3
Account 2

<!-- Page 623 -->

# AWS Organizations

- Advantages
- Multi Account vs One Account Multi VPC
- Use tagging standards for billing purposes
- Enable CloudTrail on all accounts, send logs to central S3 account
- Send CloudWatch Logs to central logging account
- Establish Cross Account Roles for Admin purposes
- Security: Service Control Policies (SCP)
- IAM policies applied to OU or Accounts to restrict Users and Roles
- They do not apply to the management account (full admin power)
- Must have an explicit allow from the root through each OU in the direct path
to the target account (does not allow anything by default – like IAM)

<!-- Page 624 -->

# SCP Hierarchy

OU (Root)
FullAWSAccess
- Management Account
- Can do anything (no SCP apply)
Deny Athena
- Account A
Management Account
FullAWSAccess
FullAWSAccess + Deny S3
Allow EC2
FullAWSAccess + Deny EC2
OU (Sandbox)
OU (Workloads)
- Account B & C
Account A
OU (Test)
Account D
Account B
FullAWSAccess
Account C
OU (Prod)
Account E
Account F
- Can do anything
- EXCEPT S3 (explicit Deny from
Sandbox OU)
- EXCEPT EC2 (explicit Deny)
- Can do anything
- EXCEPT S3 (explicit Deny from
Sandbox OU)
- Account D
- Can access EC2
- Prod OU & Account E & F
- Can do anything

<!-- Page 625 -->

# Blocklist and Allowlist strategies

More examples: https://docs.aws.amazon.com/organizations/latest/userguide/orgs_manage_policies_example-scps.html
SCP Examples

<!-- Page 626 -->

# AWS Organizations – Tag Policies

- Helps you standardize tags across resources in an
AWS Organization
- Ensure consistent tags, audit tagged resources,
maintain proper resources categorization, …
- You define tag keys and their allowed values
- Helps with AWS Cost Allocation Tags and
Attribute-based Access Control
- Prevent any non-compliant tagging operations on
specified services and resources (has no effect
on resources without tags)
- Generate a report that lists all tagged/noncompliant resources
- Use EventBridge to monitor non-compliant tags

<!-- Page 627 -->

# IAM Conditions

aws:SourceIp
restrict the client IP from
which the API calls are being made
aws:RequestedRegion
restrict the region the
API calls are made to

<!-- Page 628 -->

# IAM Conditions

ec2:ResourceTag
restrict based on tags
aws:MultiFactorAuthPresent
to force MFA

<!-- Page 629 -->

# IAM for S3

- s3:ListBucket permission applies to
arn:aws:s3:::test
- => bucket level permission
- s3:GetObject, s3:PutObject,
s3:DeleteObject applies to
arn:awn:s3:::test/*
- => object level permission

<!-- Page 630 -->

# Resource Policies & aws:PrincipalOrgID

- aws:PrincipalOrgID can be used in any resource policies to restrict
access to accounts that are member of an AWS Organization
AWS Organization
(o-yyyyyyyyyy)
…
Member Accounts
S3 Bucket
(2022-financial-data)
User outside Organization

<!-- Page 631 -->

# IAM Roles vs Resource Based Policies

- Cross account:
- attaching a resource-based policy to a resource (example: S3 bucket policy)
- OR using a role as a proxy
User
Account A
Role
Account B
Amazon S3
User
Account A
S3 Bucket
Policy
Amazon S3

<!-- Page 632 -->

# IAM Roles vs Resource-Based Policies

- When you assume a role (user, application or service), you give up your original
permissions and take the permissions assigned to the role
- When using a resource-based policy, the principal doesn’t have to give up his
permissions
- Example: User in account A needs to scan a DynamoDB table in Account A
and dump it in an S3 bucket in Account B.
- Supported by: Amazon S3 buckets, SNS topics, SQS queues, etc…

<!-- Page 633 -->

# Amazon EventBridge – Security

- When a rule runs, it needs
permissions on the target
- Resource-based policy: Lambda,
SNS, SQS, S3 buckets, API
Gateway…
EventBridge
Rule
IAM Role
- IAM role: EC2 Auto Scaling,
Systems Manager Run
Command, ECS task…
EventBridge
Rule
Lambda with
Resource based Policy
e.g. Allow EventBridge
EC2 Auto Scaling

<!-- Page 634 -->

# IAM Permission Boundaries

- IAM Permission Boundaries are supported for users and roles (not groups)
- Advanced feature to use a managed policy to set the maximum permissions
an IAM entity can get.
IAM Permission Boundary
=
+
Example:
IAM Permissions
Through IAM Policy
No Permissions

<!-- Page 635 -->

# IAM Permission Boundaries

- Can be used in combinations of
AWS Organizations SCP
Use cases
- Delegate responsibilities to non
administrators within their permission
boundaries, for example create new IAM
users
- Allow developers to self-assign policies
and manage their own permissions, while
making sure they can’t “escalate” their
privileges (= make themselves admin)
https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html
- Useful to restrict one specific user
(instead of a whole account using
Organizations & SCP)

<!-- Page 636 -->

# IAM Policy Evaluation Logic

https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html

<!-- Page 637 -->

# Example IAM Policy

- Can you perform sqs:CreateQueue?
- Can you perform sqs:DeleteQueue?
- Can you perform
ec2:DescribeInstances?

<!-- Page 638 -->

# (successor to AWS Single Sign-On)

- One login (single sign-on) for all your
- AWS accounts in AWS Organizations
- Business cloud applications (e.g., Salesforce, Box, Microsoft 365, …)
- SAML2.0-enabled applications
- EC2 Windows Instances
- Identity providers
- Built-in identity store in IAM Identity Center
- 3rd party: Active Directory (AD), OneLogin, Okta…
AWS IAM Identity Center

<!-- Page 639 -->

# AWS IAM Identity Center – Login Flow

AWS IAM Identity Center

<!-- Page 640 -->

# AWS IAM Identity Center

AWS Cloud
SSO
login
AWS
Organization
Windows
EC2
Permission Sets
Business Cloud Apps
IAM Identity Center
Built-in Identity Store
Custom SAML2.0-enabled Apps
Browser Interface
Store / retrieve
User identities
Active Directory
Users & groups
(On-premises, cloud)

<!-- Page 641 -->

# IAM Identity Center

AWS Organization
Management Account
(in Management account)
Group (Developers)
OU (Development)
OU (Production)
Bob
Dev Account A
Alice
Prod Account A
assign
Dev Account B
Prod Account B
Permission Set
ReadOnlyAccess
Permission Set
FullAccess
assign

<!-- Page 642 -->

# IAM Identity Center

- Multi-Account Permissions
- Manage access across AWS accounts in your AWS Organization
- Permission Sets – a collection of one or more IAM Policies
assigned to users and groups to define AWS access
- Application Assignments
- SSO access to many SAML 2.0 business applications (Salesforce,
Box, Microsoft 365, …)
- Provide required URLs, certificates, and metadata
- Attribute-Based Access Control (ABAC)
- Fine-grained permissions based on users’ attributes stored in
IAM Identity Center Identity Store
- Example: cost center, title, locale, …
- Use case: Define permissions once, then modify AWS access by
changing the attributes
AWS Organization
Dev
Account
Prod
Account
RDS Aurora
RDS Aurora
IAM Role
IAM Role
assume
Permission Sets
(DB Admins)
Permission Sets
(DB Admins)
Database
Admins
AWS IAM Identity Center
Fine-grained Permissions and Assignments

<!-- Page 643 -->

# What is Microsoft Active Directory (AD)?

- Found on any Windows Server
with AD Domain Services
- Database of objects: User
Accounts, Computers, Printers,
File Shares, Security Groups
- Centralized security
management, create account,
assign permissions
- Objects are organized in trees
- A group of trees is a forest
Domain Controller
John
Password

<!-- Page 644 -->

# AWS Directory Services

- AWS Managed Microsoft AD
- Create your own AD in AWS, manage users
locally, supports MFA
- Establish “trust” connections with your onpremises AD
auth
On-prem AD
- AD Connector
- Directory Gateway (proxy) to redirect to onpremises AD, supports MFA
- Users are managed on the on-premises AD
trust
auth
AWS Managed AD
proxy
On-prem AD
auth
AD Connector
- Simple AD
- AD-compatible managed directory on AWS
- Cannot be joined with on-premises AD
Simple AD

<!-- Page 645 -->

# IAM Identity Center – Active Directory Setup

- Connect to an AWS Managed Microsoft AD (Directory Service)
- Integration is out of the box
IAM Identity
Center
connect
AWS Managed
Microsoft AD
- Connect to a Self-Managed Directory
- Create Two-way Trust Relationship using AWS Managed Microsoft AD
- Create an AD Connector
AWS Managed
Microsoft AD
IAM Identity
Center
connect
two-way trust relationship
connect
proxy
AD Connector

<!-- Page 646 -->

# AWS Control Tower

- Easy way to set up and govern a secure and compliant multi-account
AWS environment based on best practices
- AWS Control Tower uses AWS Organizations to create accounts
- Benefits:
- Automate the set up of your environment in a few clicks
- Automate ongoing policy management using guardrails
- Detect policy violations and remediate them
- Monitor compliance through an interactive dashboard

<!-- Page 647 -->

# AWS Control Tower – Guardrails

- Provides ongoing governance for your Control Tower environment (AWS Accounts)
- Preventive Guardrail – using SCPs (e.g., Restrict Regions across all your accounts)
- Detective Guardrail – using AWS Config (e.g., identify untagged resources)
AWS Control Tower
Guardrail
(Detective)
AWS Config
notify
trigger
(NON_COMPLIANT)
monitor un-tagged
resources
Member
Accounts
Admin
SNS
invoke
remediate
(add tags)
Lambda

<!-- Page 648 -->

# AWS Security & Encryption

KMS, Encryption SDK, SSM Parameter Store

<!-- Page 649 -->

# Encryption in flight (TLS / SSL)

- Data is encrypted before sending and decrypted after receiving
- TLS certificates help with encryption (HTTPS)
- Encryption in flight ensures no MITM (man in the middle attack) can
happen
Username: admin
Password: supersecret
TLS Encryption
Client
Client
Username: admin
Password: supersecret
aGVsbG8gd29
ybGQgZWh…
TLS Decryption
HTTPS Website
Server
Why encryption?

<!-- Page 650 -->

# Server-side encryption at rest

- Data is encrypted after being received by the server
- Data is decrypted before being sent
- It is stored in an encrypted form thanks to a key (usually a data key)
- The encryption / decryption keys must be managed somewhere, and
the server must have access to it
HTTP(S)
Object
+
Data key
Encryption
+
Decryption
Data key
AWS Service (e.g., S3)
HTTP(S)
Object
Why encryption?

<!-- Page 651 -->

# Client-side encryption

- Data is encrypted by the client and never decrypted by the server
- Data will be decrypted by a receiving client
- The server should not be able to decrypt the data
- Could leverage Envelope Encryption
+
Object
Encryption
Data key
(client-side)
Decryption
Object
+
Data key
(client-side)
Client
store
retrieve
Encrypted object
Any storage service
(FTP, S3, …)
Why encryption?

<!-- Page 652 -->

# AWS KMS (Key Management Service)

- Anytime you hear “encryption” for an AWS service, it’s most likely KMS
- AWS manages encryption keys for us
- Fully integrated with IAM for authorization
- Easy way to control access to your data
- Able to audit KMS Key usage using CloudTrail
- Seamlessly integrated into most AWS services (EBS, S3, RDS, SSM…)
- Never ever store your secrets in plaintext, especially in your code!
- KMS Key Encryption also available through API calls (SDK, CLI)
- Encrypted secrets can be stored in the code / environment variables

<!-- Page 653 -->

# KMS Keys Types

- KMS Keys is the new name of KMS Customer Master Key
- Symmetric (AES-256 keys)
- Single encryption key that is used to Encrypt and Decrypt
- AWS services that are integrated with KMS use Symmetric CMKs
- You never get access to the KMS Key unencrypted (must call KMS API to use)
- Asymmetric (RSA & ECC key pairs)
- Public (Encrypt) and Private Key (Decrypt) pair
- Used for Encrypt/Decrypt, or Sign/Verify operations
- The public key is downloadable, but you can’t access the Private Key unencrypted
- Use case: encryption outside of AWS by users who can’t call the KMS API

<!-- Page 654 -->

# AWS KMS (Key Management Service)

- Types of KMS Keys:
- AWS Owned Keys (free): SSE-S3, SSE-SQS, SSE-DDB (default key)
- AWS Managed Key: free (aws/service-name, example: aws/rds or aws/ebs)
- Customer managed keys created in KMS: $1 / month
- Customer managed keys imported: $1 / month
- + pay for API call to KMS ($0.03 / 10000 calls)
- Automatic Key rotation:
- AWS-managed KMS Key: automatic every 1 year
- Customer-managed KMS Key: (must be enabled) automatic & on-demand
- Imported KMS Key: only manual rotation possible using alias

<!-- Page 655 -->

# Copying Snapshots across regions

Region eu-west-2
EBS Volume
Encrypted
With KMS
EBS Snapshot
Encrypted
With KMS
Region ap-southeast-2
KMS Key A
EBS Volume
Encrypted
With KMS
KMS Key B
KMS Key A
EBS Snapshot
Encrypted
With KMS
KMS Key B
KMS ReEncrypt with KMS Key B

<!-- Page 656 -->

# KMS Key Policies

- Control access to KMS keys, “similar” to S3 bucket policies
- Difference: you cannot control access without them
- Default KMS Key Policy:
- Created if you don’t provide a specific KMS Key Policy
- Complete access to the key to the root user = entire AWS account
- Custom KMS Key Policy:
- Define users, roles that can access the KMS key
- Define who can administer the key
- Useful for cross-account access of your KMS key

<!-- Page 657 -->

# Copying Snapshots across accounts

1. Create a Snapshot, encrypted with
your own KMS Key (Customer
Managed Key)
2. Attach a KMS Key Policy to authorize
cross-account access
3. Share the encrypted snapshot
4. (in target) Create a copy of the
Snapshot, encrypt it with a CMK in
your account
5. Create a volume from the snapshot
KMS Key Policy

<!-- Page 658 -->

# KMS Multi-Region Keys

AWS KMS
us-west-2
multi-Region Replica key
arn:aws:kms:us-west-2:111122223333:
key/mrk-1234abcd12ab34cd56ef1234567890ab
us-east-1
eu-west-1
sync
multi-Region Primary key
multi-Region Replica key
arn:aws:kms:us-east-1:111122223333:
key/mrk-1234abcd12ab34cd56ef1234567890ab
arn:aws:kms:eu-west-1:111122223333:
key/mrk-1234abcd12ab34cd56ef1234567890ab
ap-southeast-2
multi-Region Replica key
arn:aws:kms:ap-southeast-2:111122223333:
key/mrk-1234abcd12ab34cd56ef1234567890ab

<!-- Page 659 -->

# KMS Multi-Region Keys

- Identical KMS keys in different AWS Regions that can be used interchangeably
- Multi-Region keys have the same key ID, key material, automatic rotation…
- Encrypt in one Region and decrypt in other Regions
- No need to re-encrypt or making cross-Region API calls
- KMS Multi-Region are NOT global (Primary + Replicas)
- Each Multi-Region key is managed independently
- Use cases: global client-side encryption, encryption on Global DynamoDB, Global Aurora

<!-- Page 660 -->

# DynamoDB Global Tables and KMS MultiRegion Keys Client-Side encryption

- We can encrypt specific attributes client-side
in our DynamoDB table using the Amazon
DynamoDB Encryption Client
us-east-1
1. Encrypt attribute
with primary MRK
- Combined with Global Tables, the client-side
encrypted data is replicated to other regions
- Using client-side encryption we can protect
specific fields and guarantee only decryption
if the client has access to an API key
2. Put encrypted
attribute
Client App
Attr
(SSN)
MRK
DDB Table
replication
- If we use a multi-region key, replicated in the
same region as the DynamoDB Global table,
then clients in these regions can use lowlatency API calls to KMS in their region to
decrypt the data client-side
KMS
3. Global Table
Replication
ap-southeast-2
4. Get encrypted
attribute
Client App
Attr
(SSN)
DDB Table
MRK
5. Decrypt attribute
with replica MRK
KMS

<!-- Page 661 -->

# Client-Side encryption

us-east-1
KMS
1. Encrypt attribute
with primary MRK
2. Put encrypted
column
Client App
Col
(SSN)
MRK
Table
replication
- We can encrypt specific attributes client-side
in our Aurora table using the AWS
Encryption SDK
- Combined with Aurora Global Tables, the
client-side encrypted data is replicated to
other regions
- If we use a multi-region key, replicated in the
same region as the Global Aurora DB, then
clients in these regions can use low-latency
API calls to KMS in their region to decrypt
the data client-side
- Using client-side encryption we can protect
specific fields and guarantee only decryption
if the client has access to an API key, we can
protect specific fields even from database
admins
3. Global DB
Replication
ap-southeast-2
4. Get encrypted
column
Client App
Col
(SSN)
Table
MRK
5. Decrypt attribute
with replica MRK
KMS
Global Aurora and KMS Multi-Region Keys

<!-- Page 662 -->

# Encryption Considerations

- Unencrypted objects and objects encrypted with SSE-S3 are replicated by default
- Objects encrypted with SSE-C (customer provided key) can be replicated
- For objects encrypted with SSE-KMS, you need to enable the option
- Specify which KMS Key to encrypt the objects within the target bucket
- Adapt the KMS Key Policy for the target key
- An IAM Role with kms:Decrypt for the source KMS Key and kms:Encrypt for the target KMS Key
- You might get KMS throttling errors, in which case you can ask for a Service Quotas increase
- You can use multi-region AWS KMS Keys, but they are currently treated as
independent keys by Amazon S3 (the object will still be decrypted and then
encrypted)
S3 Replication

<!-- Page 663 -->

# AMI Sharing Process Encrypted via KMS

1.
2.
3.
4.
5.
AMI in Source Account is encrypted with KMS Key
from Source Account
Must modify the image attribute to add a Launch
Permission which corresponds to the specified target
AWS account
Must share the KMS Keys used to encrypted the
snapshot the AMI references with the target account
/ IAM Role
The IAM Role/User in the target account must have
the permissions to DescribeKey, ReEncrypt*,
CreateGrant, Decrypt
When launching an EC2 instance from the AMI,
optionally the target account can specify a new KMS
key in its own account to re-encrypt the volumes
Account - A
AMI
KMS
Key
share
share
Account - B
launch
AMI
EC2 Instance

<!-- Page 664 -->

# SSM Parameter Store

- Secure storage for configuration and secrets
- Optional Seamless Encryption using KMS
- Serverless, scalable, durable, easy SDK
- Version tracking of configurations / secrets
- Security through IAM
- Notifications with Amazon EventBridge
- Integration with CloudFormation
Applications
Plaintext
configuration
Check IAM
permissions
Encrypted
configuration
SSM Parameter
Store
Decryption
Service
AWS KMS

<!-- Page 665 -->

# SSM Parameter Store Hierarchy

- /my-department/
- my-app/
- dev/
- db-url
- db-password
- prod/
- db-url
- db-password
- other-app/
GetParameters or
GetParametersByPath API
Dev Lambda
Function
Prod Lambda
Function
- /other-department/
- /aws/reference/secretsmanager/secret_ID_in_Secrets_Manager
- /aws/service/ami-amazon-linux-latest/amzn2-ami-hvm-x86_64-gp2 (public)

<!-- Page 666 -->

# Standard and advanced parameter tiers

Standard
Advanced
Total number of parameters
allowed
(per AWS account and
Region)
10,000
100,000
Maximum size of a
parameter value
4 KB
8 KB
Parameter policies available
No
Yes
Cost
No additional charge
Charges apply
Storage Pricing
Free
$0.05 per advanced parameter per
month

<!-- Page 667 -->

# Parameters Policies (for advanced parameters)

- Allow to assign a TTL to a parameter (expiration date) to force
updating or deleting sensitive data such as passwords
- Can assign multiple policies at a time
Expiration (to delete a parameter)
ExpirationNotification (EventBridge)
NoChangeNotification (EventBridge)

<!-- Page 668 -->

# AWS Secrets Manager

- Newer service, meant for storing secrets
- Capability to force rotation of secrets every X days
- Automate generation of secrets on rotation (uses Lambda)
- Integration with Amazon RDS (MySQL, PostgreSQL, Aurora)
- Secrets are encrypted using KMS
- Mostly meant for RDS integration

<!-- Page 669 -->

# AWS Secrets Manager – Multi-Region Secrets

- Replicate Secrets across multiple AWS Regions
- Secrets Manager keeps read replicas in sync with the primary Secret
- Ability to promote a read replica Secret to a standalone Secret
- Use cases: multi-region apps, disaster recovery strategies, multi-region DB…
us-east-1 (Primary)
Secrets
Manager
replicate
MySecret-A
(primary)
us-west-2 (Secondary)
MySecret-A
(replica)
Secrets
Manager

<!-- Page 670 -->

# AWS Certificate Manager (ACM)

- Easily provision, manage, and deploy TLS Certificates
- Provide in-flight encryption for websites (HTTPS)
HTTPS
- Supports both public and private TLS certificates
provision and
- Free of charge for public TLS certificates
maintain TLS certs
- Automatic TLS certificate renewal
AWS Certificate Manager
- Integrations with (load TLS certificates on)
- Elastic Load Balancers (CLB, ALB, NLB)
- CloudFront Distributions
- APIs on API Gateway
Application
Load
Balancer
HTTP
Auto Scaling group
EC2 Instance
EC2 Instance

<!-- Page 671 -->

# ACM – Requesting Public Certificates

1.
List domain names to be included in the certificate
- Fully Qualified Domain Name (FQDN): corp.example.com
- Wildcard Domain: *.example.com
2.
Select Validation Method: DNS Validation or Email validation
- DNS Validation is preferred for automation purposes
- Email validation will send emails to contact addresses in the WHOIS database
- DNS Validation will leverage a CNAME record to DNS config (ex: Route 53)
3.
4.
It will take a few hours to get verified
The Public Certificate will be enrolled for automatic renewal
- ACM automatically renews ACM-generated certificates 60 days before expiry

<!-- Page 672 -->

# ACM – Importing Public Certificates

- Option to generate the certificate
outside of ACM and then import it
- No automatic renewal, must import a
new certificate before expiry
- ACM sends daily expiration events
starting 45 days prior to expiration
- The # of days can be configured
- Events are appearing in EventBridge
- AWS Config has a managed rule
named acm-certificate-expiration-check
to check for expiring certificates
(configurable number of days)
ACM
ACM Events:
Daily Certificate Expiry
Lambda
Rule check
EventBridge
Rule events:
Non-compliance
AWS Config
SNS
SQS

<!-- Page 673 -->

# ACM – Integration with ALB

Application Load Balancer
With HTTP -> HTTPS redirect rule
HTTP
Auto Scaling group
Redirect to HTTPS
HTTPS
provision and
maintain TLS certs
AWS Certificate Manager
EC2 Instance
EC2 Instance

<!-- Page 674 -->

# API Gateway - Endpoint Types

- Edge-Optimized (default): For global clients
- Requests are routed through the CloudFront Edge locations (improves latency)
- The API Gateway still lives in only one region
- Regional:
- For clients within the same region
- Could manually combine with CloudFront (more control over the caching
strategies and the distribution)
- Private:
- Can only be accessed from your VPC using an interface VPC endpoint (ENI)
- Use a resource policy to define access

<!-- Page 675 -->

# ACM – Integration with API Gateway

- Create a Custom Domain Name in API Gateway
us-east-1
- Edge-Optimized (default): For global clients
- Requests are routed through the CloudFront Edge locations
(improves latency)
- The API Gateway still lives in only one region
- The TLS Certificate must be in the same region as CloudFront,
in us-east-1
- Then setup CNAME or (better) A-Alias record in Route 53
linked
certificate
CloudFront
ACM
API Gateway
Edge-Optimized
- Regional:
- For clients within the same region
- The TLS Certificate must be imported on API Gateway, in the
same region as the API Stage
- Then setup CNAME or (better) A-Alias record in Route 53
ap-southeast-2
linked
certificate
API Gateway
Regional
ACM

<!-- Page 676 -->

# CloudHSM

- KMS => AWS manages the software for encryption
- CloudHSM => AWS provisions encryption hardware
- Dedicated Hardware (HSM = Hardware Security Module)
- You manage your own encryption keys entirely (not AWS)
- HSM device is tamper resistant, FIPS 140-2 Level 3 compliance
- Supports both symmetric and asymmetric encryption (SSL/TLS keys)
- No free tier available
- Must use the CloudHSM Client Software
- Redshift supports CloudHSM for database encryption and key management
- Good option to use with SSE-C encryption

<!-- Page 677 -->

# CloudHSM Diagram

AWS manages the Hardware
SSL Connection
User manages the Keys
CloudHSM Client
IAM permissions:
- CRUD an HSM Cluster
AWS CloudHSM
CloudHSM Software:
- Manage the Keys
- Manage the Users

<!-- Page 678 -->

# CloudHSM – High Availability

- CloudHSM clusters are spread across Multi AZ (HA)
- Great for availability and durability
Availability Zone 1
CloudHSM 1
CloudHSM Client
Availability Zone 2
CloudHSM 2

<!-- Page 679 -->

# CloudHSM – Integration with AWS Services

CloudHSM
- Through integration with
AWS KMS
- Configure KMS Custom
Key Store with
CloudHSM
- Example: EBS, S3, RDS …
Connector
KMS Encryption
RDS DB
Instance
EBS Volume
AWS KMS
(Custom Key Store)
keys usage logs
CloudTrail

<!-- Page 680 -->

# CloudHSM vs. KMS

Feature
AWS KMS
AWS CloudHSM
Tenancy
Multi-Tenant
Single-Tenant
Standard
FIPS 140-2 Level 3
FIPS 140-2 Level 3
Master Keys
- 
- 
- 
AWS Owned CMK
AWS Managed CMK
Customer Managed CMK
Customer Managed CMK
Key Types
- 
- 
- 
Symmetric
Asymmetric
Digital Signing
- 
- 
- 
Symmetric
Asymmetric
Digital Signing & Hashing
Key Accessibility
Accessible in multiple AWS regions (can’t
access keys outside the region it’s created in)
- 
- 
Deployed and managed in a VPC
Can be shared across VPCs (VPC Peering)
Cryptographic
Acceleration
None
- 
- 
SSL/TLS Acceleration
Oracle TDE Acceleration
Access &
Authentication
AWS IAM
You create users and manage their permissions

<!-- Page 681 -->

# CloudHSM vs. KMS

Feature
AWS KMS
High Availability
AWS Managed Service
Audit Capability
- 
- 
Free Tier
Yes
CloudTrail
CloudWatch
AWS CloudHSM
Add multiple HSMs over different AZs
- 
- 
- 
No
CloudTrail
CloudWatch
MFA support

<!-- Page 682 -->

# AWS WAF – Web Application Firewall

- Protects your web applications from common web exploits (Layer 7)
- Layer 7 is HTTP (vs Layer 4 is TCP/UDP)
- Deploy on
- Application Load Balancer
- API Gateway
- CloudFront
- AppSync GraphQL API
- Cognito User Pool

<!-- Page 683 -->

# AWS WAF – Web Application Firewall

- Define Web ACL (Web Access Control List) Rules:
- IP Set: up to 10,000 IP addresses – use multiple Rules for more IPs
- HTTP headers, HTTP body, or URI strings Protects from common attack - SQL
injection and Cross-Site Scripting (XSS)
- Size constraints, geo-match (block countries)
- Rate-based rules (to count occurrences of events) – for DDoS protection
- Web ACL are Regional except for CloudFront
- A rule group is a reusable set of rules that you can add to a web ACL

<!-- Page 684 -->

# Balancer

- WAF does not support the Network Load Balancer (Layer 4)
- We can use Global Accelerator for fixed IP and WAF on the ALB
us-east-1
Application Load
Users
Global Accelerator
Fixed IPv4: 1.2.3.4
EC2 Instances
attached
WebACL
AWS WAF
WebACL must be in the same
AWS Region as ALB
WAF – Fixed IP while using WAF with a Load

<!-- Page 685 -->

# AWS Shield: protect from DDoS attack

- DDoS: Distributed Denial of Service – many requests at the same time
- AWS Shield Standard:
- Free service that is activated for every AWS customer
- Provides protection from attacks such as SYN/UDP Floods, Reflection attacks and other
layer 3/layer 4 attacks
- AWS Shield Advanced:
- Optional DDoS mitigation service ($3,000 per month per organization)
- Protect against more sophisticated attack on Amazon EC2, Elastic Load Balancing (ELB),
Amazon CloudFront, AWS Global Accelerator, and Route 53
- 24/7 access to AWS DDoS response team (DRP)
- Protect against higher fees during usage spikes due to DDoS
- Shield Advanced automatic application layer DDoS mitigation automatically creates,
evaluates and deploys AWS WAF rules to mitigate layer 7 attacks

<!-- Page 686 -->

# AWS Firewall Manager

- Manage rules in all accounts of an AWS Organization
- Security policy: common set of security rules
- WAF rules (Application Load Balancer, API Gateways, CloudFront)
- AWS Shield Advanced (ALB, CLB, NLB, Elastic IP, CloudFront)
- Security Groups for EC2, Application Load BAlancer and ENI resources in VPC
- AWS Network Firewall (VPC Level)
- Amazon Route 53 Resolver DNS Firewall
- Policies are created at the region level
- Rules are applied to new resources as they are created (good for compliance)
across all and future accounts in your Organization

<!-- Page 687 -->

# WAF vs. Firewall Manager vs. Shield

AWS WAF
AWS Firewall Manager
AWS Shield
- WAF, Shield and Firewall Manager are used together for comprehensive protection
- Define your Web ACL rules in WAF
- For granular protection of your resources, WAF alone is the correct choice
- If you want to use AWS WAF across accounts, accelerate WAF configuration,
automate the protection of new resources, use Firewall Manager with AWS WAF
- Shield Advanced adds additional features on top of AWS WAF, such as dedicated
support from the Shield Response Team (SRT) and advanced reporting.
- If you’re prone to frequent DDoS attacks, consider purchasing Shield Advanced

<!-- Page 688 -->

# Edge Location Mitigation (BP1, BP3)

- BP1 – CloudFront
- Web Application delivery at
the edge
- Protect from DDoS Common
Attacks (SYN floods, UDP
reflection…)
- BP1 – Global Accelerator
- Access your application from
the edge
- Integration with Shield for
DDoS protection
- Helpful if your backend is not
compatible with CloudFront
- BP3 – Route 53
- Domain Name Resolution at
the edge
- DDoS Protection mechanism
AWS Best Practices for DDoS Resiliency

<!-- Page 689 -->

# Best pratices for DDoS mitigation

- Infrastructure layer defense (BP1,
BP3, BP6)
- Protect Amazon EC2 against high
traffic
- That includes using Global
Accelerator, Route 53,
CloudFront, Elastic Load Balancing
- Amazon EC2 with Auto Scaling
(BP7)
- Helps scale in case of sudden
traffic surges including a flash
crowd or a DDoS attack
- Elastic Load Balancing (BP6)
- Elastic Load Balancing scales with
the traffic increases and will
distribute the traffic to many EC2
instances
AWS Best Practices for DDoS Resiliency

<!-- Page 690 -->

# Application Layer Defense

- Detect and filter malicious web
requests (BP1, BP2)
- CloudFront cache static content and
serve it from edge locations, protecting
your backend
- AWS WAF is used on top of
CloudFront and Application Load
Balancer to filter and block requests
based on request signatures
- WAF rate-based rules can
automatically block the IPs of bad
actors
- Use managed rules on WAF to block
attacks based on IP reputation, or
block anonymous Ips
- CloudFront can block specific
geographies
- Shield Advanced (BP1, BP2, BP6)
- Shield Advanced automatic application
layer DDoS mitigation automatically
creates, evaluates and deploys AWS
WAF rules to mitigate layer 7 attacks
AWS Best Practices for DDoS Resiliency

<!-- Page 691 -->

# Attack surface reduction

- Obfuscating AWS resources (BP1,
BP4, BP6)
- Using CloudFront, API Gateway, Elastic
Load Balancing to hide your backend
resources (Lambda functions, EC2
instances)
- Security groups and Network ACLs
(BP5)
- Use security groups and NACLs to
filter traffic based on specific IP at the
subnet or ENI-level
- Elastic IP are protected by AWS Shield
Advanced
- Protecting API endpoints (BP4)
- Hide EC2, Lambda, elsewhere
- Edge-optimized mode, or CloudFront
+ regional mode (more control for
DDoS)
- WAF + API Gateway: burst limits,
headers filtering, use API keys
AWS Best Practices for DDoS Resiliency

<!-- Page 692 -->

# Amazon GuardDuty

- Intelligent Threat discovery to protect your AWS Account
- Uses Machine Learning algorithms, anomaly detection, 3rd party data
- One click to enable (30 days trial), no need to install software
- Input data includes:
- CloudTrail Events Logs – unusual API calls, unauthorized deployments
- CloudTrail Management Events – create VPC subnet, create trail, …
- CloudTrail S3 Data Events – get object, list objects, delete object, …
- VPC Flow Logs – unusual internal traffic, unusual IP address
- DNS Logs – compromised EC2 instances sending encoded data within DNS queries
- Optional Features – EKS Audit Logs, RDS & Aurora, EBS, Lambda, S3 Data Events…
- Can setup EventBridge rules to be notified in case of findings
- EventBridge rules can target AWS Lambda or SNS
- Can protect against CryptoCurrency attacks (has a dedicated “finding” for it)

<!-- Page 693 -->

# Amazon GuardDuty

VPC Flow Logs
CloudTrail Logs
DNS Logs (AWS DNS)
SNS
GuardDuty
Optional Features
S3 Logs
EBS Volumes
EventBridge
Lambda Network
Activity
RDS & Aurora
Login Activity
EKS Audit Logs &
Runtime Monitoring
Lambda

<!-- Page 694 -->

# EventBridge

- Automated Security Assessments
SSM Agent
Lambda
Function
- For EC2 instances
- Leveraging the AWS System Manager (SSM) agent
- Analyze against unintended network accessibility
- Analyze the running OS against known vulnerabilities
- For Container Images push to Amazon ECR
- Assessment of Container Images as they are pushed
Amazon
Inspector
Amazon ECR
Container Image
- For Lambda Functions
- Identifies software vulnerabilities in function code and package
dependencies
- Assessment of functions as they are deployed
assessment run state
& findings
- Reporting & integration with AWS Security Hub
- Send findings to Amazon Event Bridge
Security Hub
Amazon Inspector

<!-- Page 695 -->

# What does Amazon Inspector evaluate?

- Remember: only for EC2 instances, Container Images & Lambda functions
- Continuous scanning of the infrastructure, only when needed
- Package vulnerabilities (EC2, ECR & Lambda) – database of CVE
- Network reachability (EC2)
- A risk score is associated with all vulnerabilities for prioritization

<!-- Page 696 -->

# AWS Macie

- Amazon Macie is a fully managed data security and data privacy service
that uses machine learning and pattern matching to discover and protect
your sensitive data in AWS.
- Macie helps identify and alert you to sensitive data, such as personally
identifiable information (PII)
analyze
S3 Buckets
notify
Macie
Discover Sensitive Data (PII)
integrations
Amazon
EventBridge

<!-- Page 697 -->

# Amazon VPC

<!-- Page 698 -->

# VPC Components Diagram

VPC Flow Logs
Region
NACL
VPC
Internet
www
NACL
Public Subnet
Internet
Gateway
Amazon
DynamoDB
Corporate
Data Center
Private Subnet
Router
Security Group
CloudWatch
Private EC2 Instance
S3
NAT Gateway
Transit
Gateway
VPC Peering
Connections
VPN
DX
Route
Table
Route
Table
Security Group
Public EC2 Instance
Availability Zone
Security Group
Private EC2 Instance
VPC
Endpoint
VPN
Gateway
Customer
Gateway
Server
S2S VPN
Connection
Direct Connect
Connection
DX Location

<!-- Page 699 -->

# Understanding CIDR – IPv4

- Classless Inter-Domain Routing – a method for allocating IP addresses
- Used in Security Groups rules and AWS networking in general
- They help to define an IP address range:
- We’ve seen WW.XX.YY.ZZ/32 => one IP
- We’ve seen 0.0.0.0/0 => all IPs
- But we can define:192.168.0.0/26 =>192.168.0.0 – 192.168.0.63 (64 IP addresses)

<!-- Page 700 -->

# Understanding CIDR – IPv4

- A CIDR consists of two components
- Base IP
- Represents an IP contained in the range (XX.XX.XX.XX)
- Example: 10.0.0.0, 192.168.0.0, …
- Subnet Mask
- Defines how many bits can change in the IP
- Example: /0, /24, /32
- Can take two forms:
- /8 ó 255.0.0.0
- /16 ó 255.255.0.0
- /24 ó 255.255.255.0
- /32 ó 255.255.255.255

<!-- Page 701 -->

# /0 – all octets can change

- The Subnet Mask basically allows part of the underlying IP to get
additional next values from the base IP
. 168 .
192 . 168 .
192 . 168 .
192 . 168 .
192 . 168 .
192 . 168 .
192 . 168 .
192 . 168 .
192 . 168 .
0
0
…
0
.
.
.
.
.
.
.
.
.
192
. 168 .
…
0
0
.
.
0
192
0
0
0
0
0
0
0
0
0
/32 => allows for 1 IP (2!)
/31 => allows for 2 IP (2")
/30 => allows for 4 IP (2#)
/29 => allows for 8 IP (2$)
/28 => allows for 16 IP (2%)
/27 => allows for 32 IP (2&)
/26 => allows for 64 IP (2')
/25 => allows for 128 IP (2()
/24 => allows for 256 IP (2))
192.168.0.0
192.168.0.0 -> 192.168.0.1
192.168.0.0 -> 192.168.0.3
192.168.0.0 -> 192.168.0.7
192.168.0.0 -> 192.168.0.15
192.168.0.0 -> 192.168.0.31
192.168.0.0 -> 192.168.0.63
192.168.0.0 -> 192.168.0.127
192.168.0.0 -> 192.168.0.255
.
0
/16 => allows for 65,536 IP (2"')
192.168.0.0 -> 192.168.255.255
.
0
/0 => allows for All IPs
0.0.0.0 -> 255.255.255.255
0
0
0
0
0
0
0
Quick Memo
1*+
- 
- 
- 
- 
- 
.
Octets
2,-
.
3.-
.
4+/
Understanding CIDR – Subnet Mask
/32 – no octet can change
/24 – last octet can change
/16 – last 2 octets can change
/8 – last 3 octets can change

<!-- Page 702 -->

# Understanding CIDR – Little Exercise

- 192.168.0.0/24 = … ?
- 192.168.0.0 – 192.168.0.255 (256 IPs)
- 192.168.0.0/16 = … ?
- 192.168.0.0 – 192.168.255.255 (65,536 IPs)
- 134.56.78.123/32 = … ?
- Just 134.56.78.123
- 0.0.0.0/0
- All IPs!
- When in doubt, use this website https://www.ipaddressguide.com/cidr

<!-- Page 703 -->

# Public vs. Private IP (IPv4)

- The Internet Assigned Numbers Authority (IANA) established certain
blocks of IPv4 addresses for the use of private (LAN) and public
(Internet) addresses
- Private IP can only allow certain values:
- 10.0.0.0 – 10.255.255.255 (10.0.0.0/8) ç in big networks
- 172.16.0.0 – 172.31.255.255 (172.16.0.0/12) ç AWS default VPC in that range
- 192.168.0.0 – 192.168.255.255 (192.168.0.0/16) ç e.g., home networks
- All the rest of the IP addresses on the Internet are Public

<!-- Page 704 -->

# Default VPC Walkthrough

- All new AWS accounts have a default VPC
- New EC2 instances are launched into the default VPC if no subnet is
specified
- Default VPC has Internet connectivity and all EC2 instances inside it
have public IPv4 addresses
- We also get a public and a private IPv4 DNS names

<!-- Page 705 -->

# VPC in AWS – IPv4

- VPC = Virtual Private Cloud
- You can have multiple VPCs in an AWS region (max. 5 per region – soft limit)
- Max. CIDR per VPC is 5, for each CIDR:
- Min. size is /28 (16 IP addresses)
- Max. size is /16 (65536 IP addresses)
- Because VPC is private, only the Private IPv4 ranges are allowed:
- 10.0.0.0 – 10.255.255.255 (10.0.0.0/8)
- 172.16.0.0 – 172.31.255.255 (172.16.0.0/12)
- 192.168.0.0 – 192.168.255.255 (192.168.0.0/16)
- Your VPC CIDR should NOT overlap with your other networks (e.g., corporate)

<!-- Page 706 -->

# State of Hands-on

Region
VPC

<!-- Page 707 -->

# Adding Subnets

Region
VPC
Public Subnet
Private Subnet
Availability Zone

<!-- Page 708 -->

# VPC – Subnet (IPv4)

- AWS reserves 5 IP addresses (first 4 & last 1) in each subnet
- These 5 IP addresses are not available for use and can’t be assigned to an
EC2 instance
- Example: if CIDR block 10.0.0.0/24, then reserved IP addresses are:
- 10.0.0.0 – Network Address
- 10.0.0.1 – reserved by AWS for the VPC router
- 10.0.0.2 – reserved by AWS for mapping to Amazon-provided DNS
- 10.0.0.3 – reserved by AWS for future use
- 10.0.0.255 – Network Broadcast Address. AWS does not support broadcast in a VPC,
therefore the address is reserved
- Exam Tip, if you need 29 IP addresses for EC2 instances:
- You can’t choose a subnet of size /27 (32 IP addresses, 32 – 5 = 27 < 29)
- You need to choose a subnet of size /26 (64 IP addresses, 64 – 5 = 59 > 29)

<!-- Page 709 -->

# Internet Gateway (IGW)

- Allows resources (e.g., EC2 instances) in a VPC connect to the Internet
- It scales horizontally and is highly available and redundant
- Must be created separately from a VPC
- One VPC can only be attached to one IGW and vice versa
- Internet Gateways on their own do not allow Internet access…
- Route tables must also be edited!

<!-- Page 710 -->

# Adding Internet Gateway

Region
VPC
Public Subnet
Private Subnet
Internet
Gateway
Availability Zone

<!-- Page 711 -->

# Editing Route Tables

Region
VPC
Internet
www
Public Subnet
Internet
Gateway
Private Subnet
Router
Route
Table
Security Group
Public EC2 Instance
Availability Zone

<!-- Page 712 -->

# Bastion Hosts

- We can use a Bastion Host to SSH into
our private EC2 instances
- The bastion is in the public subnet which is
then connected to all other private subnets
- Bastion Host security group must allow
inbound from the internet on port 22 from
restricted CIDR, for example the public
CIDR of your corporation
- Security Group of the EC2 Instances must
allow the Security Group of the Bastion
Host, or the private IP of the Bastion host
Users
SSH
VPC
Public Subnet
Security Group (BastionHost-SG)
EC2 Instance
(Bastion Host)
Private Subnet
SSH
Security Group (LinuxInstance-SG)

<!-- Page 713 -->

# NAT Instance (outdated, but still at the exam)

- NAT = Network Address Translation
- Allows EC2 instances in private subnets to
connect to the Internet
- Must be launched in a public subnet
- Must disable EC2 setting: Source /
destination Check
- Must have Elastic IP attached to it
- Route Tables must be configured to route
traffic from private subnets to the NAT
Instance
Server
(IP: 50.60.4.10)
Src.: 50.60.4.10
Dest.: 12.34.56.78
Dest.: 50.60.4.10
VPC
Src.: 12.34.56.78
Public Subnet
Security Group (NATInstance-SG)
EIP
(IP: 12.34.56.78)
Dest.: 50.60.4.10
Src.: 50.60.4.10
Src.: 10.0.0.20
Dest.: 10.0.0.20
Private Subnet
IP: 10.0.0.10
NAT Instance
IP: 10.0.0.20

<!-- Page 714 -->

# NAT Instance

Region
VPC
Internet
www
Public Subnet
Internet
Gateway
Router
Private Subnet
Security Group
Security Group
Private EC2 Instance
EIP
Route
Table
Route
Table
Security Group
Public EC2 Instance
Availability Zone

<!-- Page 715 -->

# NAT Instance – Comments

- Pre-configured Amazon Linux AMI is available
- Reached the end of standard support on December 31, 2020
- Not highly available / resilient setup out of the box
- You need to create an ASG in multi-AZ + resilient user-data script
- Internet traffic bandwidth depends on EC2 instance type
- You must manage Security Groups & rules:
- Inbound:
- Allow HTTP / HTTPS traffic coming from Private Subnets
- Allow SSH from your home network (access is provided through Internet Gateway)
- Outbound:
- Allow HTTP / HTTPS traffic to the Internet

<!-- Page 716 -->

# NAT Gateway

- AWS-managed NAT, higher bandwidth, high availability, no administration
- Pay per hour for usage and bandwidth
- NATGW is created in a specific Availability Zone, uses an Elastic IP
- Can’t be used by EC2 instance in the same subnet (only from other
subnets)
- Requires an IGW (Private Subnet => NATGW => IGW)
- 5 Gbps of bandwidth with automatic scaling up to 100 Gbps
- No Security Groups to manage / required

<!-- Page 717 -->

# NAT Gateway

Region
VPC
Internet
www
Public Subnet
Internet
Gateway
Private Subnet
Security Group
Router
Private EC2 Instance
Route
Table
Route
Table
Security Group
Public EC2 Instance
Availability Zone

<!-- Page 718 -->

# NAT Gateway with High Availability

Internet
- NAT Gateway is resilient within a
single Availability Zone
www
Region
Internet
Gateway
VPC
- Must create multiple NAT
Gateways in multiple AZs for faulttolerance
- There is no cross-AZ failover
needed because if an AZ goes
down it doesn't need NAT
Public Subnet
Router
Public Subnet
NAT Gateway
NAT Gateway
Private Subnet
Private Subnet
EC2 Instance
EC2 Instance
AZ - A
AZ - B

<!-- Page 719 -->

# NAT Gateway vs. NAT Instance

NAT Gateway
NAT Instance
Availability
Highly available within AZ (create in another AZ)
Use a script to manage failover between instances
Bandwidth
Up to 100 Gbps
Depends on EC2 instance type
Maintenance
Managed by AWS
Managed by you (e.g., software, OS patches, …)
Cost
Per hour & amount of data transferred
Per hour, EC2 instance type and size, + network $
Public IPv4
Private IPv4
Security Groups
Use as Bastion Host?
More at: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-comparison.html

<!-- Page 720 -->

# Regional NAT Gateway (RNAT)

- Highly available NAT Gateway associated
with VPC
- RNAT has its own route tables
- Eliminating the need for per-AZ
deployments (shared across AZs)
- You don’t need to create Public Subnets
in your VPC to host RNAT
- Automatically detects resources in a new
AZ and expands to this AZ
Internet
Internet Gateway
(igw-1234)
VPC
Destination
Target
10.0.0.0/16
local
0.0.0.0/0
igw-1234
NAT Gateway
(nat-1234)
Destination
Target
10.0.0.0/16
local
0.0.0.0/0
nat-1234
AZ - A
AZ - B
Private
Subnet - A
Private
Subnet - B

<!-- Page 721 -->

# Security Groups & NACLs

Outgoing Request
Incoming Request
Subnet
Subnet
1
2
NACL Outbound
Rules (Stateless)
Outbound Allowed
(Stateful)
3
Security Group
NACL Outbound
Rules
2
SG Outbound
Rules
Security Group
1
NACL
SG Inbound
Rules
NACL
NACL Inbound
Rules
EC2 Instance
NACL Inbound
Rules (Stateless)
3
Inbound Allowed
(Stateful)
EC2 Instance

<!-- Page 722 -->

# Network Access Control List (NACL)

- NACL are like a firewall which control traffic from and to subnets
- One NACL per subnet, new subnets are assigned the Default NACL
- You define NACL Rules:
- Rules have a number (1-32766), higher precedence with a lower number
- First rule match will drive the decision
- Example: if you define #100 ALLOW 10.0.0.10/32 and #200 DENY 10.0.0.10/32, the IP
address will be allowed because 100 has a higher precedence over 200
- The last rule is an asterisk (*) and denies a request in case of no rule match
- AWS recommends adding rules by increment of 100
- Newly created NACLs will deny everything
- NACL are a great way of blocking a specific IP address at the subnet level

<!-- Page 723 -->

# NACLs

Region
NACL
VPC
Internet
www
NACL
Public Subnet
Internet
Gateway
Private Subnet
Security Group
Router
NAT Gateway
Private EC2 Instance
Route
Table
Route
Table
Security Group
Public EC2 Instance
Availability Zone

<!-- Page 724 -->

# Default NACL

- Accepts everything inbound/outbound with the subnets it’s associated with
- Do NOT modify the Default NACL, instead create custom NACLs
Default NACL for a VPC that supports IPv4
Inbound Rules
Rule #
Type
Protocol
Port Range
Source
Allow/Deny
100
All IPv4 Traffic
All
All
0.0.0.0/0
ALLOW
*
All IPv4 Traffic
All
All
0.0.0.0/0
DENY
Rule #
Type
Protocol
Port Range
Destination
Allow/Deny
100
All IPv4 Traffic
All
All
0.0.0.0/0
ALLOW
*
All IPv4 Traffic
All
All
0.0.0.0/0
DENY
Outbound Rules

<!-- Page 725 -->

# Ephemeral Ports

- For any two endpoints to establish a connection, they must use ports
- Clients connect to a defined port, and expect a response on an ephemeral port
- Different Operating Systems use different port ranges, examples:
- IANA & MS Windows 10 è 49152 – 65535
- Many Linux Kernels è 32768 – 60999
Request
Client
IP: 11.22.33.44
Ephemeral Port: 50105
Src. IP
11.22.33.44
Src. Port
50105
Dest. IP
55.66.77.88
Dest. Port
443
Payload …
Payload …
Dest. IP
11.22.33.44
Dest. Port
50105
Src. IP
55.66.77.88
Src. Port
443
Response
Web Server
IP: 55.66.77.88
Fixed Port: 443

<!-- Page 726 -->

# NACL with Ephemeral Ports

VPC
Database Tier
Web Subnet (Public)
DB Subnet (Private)
Web-NACL
Client
Ephemeral
Port
Allow Outbound TCP
On port 3306
To DB Subnet CIDR
Allow Inbound TCP
On port 3306
From Web Subnet CIDR
Allow Inbound TCP
On port 1024-65535
From DB Subnet CIDR
Allow Outbound TCP
On port 1024-65535
To Web Subnet CIDR
DB-NACL
Web Tier
DB Instance
Port 3306
https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html#nacl-ephemeral-ports

<!-- Page 727 -->

# target subnets CIDR

VPC
Database Tier
Web Subnet - A (Public)
DB Subnet – A (Private)
Web-NACL
Web Subnet - B (Public)
DB-NACL
Web Tier
DB Instance
DB Subnet – B (Private)
DB Instance
Create NACL rules for each

<!-- Page 728 -->

# Security Group vs. NACLs

Security Group
NACL
Operates at the instance level
Operates at the subnet level
Supports allow rules only
Supports allow rules and deny rules
Stateful: return traffic is automatically allowed,
regardless of any rules
Stateless: return traffic must be explicitly allowed by
rules (think of ephemeral ports)
All rules are evaluated before deciding whether to
allow traffic
Rules are evaluated in order (lowest to highest) when
deciding whether to allow traffic, first match wins
Applies to an EC2 instance when specified by
someone
Automatically applies to all EC2 instances in the
subnet that it’s associated with
NACL Examples: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html

<!-- Page 729 -->

# VPC Peering

- Privately connect two VPCs using AWS’
network
- Make them behave as if they were in the
same network
- Must not have overlapping CIDRs
- VPC Peering connection is NOT transitive
(must be established for each VPC that
need to communicate with one another)
- You must update route tables in each VPC’s
subnets to ensure EC2 instances can
communicate with each other
VPC - A
(A – B)
(A – C)
VPC - B
(B – C)
VPC - C

<!-- Page 730 -->

# VPC Peering – Good to know

- You can create VPC Peering connection between VPCs in different AWS
accounts/regions
- You can reference a security group in a peered VPC (works cross
accounts – same region)
Account ID

<!-- Page 731 -->

# VPC Peering

Region
NACL
VPC
Internet
www
NACL
Public Subnet
Internet
Gateway
Private Subnet
Security Group
Router
NAT Gateway
Private EC2 Instance
Route
Table
Connections
Route
Table
Security Group
Public EC2 Instance
Availability Zone

<!-- Page 732 -->

# VPC Endpoints

Amazon
DynamoDB
Region
NACL
VPC
Internet
www
NACL
Public Subnet
Internet
Gateway
Private Subnet
Router
Security Group
CloudWatch
Private EC2 Instance
S3
NAT Gateway
Route
Table
VPC Peering
Connections
Route
Table
Security Group
Public EC2 Instance
Availability Zone
VPC
Endpoint

<!-- Page 733 -->

# VPC Endpoints (AWS PrivateLink)

- Every AWS service is publicly exposed
(public URL)
- VPC Endpoints (powered by AWS
PrivateLink) allows you to connect to AWS
services using a private network instead of
using the public Internet
- They’re redundant and scale horizontally
- They remove the need of IGW, NATGW, …
to access AWS Services
- In case of issues:
Amazon SNS
www
Region
VPC
Internet
Gateway
Public Subnet
NAT
Gateway
EC2 Instance
Private Subnet
EC2 Instance
Option 1
Option 2
VPC Endpoint
- Check DNS Setting Resolution in your VPC
- Check Route Tables
Amazon SNS

<!-- Page 734 -->

# Types of Endpoints

- Interface Endpoints (powered by PrivateLink)
- Provisions an ENI (private IP address) as an entry
point (must attach a Security Group)
- Supports most AWS services
- $ per hour + $ per GB of data processed
Region
VPC
Private Subnet
VPC Endpoint
(Interface)
EC2 Instance
ENI (PrivateLink)
Amazon SNS
- Gateway Endpoints
- Provisions a gateway and must be used as a
target in a route table (does not use security
groups)
- Supports both S3 and DynamoDB
- Free
Region
VPC
Private Subnet
Amazon S3
VPC Endpoint
EC2 Instance
(Gateway)
OR
Amazon
DynamoDB

<!-- Page 735 -->

# Gateway or Interface Endpoint for S3?

- Gateway is most likely going to be
preferred all the time at the exam
- Cost: free for Gateway, $ for
interface endpoint
- Interface Endpoint is preferred
access is required from onpremises (Site to Site VPN or
Direct Connect), a different VPC
or a different region
Users
AWS Cloud
S2S VPN
Region
VPC
Interface
Endpoint
In-VPC
Apps
Gateway
Endpoint
PrivateLink
Direct Connect
Amazon S3

<!-- Page 736 -->

# Lambda in VPC accessing DynamoDB

- DynamoDB is a public service
from AWS
- Option 1: Access from the public
internet
- Because Lambda is in a VPC, it
needs a NAT Gateway in a public
subnet and an internet gateway
- Option 2 (better & free): Access
from the private VPC network
- Deploy a VPC Gateway endpoint
for DynamoDB
- Change the Route Tables
AWS Cloud
Public subnet
NAT
IGW
DynamoDB
Private subnet
VPC Gateway Endpoint
For DynamoDB

<!-- Page 737 -->

# VPC Flow Logs

- Capture information about IP traffic going into your interfaces:
- VPC Flow Logs
- Subnet Flow Logs
- Elastic Network Interface (ENI) Flow Logs
- Helps to monitor & troubleshoot connectivity issues
- Flow logs data can go to S3, CloudWatch Logs, and Kinesis Data Firehose
- Captures network information from AWS managed interfaces too: ELB,
RDS, ElastiCache, Redshift, WorkSpaces, NATGW, Transit Gateway…

<!-- Page 738 -->

# VPC Flow Logs

Region
NACL
VPC
Internet
www
NACL
Public Subnet
Internet
Gateway
Router
Route
Table
Route
Table
Security Group
Public EC2 Instance
Availability Zone
Security Group
CloudWatch
Private EC2 Instance
S3
NAT Gateway
Amazon
DynamoDB
VPC Peering
Connections
Private Subnet
VPC
Endpoint

<!-- Page 739 -->

# VPC Flow Logs Syntax

version
account-id
interface-id
dstaddr
srcaddr
dstport packets
srcport protocol bytes
start
action
end
log-status
- srcaddr & dstaddr – help identify problematic IP
- srcport & dstport – help identity problematic ports
- Action – success or failure of the request due to Security Group / NACL
- Can be used for analytics on usage patterns, or malicious behavior
- Query VPC flow logs using Athena on S3 or CloudWatch Logs Insights
- Flow Logs examples: https://docs.aws.amazon.com/vpc/latest/userguide/flow-logsrecords-examples.html

<!-- Page 740 -->

# VPC Flow Logs – Troubleshoot SG & NACL issues

Look at the “ACTION” field
Incoming Requests
Outgoing Requests
- Inbound REJECT => NACL or SG
- Outbound REJECT => NACL or SG
- Inbound ACCEPT, Outbound REJECT =>
NACL
- Outbound ACCEPT, Inbound REJECT =>
NACL
Subnet
Subnet
NACL Inbound
Rules
Security Group
NACL Outbound
Rules
NACL
NACL Outbound
Rules (Stateless)
SG Outbound
Rules
Security Group
Inbound Allowed
(Stateful)
EC2 Instance
NACL
SG Inbound
Rules
Outbound Allowed
(Stateful)
EC2 Instance
NACL Inbound
Rules (Stateless)

<!-- Page 741 -->

# VPC Flow Logs – Architectures

Top-10 IP addresses
VPC Flow Logs
CloudWatch
Contributor Insights
CloudWatch Logs
Alert
Metric Filter
SSH, RDP…
VPC Flow Logs
CloudWatch Logs
CW Alarm
Amazon SNS
VPC Flow Logs
S3 Bucket
Amazon
Athena
Amazon
QuickSight

<!-- Page 742 -->

# VPC Flow Logs – CloudWatch Permissions

- IAM Service Role associated with VPC Flow Logs must have the required
permissions to publish logs to CloudWatch Logs
- logs:CreateLogGroup, logs:CreateLogStream, or logs:PutLogEvents
IAM Service Role
logs
VPC Flow Logs
CloudWatch Logs

<!-- Page 743 -->

# AWS Site-to-Site VPN

VPC Flow Logs
Region
NACL
VPC
Internet
www
NACL
Public Subnet
Internet
Gateway
Private Subnet
Router
Route
Table
Route
Table
Security Group
Public EC2 Instance
Availability Zone
Security Group
CloudWatch
Private EC2 Instance
S3
NAT Gateway
Amazon
DynamoDB
VPC Peering
Connections
Corporate
Data Center
Security Group
Private EC2 Instance
VPC
Endpoints
VPN
Gateway
Customer
Gateway
S2S VPN
Connection
Server

<!-- Page 744 -->

# AWS Site-to-Site VPN

- Virtual Private Gateway (VGW)
- VPN concentrator on the AWS side of the VPN connection
- VGW is created and attached to the VPC from which you want to create the
Site-to-Site VPN connection
- Possibility to customize the ASN (Autonomous System Number)
- Customer Gateway (CGW)
- Software application or physical device on customer side of the VPN connection
- https://docs.aws.amazon.com/vpn/latest/s2svpn/your-cgw.html#DevicesTested

<!-- Page 745 -->

# Site-to-Site VPN Connections

Route Table
VPC
(Route Propagation enabled)
Private Subnet
- Customer Gateway Device (On-premises)
Security Group
- What IP address to use?
- Public Internet-routable IP address for your Customer
Gateway device
- If it’s behind a NAT device that’s enabled for NAT
traversal (NAT-T), use the public IP address of the NAT
device
- Important step: enable Route Propagation for
the Virtual Private Gateway in the route table
that is associated with your subnets
- If you need to ping your EC2 instances from
on-premises, make sure you add the ICMP
protocol on the inbound of your security
groups
Virtual Private
Gateway
OR
Customer
Gateway
(Public IP)
NAT Device
(Public IP)
Corporate Data Center
Customer
Gateway
(Private IP)
Server

<!-- Page 746 -->

# AWS VPN CloudHub

- Provide secure communication between
multiple sites, if you have multiple VPN
connections
Customer Network
VPC
Availability Zone
- Low-cost hub-and-spoke model for
primary or secondary network connectivity
between different locations (VPN only)
- It’s a VPN connection so it goes over the
public Internet
- To set it up, connect multiple VPN
connections on the same VGW, setup
dynamic routing and configure route tables
Customer
Gateway
Private Subnet 1
Customer Network
EC2 Instances
Availability Zone
Private Subnet 2
Virtual
Private
Gateway
(VGW)
Customer
Gateway
Customer Network
EC2 Instances
Customer
Gateway

<!-- Page 747 -->

# Direct Connect (DX)

- Provides a dedicated private connection from a remote network to your VPC
- Dedicated connection must be setup between your DC and AWS Direct
Connect locations
- You need to setup a Virtual Private Gateway on your VPC
- Access public resources (S3) and private (EC2) on same connection
- Use Cases:
- Increase bandwidth throughput - working with large data sets – lower cost
- More consistent network experience - applications using real-time data feeds
- Hybrid Environments (on prem + cloud)
- Supports both IPv4 and IPv6

<!-- Page 748 -->

# Direct Connect Diagram

Region
(us-east-1)
Corporate
data center
VPC
Private Subnet
VLAN 1
VLAN 2
Virtual Private Gateway
EC2 Instances
AWS Direct
Connect Endpoint
Customer or
partner router
AWS Cage
Customer or
partner cage
AWS Direct Connect Location
Amazon Glacier
Amazon S3
Private virtual interface
Public virtual interface
Customer
router/firewall
Customer Network

<!-- Page 749 -->

# Direct Connect Gateway

- If you want to setup a Direct Connect to one or more VPC in many
different regions (same account), you must use a Direct Connect Gateway
Region
(us-east-1)
Region
(us-west-1)
VPC
VPC
Customer network
10.0.0.0/16
172.16.0.0/16
Private virtual
interface
Private virtual
interface
Private virtual
interface
AWS Direct
Connect
connection

<!-- Page 750 -->

# Direct Connect – Connection Types

- Dedicated Connections: from 1 Gbps to 400 Gbps
- Physical ethernet port dedicated to a customer
- Request made to AWS first, then completed by AWS Direct Connect Partners
- Hosted Connections: from 50Mbps to 25 Gbps
- Connection requests are made via AWS Direct Connect Partners
- Capacity can be added or removed on demand
- Lead times are often longer than 1 month to establish a new connection

<!-- Page 751 -->

# Direct Connect – Encryption

- Data in transit is not encrypted but is
private
- AWS Direct Connect + VPN
provides an IPsec-encrypted private
connection
- Good for an extra level of security,
but slightly more complex to put in
place
Region
(us-east-1)
VPC
Availability Zone
(us-east-1a)
Private Subnet 1
Client
Client
EC2 Instances
Availability Zone
(us-east-1b)
AWS Direct
VPN
Connect Endpoint Connection
Customer
router/firewall
Private Subnet 2
AWS Direct
Connect Location
EC2 Instances
Corporate
data center
Customer Network

<!-- Page 752 -->

# Direct Connect - Resiliency

High Resiliency for Critical Workloads
Maximum Resiliency for Critical Workloads
Corporate
data center
Region
AWS Direct
Connect Location - 1
Corporate
data center
Region
AWS Direct
Connect Location - 1
Corporate
data center
Corporate
data center
AWS Direct
Connect Location - 2
AWS Direct
Connect Location - 2
One connection at multiple locations
Maximum resilience is achieved by separate connections
terminating on separate devices in more than one location.

<!-- Page 753 -->

# Site-to-Site VPN connection as a backup

- In case Direct Connect fails, you can set up a backup Direct Connect
connection (expensive), or a Site-to-Site VPN connection
AWS Cloud
Direct Connect
Primary Connection
VPC
Corporate DC
Site-to-Site VPN
Backup Connection

<!-- Page 754 -->

# Network topologies can become complicated

VPC Peering
Connection
VPN Connection
Customer Gateway
Amazon VPC
Amazon VPC
VPN Connection
VPC Peering
Connection
VPC Peering
Connection
VPC Peering
Connection
Direct Connect
Gateway
VPN Connection
Amazon VPC
VPC Peering
Connection
Amazon VPC

<!-- Page 755 -->

# Transit Gateway

- For having transitive peering between thousands of VPC and
on-premises, hub-and-spoke (star) connection
AWS Direct
Connect Gateway
- Regional resource, can work cross-region
- Share cross-account using Resource Access Manager (RAM)
- You can peer Transit Gateways across regions
- Route Tables: limit which VPC can talk with other VPC
Amazon VPC
- Works with Direct Connect Gateway, VPN connections
- Supports IP Multicast (not supported by any other AWS
service)
Amazon VPC
Transit
Gateway
Amazon VPC
Amazon VPC
VPN Connection
Customer Gateway

<!-- Page 756 -->

# Transit Gateway: Site-to-Site VPN ECMP

VPC
VP
C
hm
VPC
attac
hm
m
tach
t
a
VPC
en
t
VPN attachment
ent
ent
at
ta
c
hm
VPC
at
ta
c
en
t
VPC
VP
C
- ECMP = Equal-cost multi-path
routing
- Routing strategy to allow to
forward a packet over multiple
best path
- Use case: create multiple Siteto-Site VPN connections to
increase the bandwidth of your
connection to AWS
VPC
AWS Transit Gateway
Corporate data center
172.16.0.0/16

<!-- Page 757 -->

# Transit Gateway: throughput with ECMP

VPN to transit gateway
VPN to virtual private gateway
1x
= 1x
1x
= 1.25 Gbps
VPN connection
(2 tunnels)
VPC
VPC
VPC
VPC
VPC
1x
= 1x
1x
= 2.5 Gbps (ECMP) – 2 tunnels used
2x
= 5.0 Gbps (ECMP)
3x
= 7.5 Gbps (ECMP)
per GB of TGW
processed data

<!-- Page 758 -->

# between multiple accounts

AWS Cloud
Corporate
data center
Region
Account 1
Clients
VPC
VLAN
Transit VIF
Account 2
Transit
Gateway
Direct
Connect
Gateway
Clients
AWS Direct
Connect endpoint
Customer
router/firewall
AWS Direct
Connect Location
Servers
VPC
You can use AWS Resource Access Manager to share Transit
Gateway with other accounts.
Transit Gateway – Share Direct Connect

<!-- Page 759 -->

# VPC – Traffic Mirroring

- Allows you to capture and inspect network
traffic in your VPC
- Route the traffic to security appliances that
you manage
- Capture the traffic
- From (Source) – ENIs
- To (Targets) – an ENI or a Network Load Balancer
- Capture all packets or capture the packets of
your interest (optionally, truncate packets)
- Source and Target can be in the same VPC or
different VPCs (VPC Peering)
- Use cases: content inspection, threat
monitoring, troubleshooting, …
Source A
Source B
Inbound &
Outbound traffic
Inbound &
Outbound
traffic
Traffic Mirroring
(filter traffic, optional)
Network Load
Balancer
Auto Scaling group
EC2 instances with Security Appliances

<!-- Page 760 -->

# What is IPv6?

- IPv4 designed to provide 4.3 Billion addresses (they’ll be exhausted soon)
- IPv6 is the successor of IPv4
- IPv6 is designed to provide 3.4 × 10,- unique IP addresses
- Every IPv6 address in AWS is public and Internet-routable (no private range)
- Format è x.x.x.x.x.x.x.x (x is hexadecimal, range can be from 0000 to ffff)
- Examples:
- 2001:db8:3333:4444:5555:6666:7777:8888
- 2001:db8:3333:4444:cccc:dddd:eeee:ffff
- :: è all 8 segments are zero
- 2001:db8:: è the last 6 segments are zero
- ::1234:5678 è the first 6 segments are zero
- 2001:db8::1234:5678 è the middle 4 segments are zero

<!-- Page 761 -->

# IPv6 in VPC

Internet
- IPv4 cannot be disabled for your VPC and subnets
- You can enable IPv6 (they’re public IP addresses)
to operate in dual-stack mode
- Your EC2 instances will get at least a private
internal IPv4 and a public IPv6
- They can communicate using either IPv4 or IPv6
to the internet through an Internet Gateway
VPC
Internet
Gateway
IPv4 & IPv6
EC2 Instance
(Private IP: 10.0.0.5)
(IPv6: 2001:db8::ff00:42:8329)

<!-- Page 762 -->

# IPv4 Troubleshooting

- IPv4 cannot be disabled for your VPC
and subnets
- So, if you cannot launch an EC2 instance
in your subnet
- It’s not because it cannot acquire an IPv6
(the space is very large)
- It’s because there are no available IPv4 in
your subnet
- Solution: create a new IPv4 CIDR in
your subnet
User
create
VPC
(IPv4: 192.168.0.0/24)
(IPv4: 10.0.0.0/24)
(IPv6: 2001:db8:1234:5678::/56)
…
192.168.0.10
192.168.0.15
10.0.0.35

<!-- Page 763 -->

# IPv6: 2001:db8::e1c3

- Used for IPv6 only
- (similar to a NAT Gateway but for IPv6)
- Allows instances in your VPC outbound
connections over IPv6 while preventing
the internet to initiate an IPv6 connection
to your instances
- You must update the Route Tables
Internet
initiate connections
from both sides
VPC
Internet
Gateway
Egress-only Internet Gateway
can’t initiate
connections from
Internet
Egress-only
Internet Gateway
Public Subnet
Private Subnet
IPv6: 2001:db8::b1c2

<!-- Page 764 -->

# IPv6 Routing

Route Table
(Public Subnet)
Region
VPC
(IPv4: 10.0.0.0/16)
(IPv6: 2001:db8:1234:1a00::/56)
NAT Gateway
(IPv4)
Public Subnet
(IPv4: 10.0.0.0/24)
(IPv6: 2001:db8:1234:1a00::/64)
Private IPv4: 10.0.0.5
EIP: 198.51.100.1
IPv6: 2001:db8:1234:1a00::123
Target
10.0.0.0/16
local
2001:db8:1234:1a00::/56
local
0.0.0.0/0
igw-id
::/0
igw-id
EIP: 198.51.100.1
Web server
Private Subnet
(IPv4: 10.0.1.0/24)
(IPv6: 2001:db8:1234:1a02::/64)
Internet
Gateway
(IPv4 & IPv6)
Internet
Route Table
Egress-only
Internet Gateway
(IPv6)
Private IPv4: 10.0.1.5
IPv6: 2001:db8:1234:1a02::456
Server
Destination
(Private Subnet)
Destination
Target
10.0.0.0/16
local
2001:db8:1234:1a00::/56
local
0.0.0.0/0
nat-gateway-id
::/0
eigw-id

<!-- Page 765 -->

# VPC Section Summary (1/3)

- CIDR – IP Range
- VPC – Virtual Private Cloud => we define a list of IPv4 & IPv6 CIDR
- Subnets – tied to an AZ, we define a CIDR
- Internet Gateway – at the VPC level, provide IPv4 & IPv6 Internet Access
- Route Tables – must be edited to add routes from subnets to the IGW, VPC Peering
Connections, VPC Endpoints, …
- Bastion Host – public EC2 instance to SSH into, that has SSH connectivity to EC2
instances in private subnets
- NAT Instances – gives Internet access to EC2 instances in private subnets. Old, must
be setup in a public subnet, disable Source / Destination check flag
- NAT Gateway – managed by AWS, provides scalable Internet access to private EC2
instances, when the target is an IPv4 address

<!-- Page 766 -->

# VPC Section Summary (2/3)

- NACL – stateless, subnet rules for inbound and outbound, don’t forget Ephemeral
Ports
- Security Groups – stateful, operate at the EC2 instance level
- VPC Peering – connect two VPCs with non overlapping CIDR, non-transitive
- VPC Endpoints – provide private access to AWS Services (S3, DynamoDB,
CloudFormation, SSM) within a VPC
- VPC Flow Logs – can be setup at the VPC / Subnet / ENI Level, for ACCEPT and
REJECT traffic, helps identifying attacks, analyze using Athena or CloudWatch Logs
Insights
- Site-to-Site VPN – setup a Customer Gateway on DC, a Virtual Private Gateway on
VPC, and site-to-site VPN over public Internet
- AWS VPN CloudHub – hub-and-spoke VPN model to connect your sites

<!-- Page 767 -->

# VPC Section Summary (3/3)

- Direct Connect – setup a Virtual Private Gateway on VPC, and establish a
direct private connection to an AWS Direct Connect Location
- Direct Connect Gateway – setup a Direct Connect to many VPCs in different
AWS regions
- AWS PrivateLink / VPC Endpoint Services:
- Connect services privately from your service VPC to customers VPC
- Doesn’t need VPC Peering, public Internet, NAT Gateway, Route Tables
- Must be used with Network Load Balancer & ENI
- ClassicLink – connect EC2-Classic EC2 instances privately to your VPC
- Transit Gateway – transitive peering connections for VPC, VPN & DX
- Traffic Mirroring – copy network traffic from ENIs for further analysis
- Egress-only Internet Gateway – like a NAT Gateway, but for IPv6 targets

<!-- Page 768 -->

# Networking Costs in AWS per GB - Simplified

Region
Region
Availability Zone
Availability Zone
Availability Zone
Free for traffic in
Free if using
private IP
$0.01 if
Using private IP
$0.02 if using
Public IP / Elastic IP
$0.02
Inter-region
- Use Private IP
instead of Public
IP for good
savings and
better network
performance
- Use same AZ for
maximum savings
(at the cost of
high availability)

<!-- Page 769 -->

# Minimizing egress traffic network cost

- Egress traffic: outbound
traffic (from AWS to
outside)
- Ingress traffic: inbound
traffic - from outside to
AWS (typically free)
- Try to keep as much
internet traffic within
AWS to minimize costs
- Direct Connect location
that are co-located in the
same AWS Region result
in lower cost for egress
network
Egress cost is high
AWS Cloud
DB Query
100 MB
Query Results
50 KB
Application
Database
Egress cost is minimized
AWS Cloud
Corporate data center
DB Query
100 MB
Database
Corporate data center
Application
Query Results
50 KB

<!-- Page 770 -->

# S3 Data Transfer Pricing – Analysis for USA

- S3 ingress: free
- S3 to Internet: $0.09 per GB
- S3 Transfer Acceleration:
- Faster transfer times (50 to 500% better)
- Additional cost on top of Data Transfer
Pricing: +$0.04 to $0.08 per GB
- S3 to CloudFront: $0.00 per GB
- CloudFront to Internet: $0.085 per GB
(slightly cheaper than S3)
- Caching capability (lower latency)
- Reduce costs associated with S3 Requests
Pricing (7x cheaper with CloudFront)
- S3 Cross Region Replication: $0.02 per GB
internet
$0.09
Transfer acceleration +$0.04
Edge location
$0.00
Replication $0.02
$0.085
CloudFront

<!-- Page 771 -->

# NAT Gateway vs Gateway VPC Endpoint

Region
(us-east-1)
Subnet 1 route table
Destination
Target
10.0.0.0/16
Local
0.0.0.0/0
igw-id
Subnet 2 route table
Destination
Target
10.0.0.0/16
Local
pl-id for
Amazon S3
vpce-id
Private subnet 1
(10.0.0.0/24)
Public subnet
EC2 Instance
NAT Gateway
Internet
Gateway
Internet
Private subnet 2
(10.0.1.0/24)
EC2 Instance
$0.045 NAT Gateway / hour
$0.045 NAT Gateway data processed / GB
$0.09 Data transfer out to S3 (cross-region)
$0.00 Data transfer out to S3 (same-region)
VPC
(10.0.0.0/16)
VPC Endpoint
S3 Bucket
No cost for using Gateway Endpoint.
$0.01 Data transfer in/out (sameregion)
Pricing:

<!-- Page 772 -->

# Network Protection on AWS

- To protect network on AWS, we’ve seen
- Network Access Control Lists (NACLs)
- Amazon VPC security groups
- AWS WAF (protect against malicious requests)
- AWS Shield & AWS Shield Advanced
- AWS Firewall Manager (to manage them across accounts)
- But what if we want to protect in a sophisticated way our entire VPC?

<!-- Page 773 -->

# AWS Network Firewall

internet
- Protect your entire Amazon VPC
- From Layer 3 to Layer 7 protection
- Any direction, you can inspect
- VPC to VPC traffic
- Outbound to internet
- Inbound from internet
- To / from Direct Connect & Site-to-Site VPN
VPC
Direct Connect
Private subnet
Corporate DC
- Internally, the AWS Network Firewall uses
the AWS Gateway Load Balancer
- Rules can be centrally managed crossaccount by AWS Firewall Manager to apply
to many VPCs
VPN connection
Peered VPC

<!-- Page 774 -->

# Network Firewall – Fine Grained Controls

- Supports 1000s of rules
- IP & port - example: 10,000s of IPs filtering
- Protocol – example: block the SMB protocol for outbound communications
- Stateful domain list rule groups: only allow outbound traffic to *.mycorp.com or third-party
software repo
- General pattern matching using regex
- Traffic filtering: Allow, drop, or alert for the traffic that matches the rules
- Active flow inspection to protect against network threats with intrusion-prevention
capabilities (like Gateway Load Balancer, but all managed by AWS)
- Send logs of rule matches to Amazon S3, CloudWatch Logs, Kinesis Data Firehose

<!-- Page 775 -->

# Disaster Recovery & Migrations

<!-- Page 776 -->

# Disaster Recovery Overview

- Any event that has a negative impact on a company’s business continuity
or finances is a disaster
- Disaster recovery (DR) is about preparing for and recovering from a
disaster
- What kind of disaster recovery?
- On-premise => On-premise: traditional DR, and very expensive
- On-premise => AWS Cloud: hybrid recovery
- AWS Cloud Region A => AWS Cloud Region B
- Need to define two terms:
- RPO: Recovery Point Objective
- RTO: Recovery Time Objective

<!-- Page 777 -->

# RPO and RTO

Data loss
RPO
Downtime
Disaster
RTO

<!-- Page 778 -->

# Disaster Recovery Strategies

- Backup and Restore
- Pilot Light
- Warm Standby
- Hot Site / Multi Site Approach
Faster RTO

<!-- Page 779 -->

# Backup and Restore (High RPO)

Corporate data
center
AWS Cloud
AWS Cloud
Amazon EC2
Amazon S3
AWS Snowball
Glacier
lifecycle
AWS Storage Gateway
AMI
AWS Cloud
EBS
Scheduled regular
snapshots
Redshift
Snapshot
RDS
Amazon RDS

<!-- Page 780 -->

# Disaster Recovery – Pilot Light

- A small version of the app is always running in the cloud
- Useful for the critical core (pilot light)
- Very similar to Backup and Restore
- Faster than Backup and Restore as critical systems are already up
Corporate data
center
Route 53
AWS Cloud
EC2 (not running)
Data Replication
RDS (running)

<!-- Page 781 -->

# Warm Standby

- Full system is up and running, but at minimum size
- Upon disaster, we can scale to production load
Corporate data
center
Reverse
proxy
AWS Cloud
Route 53
ELB
App
Server
Primary
DB
EC2 Auto Scaling
(minimum)
Data Replication
RDS Secondary (running)
failover

<!-- Page 782 -->

# Multi Site / Hot Site Approach

- Very low RTO (minutes or seconds) – very expensive
- Full Production Scale is running AWS and On Premise
Corporate data
center
Reverse
proxy
active
active
AWS Cloud
Route 53
ELB
App
Server
Primary
DB
EC2 Auto Scaling
(production)
Data Replication
RDS secondary (running)
failover

<!-- Page 783 -->

# All AWS Multi Region

AWS Cloud
active
active
AWS Cloud
Route 53
ELB
ELB
EC2 Auto Scaling
(production)
EC2 Auto Scaling
(production)
Aurora Global (primary)
Data Replication
Aurora Global (secondary)
failover

<!-- Page 784 -->

# Disaster Recovery Tips

- Backup
- 
- 
- 
EBS Snapshots, RDS automated backups / Snapshots, etc…
Regular pushes to S3 / S3 IA / Glacier, Lifecycle Policy, Cross Region Replication
From On-Premise: Snowball or Storage Gateway
- High Availability
- 
- 
- 
Use Route53 to migrate DNS over from Region to Region
RDS Multi-AZ, ElastiCache Multi-AZ, EFS, S3
Site to Site VPN as a recovery from Direct Connect
- Replication
- 
- 
- 
RDS Replication (Cross Region), AWS Aurora + Global Databases
Database replication from on-premises to RDS
Storage Gateway
- Automation
- 
- 
- 
CloudFormation / Elastic Beanstalk to re-create a whole new environment
Recover / Reboot EC2 instances with CloudWatch if alarms fail
AWS Lambda functions for customized automations
- Chaos
- 
Netflix has a “simian-army” randomly terminating EC2

<!-- Page 785 -->

# AWS Elastic Disaster Recovery (DRS)

- Used to be named “CloudEndure Disaster Recovery”
- Quickly and easily recover your physical, virtual, and cloud-based servers into AWS
- Example: protect your most critical databases (including Oracle, MySQL, and SQL Server),
enterprise apps (SAP), protect your data from ransomware attacks, …
- Continuous block-level replication for your servers
Corporate Data Center / Any cloud
AWS Cloud
Elastic Disaster Recovery
OS
Staging
continuous replication
(seconds)
Apps
DB
Disks
AWS Replication
Agent
failover
(minutes)
Low-cost EC2 instances
& EBS volumes
failback
Production
Target EC2 instances
& EBS volumes

<!-- Page 786 -->

# DMS – Database Migration Service

- Quickly and securely migrate databases to
AWS, resilient, self healing
- The source database remains available
during the migration
- Supports:
- Homogeneous migrations: ex Oracle to
Oracle
- Heterogeneous migrations: ex Microsoft SQL
Server to Aurora
- Continuous Data Replication using CDC
- You must create an EC2 instance to
perform the replication tasks
Source DB
EC2 instance
Running DMS
Target DB

<!-- Page 787 -->

# DMS Sources and Targets

SOURCES:
- On-Premises and EC2 instances
databases: Oracle, MS SQL Server,
MySQL, MariaDB, PostgreSQL,
MongoDB, SAP, DB2
- Azure: Azure SQL Database
- Amazon RDS: all including
Aurora
- Amazon S3
- DocumentDB
TARGETS:
- On-Premises and EC2 instances
databases: Oracle, MS SQL Server,
MySQL, MariaDB, PostgreSQL, SAP
- Amazon RDS
- Redshift, DynamoDB, S3
- OpenSearch Service
- Kinesis Data Streams
- Apache Kafka
- DocumentDB & Amazon Neptune
- Redis & Babelfish

<!-- Page 788 -->

# AWS Schema Conversion Tool (SCT)

- Convert your Database’s Schema from one engine to another
- Example OLTP: (SQL Server or Oracle) to MySQL, PostgreSQL, Aurora
- Example OLAP: (Teradata or Oracle) to Amazon Redshift
- Prefer compute-intensive instances to optimize data conversions
Source DB
DMS + SCT
Target DB (different engine)
- You do not need to use SCT if you are migrating the same DB engine
- Ex: On-Premise PostgreSQL => RDS PostgreSQL
- The DB engine is still PostgreSQL (RDS is the platform)

<!-- Page 789 -->

# DMS - Continuous Replication

Region
Corporate data center
VPC
Public Subnet
Private Subnet
Data migration
Full load +
CDC
Oracle DB
(source)
Schema conversion
Server with
AWS SCT installed
AWS DMS
Replication
Instance
Amazon RDS
for MySQL DB
(target)

<!-- Page 790 -->

# AWS DMS – Multi-AZ Deployment

- When Multi-AZ Enabled, DMS
provisions and maintains a
synchronously stand replica in a
different AZ
AWS Region
Availability Zone - A
- Advantages:
- Provides Data Redundancy
- Eliminates I/O freezes
- Minimizes latency spikes
Availability Zone - B
synchronous
replication
DMS Replication
Instance
DMS Replication
Instance
(Standby Replica)

<!-- Page 791 -->

# RDS & Aurora MySQL Migrations

- RDS MySQL to Aurora MySQL
- Option 1: DB Snapshots from RDS MySQL restored as
MySQL Aurora DB
- Option 2: Create an Aurora Read Replica from your RDS
MySQL, and when the replication lag is 0, promote it as its
own DB cluster (can take time and cost $)
- External MySQL to Aurora MySQL
- Option 1:
- Use Percona XtraBackup to create a file backup in Amazon S3
- Create an Aurora MySQL DB from Amazon S3
Read Replica
Percona
XtraBackup
import
- Option 2:
- Create an Aurora MySQL DB
- Use the mysqldump utility to migrate MySQL into Aurora
(slower than S3 method)
- Use DMS if both databases are up and running
mysqldump

<!-- Page 792 -->

# RDS & Aurora PostgreSQL Migrations

- RDS PostgreSQL to Aurora PostgreSQL
- Option 1: DB Snapshots from RDS PostgreSQL restored
as PostgreSQL Aurora DB
- Option 2: Create an Aurora Read Replica from your RDS
PostgreSQL, and when the replication lag is 0, promote it
as its own DB cluster (can take time and cost $)
- External PostgreSQL to Aurora PostgreSQL
- Create a backup and put it in Amazon S3
- Import it using the aws_s3 Aurora extension
- Use DMS if both databases are up and running
Read Replica
backup
import

<!-- Page 793 -->

# On-Premise strategy with AWS

- Ability to download Amazon Linux 2 AMI as a VM (.iso format)
- VMWare, KVM, VirtualBox (Oracle VM), Microsoft Hyper-V
- VM Import / Export
- Migrate existing applications into EC2
- Create a DR repository strategy for your on-premises VMs
- Can export back the VMs from EC2 to on-premises
- AWS Application Discovery Service
- Gather information about your on-premises servers to plan a migration
- Server utilization and dependency mappings
- Track with AWS Migration Hub
- AWS Database Migration Service (DMS)
- replicate On-premise => AWS , AWS => AWS, AWS => On-premise
- Works with various database technologies (Oracle, MySQL, DynamoDB, etc..)
- AWS Application Migration Service (MGN)
- Incremental replication of on-premises live servers to AWS

<!-- Page 794 -->

# AWS Backup

- Fully managed service
- Centrally manage and automate backups across AWS services
- No need to create custom scripts and manual processes
- Supported services:
- Amazon EC2 / Amazon EBS
- Amazon S3
- Amazon RDS (all DBs engines) / Amazon Aurora / Amazon DynamoDB
- Amazon DocumentDB / Amazon Neptune
- Amazon EFS / Amazon FSx (Lustre & Windows File Server)
- AWS Storage Gateway (Volume Gateway)
- Supports cross-region backups
- Supports cross-account backups

<!-- Page 795 -->

# AWS Backup

- Supports PITR for supported services
- On-Demand and Scheduled backups
- Tag-based backup policies
- You create backup policies known as Backup Plans
- Backup frequency (every 12 hours, daily, weekly, monthly, cron expression)
- Backup window
- Transition to Cold Storage (Never, Days, Weeks, Months, Years)
- Retention Period (Always, Days, Weeks, Months, Years)

<!-- Page 796 -->

# AWS Backup

Assign AWS Resources
EC2
Create Backup Plan
(frequency, retention
policy)
RDS
EBS
S3
DynamoDB DocumentDB
Automatically
backed up to
Amazon S3
EFS
Aurora
FSx
Storage
Gateway
Neptune

<!-- Page 797 -->

# AWS Backup Vault Lock

- Enforce a WORM (Write Once Read Many)
state for all the backups that you store in
your AWS Backup Vault
backup
- Additional layer of defense to protect your
backups against:
- Inadvertent or malicious delete operations
- Updates that shorten or alter retention periods
- Even the root user cannot delete backups
when enabled
Backup Vault Lock Policy
Backups can’t be deleted

<!-- Page 798 -->

# AWS Application Discovery Service

- Plan migration projects by gathering information about on-premises data centers
- Server utilization data and dependency mapping are important for migrations
- Agentless Discovery (AWS Agentless Discovery Connector)
- VM inventory, configuration, and performance history such as CPU, memory, and disk usage
- Agent-based Discovery (AWS Application Discovery Agent)
- System configuration, system performance, running processes, and details of the network
connections between systems
- Resulting data can be viewed within AWS Migration Hub

<!-- Page 799 -->

# AWS Application Migration Service (MGN)

- The “AWS evolution” of CloudEndure Migration, replacing AWS Server Migration Service (SMS)
- Lift-and-shift (rehost) solution which simplify migrating applications to AWS
- Converts your physical, virtual, and cloud-based servers to run natively on AWS
- Supports wide range of platforms, Operating Systems, and databases
- Minimal downtime, reduced costs
Corporate Data Center / Any cloud
AWS Cloud
Application Migration Service
OS
continuous replication
Apps
Production
cutover
DB
Disks
Staging
AWS Replication
Agent
Low-cost EC2 instances
& EBS volumes
Target EC2 instances
& EBS volumes

<!-- Page 800 -->

# VMware Cloud on AWS

- Some customers use VMware Cloud to manage their on-premises Data Center
- They want to extend the Data Center capacity to AWS, but keep using the VMware Cloud software
- …Enter VMware Cloud on AWS
- Use cases
- Migrate your VMware vSphere-based workloads to AWS
- Run your production workloads across VMware vSphere-based private, public, and hybrid cloud environments
- Have a disaster recover strategy
Customer Data Center
AWS Cloud
AWS Services
On-Premises vCenter
vSphere-based
environment
VMware Cloud
on AWS
vSphere
Amazon
EC2
Amazon
S3
Direct
Connect
Amazon
FSx
Amazon
RDS
Amazon
Redshift

<!-- Page 801 -->

# Transferring large amount of data into AWS

- Example: transfer 200 TB of data in the cloud. We have a 100 Mbps internet
connection.
- Over the internet / Site-to-Site VPN:
- Immediate to setup
- Will take 200(TB)*1000(GB)*1000(MB)*8(Mb)/100 Mbps = 16,000,000s = 185d
- Over direct connect 1Gbps:
- Long for the one-time setup (over a month)
- Will take 200(TB)*1000(GB)*8(Gb)/1 Gbps = 1,600,000s = 18.5d
- Over Snowball:
- Takes about 1 week for the end-to-end transfer
- Can be combined with DMS
- For on-going replication / transfers: Site-to-Site VPN or DX with DMS or
DataSync

<!-- Page 802 -->

# More Solutions Architecture

<!-- Page 803 -->

# Lambda, SNS & SQS

Try, retry
retries
(poll)
SQS
asynchronous
DLQ
SQS + Lambda
SNS
Try, retry
blocking
SQS FIFO
DLQ
SQS
SNS + Lambda
SQS FIFO + Lambda
DLQ

<!-- Page 804 -->

# Fan Out Pattern: deliver to multiple SQS

SQS
SDK
PUT #1
PUT #2
SQS
SDK
SNS
PUT
subscribe
PUT #3
Option 1
Option 2 – Fan Out

<!-- Page 805 -->

# S3 Event Notifications

- S3:ObjectCreated, S3:ObjectRemoved,
S3:ObjectRestore, S3:Replication…
- Object name filtering possible (*.jpg)
- Use case: generate thumbnails of images
uploaded to S3
- Can create as many “S3 events” as desired
SNS
events
Amazon S3
- S3 event notifications typically deliver events
in seconds but can sometimes take a minute
or longer
SQS
Lambda Function

<!-- Page 806 -->

# with Amazon EventBridge

events
All events
Amazon S3
bucket
rules
Amazon
EventBridge
Over 18
AWS services
as destinations
- Advanced filtering options with JSON rules (metadata, object size, name...)
- Multiple Destinations – ex Step Functions, Kinesis Streams / Firehose…
- EventBridge Capabilities – Archive, Replay Events, Reliable delivery
S3 Event Notifications

<!-- Page 807 -->

# Amazon EventBridge – Intercept API Calls

User
DeleteTable API Call 💥
Log API call
DynamoDB
alert
event
CloudTrail
(any API call)
Amazon
EventBridge
SNS

<!-- Page 808 -->

# Kinesis Data Streams example

requests
Client
records
send
API Gateway
store .json
files
Kinesis Data
Streams
Kinesis Data
Firehose
Amazon S3
API Gateway – AWS Service Integration

<!-- Page 809 -->

# Caching Strategies

Redis
Memcached
DAX
Client
CloudFront
API Gateway
App logic
EC2 / Lambda
Database
S3
CloudFront (edge)
Caching, TTL, Network, Computation, Cost, Latency

<!-- Page 810 -->

# Blocking an IP address

VPC
Public Subnet
Security Group (allow rules)
Client
NACL
Deny + Allow rules
EC2 Instance
public IP + Firewall Software (optional)

<!-- Page 811 -->

# Blocking an IP address – with an ALB

VPC
Public Subnet
Client
NACL
Private Subnet
ALB Security Group
EC2 Security Group
Application Load Balancer
EC2 Instance
Connection Termination
Private IP

<!-- Page 812 -->

# Blocking an IP address – with an NLB

VPC
Public Subnet
Client
NACL
Private Subnet
NLB Security Group
EC2 Security Group
Network Load Balancer
EC2 Instance
Private IP

<!-- Page 813 -->

# Blocking an IP address – ALB + WAF

VPC
Public Subnet
Client
NACL
Private Subnet
ALB Security Group
EC2 Security Group
Application Load Balancer
EC2 Instance
Private IP
IP Address Filtering
AWS WAF

<!-- Page 814 -->

# Blocking an IP address – ALB, CloudFront & WAF

VPC
Public Subnet
ALB Security Group
CloudFront
Public IPs
Client
NACL
Application Load Balancer
Public
NOT helpful
CloudFront
(Geo Restriction)
AWS WAF
(IP Address Filtering)
Private Subnet
EC2 Security Group
EC2 Instance
Private IP

<!-- Page 815 -->

# High Performance Computing (HPC)

- The cloud is the perfect place to perform HPC
- You can create a very high number of resources in no time
- You can speed up time to results by adding more resources
- You can pay only for the systems you have used
- Perform genomics, computational chemistry, financial risk modeling,
weather prediction, machine learning, deep learning, autonomous driving
- Which services help perform HPC?

<!-- Page 816 -->

# Data Management & Transfer

- AWS Direct Connect:
- Move GB/s of data to the cloud, over a private secure network
- Snowball & Snowmobile
- Move PB of data to the cloud
- AWS DataSync
- Move large amount of data between on-premises and S3, EFS, FSx for Windows

<!-- Page 817 -->

# Compute and Networking

- EC2 Instances:
- CPU optimized, GPU optimized
- Spot Instances / Spot Fleets for cost savings + Auto Scaling
- EC2 Placement Groups: Cluster for good network performance
EC2
EC2
EC2
EC2
EC2
EC2
Same Rack
Same AZ
Placement group
Cluster
Low latency
10Gbps network

<!-- Page 818 -->

# Compute and Networking

- EC2 Enhanced Networking (SR-IOV)
- Higher bandwidth, higher PPS (packet per second), lower latency
- Option 1: Elastic Network Adapter (ENA) up to 100 Gbps
- Option 2: Intel 82599 VF up to 10 Gbps – LEGACY
- Elastic Fabric Adapter (EFA)
- Improved ENA for HPC, only works for Linux
- Great for inter-node communications, tightly coupled workloads
- Leverages Message Passing Interface (MPI) standard
- Bypasses the underlying Linux OS to provide low-latency, reliable transport

<!-- Page 819 -->

# Storage

- Instance-attached storage:
- EBS: scale up to 256,000 IOPS with io2 Block Express
- Instance Store: scale to millions of IOPS, linked to EC2 instance, low latency
- Network storage:
- Amazon S3: large blob, not a file system
- Amazon EFS: scale IOPS based on total size, or use provisioned IOPS
- Amazon FSx for Lustre:
- HPC optimized distributed file system, millions of IOPS
- Backed by S3

<!-- Page 820 -->

# Automation and Orchestration

- AWS Batch
- AWS Batch supports multi-node parallel jobs, which enables you to run single
jobs that span multiple EC2 instances.
- Easily schedule jobs and launch EC2 instances accordingly
- AWS ParallelCluster
- Open-source cluster management tool to deploy HPC on AWS
- Configure with text files
- Automate creation of VPC, Subnet, cluster type and instance types
- Ability to enable EFA on the cluster (improves network performance)

<!-- Page 821 -->

# Creating a highly available EC2 instance

monitor
Attachment
What time is it?
5:30 pm!
Public EC2
CloudWatch Event
(or Alarm based on metric)
Elastic IP Address
Start the instance
Attach the Elastic IP
Standby EC2 instance

<!-- Page 822 -->

# With an Auto Scaling Group

Auto Scaling group
Availability Zone 1
What time is it?
Public EC2
5:30 pm!
Elastic IP Address
Availability Zone 2
EC2 User Data
Attachment
Based on Tag
Replacement
EC2 instance
ASG Settings
1 min
1 max
1 desired
>= 2 AZ
EC2 user data to attach
The Elastic IP
EC2 instance role to
Allow API calls to attach
The Elastic IP
Creating a highly available EC2 instance

<!-- Page 823 -->

# With ASG + EBS

Auto Scaling group
Availability Zone 1
EBS Snapshot
On ASG Terminate lifecycle hook
What time is it?
Public EC2 EBS Volume
5:30 pm!
Elastic IP Address
EC2 User Data
Attachment
Based on Tag
Availability Zone 2
EBS
Replacement
EC2 instance
EBS Snapshot
+ tags
EBS Volume created + attached
On ASG Launch lifecycle hook
Creating a highly available EC2 instance

<!-- Page 824 -->

# Other Services

Overview of Services that might come up in a few questions

<!-- Page 825 -->

# What is CloudFormation

- CloudFormation is a declarative way of outlining your AWS
Infrastructure, for any resources (most of them are supported).
- For example, within a CloudFormation template, you say:
- I want a security group
- I want two EC2 instances using this security group
- I want an S3 bucket
- I want a load balancer (ELB) in front of these machines
- Then CloudFormation creates those for you, in the right order, with the
exact configuration that you specify

<!-- Page 826 -->

# Benefits of AWS CloudFormation (1/2)

- Infrastructure as code
- No resources are manually created, which is excellent for control
- Changes to the infrastructure are reviewed through code
- Cost
- Each resources within the stack is tagged with an identifier so you can easily see how
much a stack costs you
- You can estimate the costs of your resources using the CloudFormation template
- Savings strategy: In Dev, you could automation deletion of templates at 5 PM and
recreated at 8 AM, safely

<!-- Page 827 -->

# Benefits of AWS CloudFormation (2/2)

- Productivity
- Ability to destroy and re-create an infrastructure on the cloud on the fly
- Automated generation of Diagram for your templates!
- Declarative programming (no need to figure out ordering and orchestration)
- Don’t re-invent the wheel
- Leverage existing templates on the web!
- Leverage the documentation
- Supports (almost) all AWS resources:
- Everything we’ll see in this course is supported
- You can use “custom resources” for resources that are not supported

<!-- Page 828 -->

# Composer

- Example: WordPress CloudFormation Stack
- We can see all the resources
- We can see the relations between the components
+
CloudFormation
CloudFormation + Infrastructure Composer
Infrastructure

<!-- Page 829 -->

# CloudFormation – Service Role

- IAM role that allows CloudFormation to
create/update/delete stack resources on your
behalf
- Give ability to users to create/update/delete the
stack resources even if they don’t have
permissions to work with the resources in the
stack
- Use cases:
Permissions
- cloudformation:*
- iam:PassRole
User
Template
Service Role
- s3:*Bucket
CloudFormation
- You want to achieve the least privilege principle
- But you don’t want to give the user all the required
permissions to create the stack resources
- User must have iam:PassRole permissions
S3 bucket
Stack

<!-- Page 830 -->

# Amazon Simple Email Service (Amazon SES)

- Fully managed service to send emails securely, globally and at scale
- Allows inbound/outbound emails
- Reputation dashboard, performance insights, anti-spam feedback
- Provides statistics such as email deliveries, bounces, feedback loop
results, email open
- Supports DomainKeys Identified Mail (DKIM) and Sender Policy
Framework (SPF)
- Flexible IP deployment: shared, dedicated, and customer-owned IPs
- Send emails using your application using AWS Console, APIs, or SMTP
- Use cases: transactional, marketing and bulk email communications
Users
bulk emails
Amazon SES
APIs
or SMTP
Application

<!-- Page 831 -->

# Amazon Pinpoint

- Scalable 2-way (outbound/inbound) marketing
communications service
- Supports email, SMS, push, voice, and in-app messaging
- Ability to segment and personalize messages with the
right content to customers
- Possibility to receive replies
- Scales to billions of messages per day
- Use cases: run campaigns by sending marketing, bulk,
transactional SMS messages
- Versus Amazon SNS or Amazon SES
Amazon
Pinpoint
SMS
Customers
stream events
(e.g., TEXT_SUCCESS,
TEXT_DELIVERED, …)
- In SNS & SES you managed each message's audience,
content, and delivery schedule
- In Amazon Pinpoint, you create message templates,
delivery schedules, highly-targeted segments, and full
campaigns
SNS
Kinesis Data
Firehose
CloudWatch
Logs

<!-- Page 832 -->

# Systems Manager – SSM Session Manager

- Allows you to start a secure shell on your EC2 and
on-premises servers
- No SSH access, bastion hosts, or SSH keys needed
- No port 22 needed (better security)
- Supports Linux, macOS, and Windows
- Send session log data to S3 or CloudWatch Logs
EC2 Instance
(SSM Agent)
Execute
commands
Session
Manager
IAM
Permissions
User

<!-- Page 833 -->

# Systems Manager – Run Command

Amazon SNS
Amazon S3
n
t io
ica
t if
no
- Execute a document (= script) or just run a
command
- Run command across multiple instances
(using resource groups)
- No need for SSH
- Command Output can be shown in the AWS
Console, sent to S3 bucket or CloudWatch
Logs
- Send notifications to SNS about command
status (In progress, Success, Failed, …)
- Integrated with IAM & CloudTrail
- Can be invoked using EventBridge
EventBridge
trigger
output
Run Command
CloudWatch
Logs
EC2 Instances
EC2 Instances
(with SSM Agent) (with SSM Agent)

<!-- Page 834 -->

# Systems Manager – Patch Manager

- Automates the process of patching managed
instances
- OS updates, applications updates, security
updates
- Supports EC2 instances and on-premises
servers
- Supports Linux, macOS, and Windows
- Patch on-demand or on a schedule using
Maintenance Windows
- Scan instances and generate patch compliance
report (missing patches)
AWS Console
AWS SDK Maintenance
Windows
run
AWS-RunBatchBaseline
Run Command
EC2 Instances
(with SSM Agent)
EC2 Instances
(with SSM Agent)

<!-- Page 835 -->

# Systems Manager – Maintenance Windows

- Defines a schedule for when to perform actions on your instances
- Example: OS patching, updating drivers, installing software, …
- Maintenance Window contains
- Schedule
- Duration
- Set of registered instances
- Set of registered tasks
trigger every 24 hour
Maintenance Windows
update
EC2 Instances
(with SSM Agent)
Run Command
EC2 Instances
(with SSM Agent)

<!-- Page 836 -->

# Systems Manager - Automation

- Simplifies common maintenance and
deployment tasks of EC2 instances and other
AWS resources
- Examples: restart instances, create an AMI,
EBS snapshot
- Automation Runbook – SSM Documents to
define actions preformed on your EC2
instances or AWS resources (pre-defined or
custom)
- Can be triggered using:
- Manually using AWS Console, AWS CLI or SDK
- Amazon EventBridge
- On a schedule using Maintenance Windows
- By AWS Config for rules remediations
AWS Console
AWS SDK Maintenance Amazon
AWS Config
Windows EventBridge Remediation
execute automation
(AWS-RestartEC2Instance)
Runbooks
(automation documents)
execute
AWS Resources
EC2 Instances
SSM Automation
…
EBS
AMI
RDS

<!-- Page 837 -->

# Cost Explorer

- Visualize, understand, and manage your AWS costs and usage over time
- Create custom reports that analyze cost and usage data.
- Analyze your data at a high level: total costs and usage across all accounts
- Or Monthly, hourly, resource level granularity
- Choose an optimal Savings Plan (to lower prices on your bill)
- Forecast usage up to 18 months based on previous usage

<!-- Page 838 -->

# Cost Explorer – Monthly Cost by AWS Service

<!-- Page 839 -->

# Cost Explorer– Hourly & Resource Level

<!-- Page 840 -->

# Alternative to Reserved Instances

Cost Explorer – Savings Plan

<!-- Page 841 -->

# Cost Explorer – Forecast Usage

<!-- Page 842 -->

# AWS Cost Anomaly Detection

- Continuously monitor your cost and usage using ML to detect unusual spends
- It learns your unique, historic spend patterns to detect one-time cost spike
and/or continuous cost increases (you don’t need to define thresholds)
- Monitor AWS services, member accounts, cost allocation tags, or cost categories
- Sends you the anomaly detection report with root-cause analysis
- Get notified with individual alerts or daily/weekly summary (using SNS)
AWS Cost Anomaly
Detection
reduce cost surprises
with Machine Learning
Create Cost Monitor
Identify unusual spend at
the granularity level
that you specify
Get Alerted
Receive alerts when
unusual spend is detected
Analyze Root Cause
Analyze the root cause
behind the anomaly and
the impact on your costs

<!-- Page 843 -->

# AWS Outposts

- Hybrid Cloud: businesses that keep an onpremises infrastructure alongside a cloud
infrastructure
- Therefore, two ways of dealing with IT systems:
- One for the AWS cloud (using the AWS console,
CLI, and AWS APIs)
- One for their on-premises infrastructure
- AWS Outposts are “server racks” that offers the
same AWS infrastructure, services, APIs & tools
to build your own applications on-premises just as
in the cloud
- AWS will setup and manage “Outposts Racks”
within your on-premises infrastructure and you
can start leveraging AWS services on-premises
- You are responsible for the Outposts Rack
physical security
AWS
Cloud
Corporate
data center
On-prem
servers
Extension of
AWS services
Outposts
Racks

<!-- Page 844 -->

# AWS Outposts

- Benefits:
- Low-latency access to on-premises systems
- Local data processing
- Data residency
- Easier migration from on-premises to the cloud
- Fully managed service
- Some services that work on Outposts:
Amazon EC2
Amazon EBS
Amazon S3
Amazon EKS
Amazon ECS
Amazon RDS
Amazon EMR

<!-- Page 845 -->

# AWS Batch

- Fully managed batch processing at any scale
- Efficiently run 100,000s of computing batch jobs on AWS
- A “batch” job is a job with a start and an end (opposed to continuous)
- Batch will dynamically launch EC2 instances or Spot Instances
- AWS Batch provisions the right amount of compute / memory
- You submit or schedule batch jobs and AWS Batch does the rest!
- Batch jobs are defined as Docker images and run on ECS, EKS, Fargate
- Helpful for cost optimizations and focusing less on the infrastructure

<!-- Page 846 -->

# AWS Batch – Simplified Example

AWS Batch
EC2 Instance
Amazon S3
Trigger
ECS
Spot Instance
Insert
processed object
Amazon S3

<!-- Page 847 -->

# Batch vs Lambda

- Lambda:
- Time limit
- Limited runtimes
- Limited temporary disk space
- Serverless
- Batch:
- No time limit
- Any runtime as long as it’s packaged as a Docker image
- Rely on EBS / instance store for disk space
- Relies on EC2 (can be managed by AWS)

<!-- Page 848 -->

# Amazon AppFlow

- Fully managed integration service that enables you to securely transfer
data between Software-as-a-Service (SaaS) applications and AWS
- Sources: Salesforce, SAP, Zendesk, Slack, and ServiceNow
- Destinations: AWS services like Amazon S3, Amazon Redshift or nonAWS such as SnowFlake and Salesforce
- Frequency: on a schedule, in response to events, or on demand
- Data transformation capabilities like filtering and validation
- Encrypted over the public internet or privately over AWS PrivateLink
- Don’t spend time writing the integrations and leverage APIs immediately

<!-- Page 849 -->

# Amazon AppFlow

Sources
Destinations
Amazon
AppFlow

<!-- Page 850 -->

# AWS Amplify - web and mobile applications

- A set of tools and services that helps you develop and deploy scalable full stack web and mobile
applications
- Authentication, Storage, API (REST, GraphQL), CI/CD, PubSub, Analytics, AI/ML Predictions,
Monitoring, …
- Connect your source code from GitHub, AWS CodeCommit, Bitbucket, GitLab, or upload directly
connect frontend to backend
using
Amplify Frontend Libraries
configure backend
using
Amplify CLI
Amazon S3 Amazon Cognito
build using Amplify Console
& deploy
…
Frontend
Amazon
SageMaker
…
Amplify
Console
Amazon
CloudFront
Amazon Lex
AWS
API
AppSync Gateway
Lambda DynamoDB
Amplify backend
…

<!-- Page 851 -->

# Instance Scheduler on AWS

- AWS solution deployed through CloudFormation
(not a service)
- Automatically start/stop your AWS services to reduce
costs (up to 70%)
- Example: stop company’s EC2 instances outside
business hours
- Supports EC2 instances, EC2 Auto Scaling Groups, and
RDS instances
- Schedules are managed in a DynamoDB table
- Uses resources’ tags and Lambda to stop/start
instances
- Supports cross-account and cross-region resources
- https://aws.amazon.com/solutions/implementations/ins
tance-scheduler-on-aws/

<!-- Page 852 -->

# White Papers & Architectures

Well Architected Framework, Disaster Recovery, etc…

<!-- Page 853 -->

# Section Overview

- Well Architected Framework Whitepaper
- Well Architected Tool
- AWS Trusted Advisor
- Reference architectures resources (for real-world)
- Disaster Recovery on AWS Whitepaper

<!-- Page 854 -->

# General Guiding Principles

- https://aws.amazon.com/architecture/well-architected
- Stop guessing your capacity needs
- Test systems at production scale
- Automate to make architectural experimentation easier
- Allow for evolutionary architectures
- Design based on changing requirements
- Drive architectures using data
- Improve through game days
- Simulate applications for flash sale days
Well Architected Framework

<!-- Page 855 -->

# 6 Pillars

- 1) Operational Excellence
- 2) Security
- 3) Reliability
- 4) Performance Efficiency
- 5) Cost Optimization
- 6) Sustainability
- They are not something to balance, or trade-offs, they’re a synergy
Well Architected Framework

<!-- Page 856 -->

# AWS Well-Architected Tool

- Free tool to review your architectures against the 6 pillars Well-Architected
Framework and adopt architectural best practices
- How does it work?
- Select your workload and answer questions
- Review your answers against the 6 pillars
- Obtain advice: get videos and documentations, generate a report, see the results in a dashboard
- Let’s have a look: https://console.aws.amazon.com/wellarchitected
https://aws.amazon.com/blogs/aws/new-aws-well-architected-tool-review-workloads-against-best-practices/

<!-- Page 857 -->

# Trusted Advisor

- No need to install anything – high level
AWS account assessment
- Analyze your AWS accounts and provides
recommendation on 6 categories:
- Cost optimization
- Performance
- Security
- Fault tolerance
- Service limits
- Operational Excellence
- Business & Enterprise Support plan
- Full Set of Checks
- Programmatic Access using AWS Support API

<!-- Page 858 -->

# More Architecture Examples

- We’ve explored the most important architectural patterns:
- Classic: EC2, ELB, RDS, ElastiCache, etc…
- Serverless: S3, Lambda, DynamoDB, CloudFront, API Gateway, etc…
- If you want to see more AWS architectures:
- https://aws.amazon.com/architecture/
- https://aws.amazon.com/solutions/

<!-- Page 859 -->

# Exam Review & Tips

<!-- Page 860 -->

# State of learning checkpoint

- Let’s look how far we’ve gone on our learning journey
- https://aws.amazon.com/certification/certified-solutions-architectassociate/

<!-- Page 861 -->

# Practice makes perfect

- If you’re new to AWS, take a bit of AWS practice thanks to this course
before rushing to the exam
- The exam recommends you to have one or more years of hands-on
experience on AWS
- Practice makes perfect!
- If you feel overwhelmed by the amount of knowledge you just learned,
just go through it one more time

<!-- Page 862 -->

# Proceed by elimination

- Most questions are going to be scenario based
- For all the questions, rule out answers that you know for sure are wrong
- For the remaining answers, understand which one makes the most sense
- There are very few trick questions
- Don’t over-think it
- If a solution seems feasible but highly complicated, it’s probably wrong

<!-- Page 863 -->

# Skim the AWS Whitepapers

- You can read about some AWS White Papers here:
- Architecting for the Cloud: AWS Best Practices
- AWS Well-Architected Framework
- AWS Disaster Recovery (https://aws.amazon.com/disaster-recovery/)
- Overall we’ve explored all the most important concepts in the course
- It’s never bad to have a look at the whitepapers you think are
interesting!

<!-- Page 864 -->

# Read each service’s FAQ

- FAQ = Frequently asked questions
- Example: https://aws.amazon.com/vpc/faqs/
- FAQ cover a lot of the questions asked at the exam
- They help confirm your understanding of a service

<!-- Page 865 -->

# Get into the AWS Community

- Help out and discuss with other people in the course Q&A
- Review questions asked by other people in the Q&A
- Do the practice test in this section
- Read forums online
- Read online blogs
- Attend local meetups and discuss with other AWS engineers
- Watch re-invent videos on Youtube (AWS Conference)

<!-- Page 866 -->

# How will the exam work?

- You’ll have to register online at https://www.aws.training/
- Fee for the exam is 150 USD
- Provide one identity documents (ID, Passport, details are in emails sent to you…)
- No notes are allowed, no pen is allowed, no speaking
- 65 questions will be asked in 130 minutes
- Use the “Flag” feature to mark questions you want to re-visit
- At the end you can optionally review all the questions / answers
- To pass you need a score of a least 720 out of 1000
- You will know within 5 days if you passed / failed the exams (most of the time less)
- You will know the overall score a few days later (email notification)
- You will not know which answers were right / wrong
- If you fail, you can retake the exam again 14 days later

<!-- Page 867 -->

# Your AWS Certification journey

Foundational
Professional
Associate
Specialty
Knowledge-based certification for
foundational understanding of AWS Cloud.
No prior experience needed.
Role-based certifications that showcase your knowledge
and skills on AWS and build your credibility as an AWS Cloud professional.
Prior cloud and/or strong on-premises IT experience recommended.
Role-based certifications that validate advanced skills
and knowledge required to design secure, optimized,
and modernized applications and to automate processes on AWS.
2 years of prior AWS Cloud experience recommended.
Dive deeper and position yourself as a trusted advisor to your
stakeholders and/or customers in these strategic areas.
Refer to the exam guides on the exam pages for recommended experience.

<!-- Page 868 -->

# AWS Certification Paths – Architecture

Architecture
Solutions Architect
Design, develop, and manage
cloud infrastructure and assets,
work with DevOps to migrate
applications to the cloud
optional for IT/
cloud professionals
recommended for IT/cloud
professionals to leverage AI
Dive Deep
Architecture
Application Architect
Design significant aspects of
application architecture including
user interface, middleware, and
infrastructure, and ensure
enterprise-wide scalable, reliable,
and manageable systems
optional for IT/
cloud professionals
recommended for IT/cloud
professionals to leverage AI
Dive Deep
https://d1.awsstatic.com/training-andcertification/docs/AWS_certification_paths.pdf

<!-- Page 869 -->

# AWS Certification Paths – Operations

Operations
Systems Administrator
Install, upgrade, and maintain
computer components and
software, and integrate
automation processes
optional for IT/
cloud professionals
Dive Deep
Operations
Cloud Engineer
Implement and operate an
organization’s networked computing
infrastructure and Implement
security systems to maintain
data safety
optional for IT/
cloud professionals
Dive Deep

<!-- Page 870 -->

# AWS Certification Paths – DevOps

DevOps
Test Engineer
Embed testing and quality
best practices for software
development from design to release,
throughout the product life cycle
DevOps
optional for IT/
cloud professionals
Cloud DevOps Engineer
Design, deployment, and operations
of large-scale global hybrid
cloud computing environment,
advocating for end-to-end
automated CI/CD DevOps pipelines
DevOps
optional for IT/
cloud professionals
Optional
DevSecOps Engineer
Accelerate enterprise cloud adoption
while enabling rapid and stable delivery
of capabilities using CI/CD principles,
methodologies, and technologies
optional for IT/
cloud professionals
recommended for IT/cloud
professionals working on
AI/ML projects
recommended for IT/cloud
professionals working on
AI/ML projects
Dive Deep

<!-- Page 871 -->

# AWS Certification Paths – Security

Security
Cloud Security Engineer
Design computer security architecture
and develop detailed cyber security designs.
Develop, execute, and track performance
of security measures to protect information
optional for IT/
cloud professionals
recommended for IT/cloud
professionals to secure
AI/ML systems
Dive Deep
Security
Cloud Security Architect
Design and implement enterprise cloud
solutions applying governance to identify,
communicate, and minimize business and
technical risks
optional for IT/
cloud professionals
recommended for IT/cloud
professionals to secure
AI/ML systems
Dive Deep

<!-- Page 872 -->

# Networking

Development
Software Development Engineer
Develop, construct, and maintain
software across platforms and devices
optional for IT/
cloud professionals
recommended for IT/cloud
professionals to leverage AI
Network Engineer
Design and implement computer
and information networks, such as
local area networks (LAN),
wide area networks (WAN),
intranets, extranets, etc.
optional for IT/
cloud professionals
Dive Deep
AWS Certification Paths – Development &

<!-- Page 873 -->

# AI/ML

Data Analytics
Cloud Data Engineer
Automate collection and processing
of structured/semi-structured data
and monitor data pipeline performance
optional for IT/
cloud professionals
recommended for IT/cloud
professionals working on
AI/ML projects
Dive Deep
Machine Learning Engineer
Research, build, and design artificial
intelligence (AI) systems to automate
predictive models, and design machine
learning systems, models, and schemes
optional for IT/
cloud professionals
optional for AI/ML
professionals
Dive Deep
AWS Certification Paths – Data Analytics &

<!-- Page 874 -->

# AWS Certification Paths – AI/ML

AI/ML
Prompt Engineer
Design, test, and refine text
prompts to optimize the
performance of AI language models
AI/ML
Dive Deep
optional for IT/
cloud professionals
Machine Learning Ops Engineer
Build and maintain AI and ML platforms
and infrastructure. Design, implement,
and operationally support AI/ML model
activity and deployment infrastructure
AI/ML
optional for IT/
cloud professionals
optional for AI/ML
professionals
Data Scientist
Develop and maintain AI/ML models
to solve business problems. Train and
fine tune models and evaluate
their performance
optional for IT/
cloud professionals
optional for AI/ML
professionals

<!-- Page 875 -->

# Congratulations!

<!-- Page 876 -->

# Congratulations!

- Congrats on finishing the course!
- I hope you will pass the exam without a hitch J
- If you haven’t done so yet, I’d love a review from you!
- If you passed, I’ll be more than happy to know I’ve helped
- Post it in the Q&A to help & motivate other students. Share your tips!
- Post it on LinkedIn and tag me!
- Overall, I hope you learned how to use AWS and that you will be a
tremendously good AWS Solutions Architect
