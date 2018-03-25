Piotr Szczepanski 

# Three Tier AWS Infrastructure
### Basic Theory and setup notes

References:
- Sai Kiran Rathan

- https://www.udemy.com/setup-aws-infrastructure-for-production-learn-terraform/learn/v4/overview

#  Architecture 

![alt text](https://github.com/szczepanski/cloud-aws/blob/master/dev-1-architecture/architecture%20design.png "Logo Title Text 1")


- Region - geographic
- VPC - private IP space 
- two availability zones - resilience
- 6 subnets - three per each availability zone

# VPC and Network setup
## Three-Tier Architecture:
- Private subnet
- Public Subnet
- DMZ subnet

## VPC components: 

- 2 Elastic load balancers - DMZ Subnet 
- 2 Application servers - public subnet 
- 2 database servers - private subnets

Client can only access ELB DMZ subnet. 

APP servers communicate only with ELB. 
Database servers communicate with App servers only

Security groups act as a firewall - restricting access. 

## Steps

VLSM / CIDR Subnet calculator 
1. www.vlsm-calc.net
2. Vpc->create Vpc [specify name and CIDR range with subnet mask, no dedicated Tenancy]
192.160.0.0/19
3. Set Up 6 Subnets to override the 3 default subnets - size 256 hosts each 
![alt text](https://github.com/szczepanski/cloud-aws/blob/master/dev-1-architecture/Screen%20Shot%202018-03-11%20at%2013.17.37.png "Logo Title Text 1")

## Route table, Internet Gateway IGW and NAT Gateway 
Basic route table has been configured automatically once the initial VPC setup has been done. 
### Internet Gateway config and additional Route Tables 
4. Configure new Gateway and attach to VPC
5. Set Up Main Route Table - RT:
 Go to routes/ routes tab/edit/ and add another route:
0.0.0.0/0 - all web;  select target IGW: Dev-1-IGW; save
6. Set up new RT - Subnet specific - DMZ:
Add internet address and target Dev-1-IGW. 
save and switch to subnet associations tab and add two DMZ subnets
7. Set up new RT Subnet specific  - Public:
Add relevant subnets
8. Set up new RT Subnet specific  - Private:
Add relevant subnets
### NAT - Set up private route tables - NAT via NAT Gateway
The purpose of NAT - all private instances/ subnets cannot be accessed via Internet BUT can access internet without Public IP - NAT. 
9. NAT Gateways > Create NAT GW - two (HIGH availability HA)
One NAT GW per availability zone 
select public subnet 1 and allocate new EIP - elastic IP -  NAT GW 1
select public subnet 2 and allocate new EIP - elastic IP - NAT GW 2
10. Set up two new NAT private route tables (HA)
- 0.0.0.0/0 - all web;  select target NAT GW 1; save
- switch to subnet associations tab and Private subnet 1
- 0.0.0.0/0 - all web;  select target NAT GW 2; save
- switch to subnet associations tab and Private subnet 2

## Enable Auto Assign Public IP on subnets

Public IPs are required for application servers to be available over the internet. 

11. VPC/Subnets section 
- select Public Subnet 1 > modify auto assign IP setting (subnet actions) => tick enable
- select Public Subnet 2 > modify auto assign IP setting (subnet actions) => tick enable

# Application Server Setup 

## IAM Role - Identity and Access Management

Allows entities to call AWS services on one's behalf. 
Purpose: to allow the instance to perform specific actions and stronger security as AWS will handle permissions behind the scenes. 

IAM => Roles - pick service that will be using the role: EC2
Always go for the least privileged access method. 

12. Create a role with no permissions as these will be established and added later. 
Name it Dev-1EC2-Role

## EC2 - Application Server 1 Setup

13. Launch new instance - EC2-Dev-1
- Amazon Linux
- t2 micro
- change network to VPC-Dev-1 
- select Public Subnet 1 (app server available in public domain) 
- ensure auto assign IP setting is enabled
- Select Role Dev-1-EC2-Role
- ensure shutdown behaviour is set to Stop
- Tenancy - shared
- keep network interface settings default
- go to next config steps - storage
- select default 8G general purpose storage
- ensure Delete on termination is ticked (to avoid extra costs when not using it)
- go to add tags: (tags are needed also in terms of accountancy/billing - to recognize what instance added to costs)
  - Name: DEV-1 App Server 
  - Environment: Development 
- configure new security group Dev-1 Public SG 
Leave SSH port open to single - own IP - for testing 
- Add Rule - HTTP 0.0.0.0/0 
- create and save new Key Pair - Dev-1-KeyPair.pem
when testing ssh - ensure .pem file permissions are set to min 400

## ssh testing
14. ssh -i /KeyPair/file/location.pem  ec2-user@x.x.x.x

## initial server setup / patch
15. 
sudo yum update -y  [option -y => promptless]
sudo yum install -y httpd php
sudo service httpd start

## verify http apache server 
browser or curl to server public IP

## amend landing page - optional
```shell 
cd /var/www/html
echo "Piotr Testing Page" > index.html
``` 

## S3 configuration

16. configure standard S3 bucket, add new folder => builds and upload 
sample page/application  

## IAM Role adjustment
17. Create policy 
IAM>Roles>Dev-1-EC2-role> Add inline policy>JSON Tab
For assistance:
https://docs.aws.amazon.com/AmazonS3/latest/dev/using-with-s3-actions.html
```JSON
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "dev1s3accesspolicy",
            "Effect": "Allow",
            "Action": [
                "s3:*"
            ],
            "Resource": [
                "arn:aws:s3:::piotr.szczepanski/Dev-1/*",
                "arn:aws:s3:::piotr.szczepanski/Dev-1"
            ]
        }
    ]
}
```
18. Attach above policy to Dev-1-EC2-role

## Adjust ec2 app server via ssh 
19. 
```shell
aws s3 cp help
aws s3 cp s3://piotr.szczepanski/Dev-1/builds/Dev-1-landingPage.zip Dev-1-landingPage.zip
rm index.html 
unzip Dev-1-landingPage.zip
rm -fR Dev-1-landingPage.zip
```
## update user data script

20. 
-y => promptless command flag
```shell
#!/bin/bash

sudo su
yum update -y
yum install -y httpd php
cd /var/www/html
aws s3 cp s3://piotr.szczepanski/Dev-1/builds/Dev-1-landingPage.zip Dev-1-landingPage.zip
unzip Dev-1-landingPage.zip
mv Dev-1-landingPage/* .
rm -rf Dev-1-landingPage.zip Dev-1-landingPage
sudo service httpd start
```

## Lunch Dev-1 EC2 Application Server 1 from scratch 
21. Add steps as before plus add a script in advanced setting - user data

## Check Services and daemons
service --status-all

## EC2 - Application Server 2 Setup
22. To create App Server 2 use right-click > launch more like this
Adjust name and subnet => change to 2nd availability 

## 
https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html

# Load Balancer Configuration

https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html

![alt text](https://github.com/szczepanski/cloud-aws/blob/master/dev-1-architecture/load_balencer_architecture.png)

## Set up

23. Load balancer type => select Application LB
Network LP option = > select only for high-performance networks, static IPs etc.  

Set name, internet facing, protocol http,select VPC,select avail. zones and related subnets:
eu-west-2a      DMZ Subnet 1
eu-west-2b      DMZ Subnet 2

Add tags: name, and environment
24. Configure Security Groups => name and create new security group => leave default TCP 80 ; to be fine-tuned later
25 Configure Routing and Health checks- name and create new target group
leave 

![alt text](https://github.com/szczepanski/cloud-aws/blob/master/dev-1-architecture/ALB%20config.png)

26. Register Targets
Add two app servers: 
Dev-1 EC2 Application Server 1
Dev-1 EC2 Application Server 2
Review and create

## Load balancer checks 

All stats (latency, requests, health and more) can be obtained Load Balancer or target groups. 
To access app servers via the App Load balancer, copy-paste into browser DNS A record (Load Balancer/ description tab)

## Security Groups clean up 
### Ensure app servers can access internet - unidirectional way - outbound only - needed when updating/ patching systems - access to external repositories, packages etc. 
Ensure client interaction is restricted only to Load balancers level - DMZ zones in following way:  

- ensure only load balancers are facing incoming web traffic

- ensure public subnet(app servers) can receive web traffic (port 80) from App Load balancers ONLY. 
27. Amend Dev-1 Public SG (App servers SG) so it gets traffic from Dev-1-External-Application-Load-Balancer-SG by pasting ALB SG ID into source field. 

![alt text](https://github.com/szczepanski/cloud-aws/blob/master/dev-1-architecture/adjust%20public%20SG.png)

- 28. ensure traffic from load balances - outbound - goes to app servers ONLY (not further to DB) - by pasting Dev-1 Public SG (Apps Servers SG) ID into App Load Balancers SG - outbound - destination field 

![alt text](https://github.com/szczepanski/cloud-aws/blob/master/dev-1-architecture/adjust%20load%20balancer%20SG.png)

- ensure there is no wide open ssh access to app servers

# Auto Scaling Groups Introduction

https://docs.aws.amazon.com/autoscaling/ec2/userguide/what-is-amazon-ec2-auto-scaling.html

![alt text](https://github.com/szczepanski/cloud-aws/blob/master/dev-1-architecture/autoscaling.png)

## Create launch configuration
29. EC2>Launch Auto Scaling

![alt text](https://github.com/szczepanski/cloud-aws/blob/master/dev-1-architecture/LC%20setup.png)

Add default storage, tick delete on termination
Select existing public security group for the EC2 app servers - Dev-1 Public SG
Review, select the .pem file and proceed to 

## Create Auto Scaling Group

30. 

![alt text](https://github.com/szczepanski/cloud-aws/blob/master/dev-1-architecture/AS%20group.png)

Save and update as follows:

![alt text](https://github.com/szczepanski/cloud-aws/blob/master/dev-1-architecture/update%20asg.png]
Save the update and check 
Verify whether the minimum 2 instances launched within correct 2 avail.zones.
ensure Application LB A DNS record works (browser check)

## Create Scaling Policies, Cloudwatch alarms, Simple Notification Services - SNS

31. Go to SNS and create 4 topics:
Dev-1 Scale-up alarm 
Dev-1 Scale down alarm 
Dev-1-Service-Anomaly 
Dev-1-AutoScalingActivityAlarm

32. EC2>Auto Scaling Target Groups Scaling Policies> Add policy> create simple Scale policy

Create Scale UP policy 

![alt text](https://github.com/szczepanski/cloud-aws/blob/master/dev-1-architecture/create%20scaling%20policy.png)


and create new alarm - High CPU -to trigger/add one new instance)

![alt text](https://github.com/szczepanski/cloud-aws/blob/master/dev-1-architecture/new%20alarm%20scale%20up.png)


33. Create Scale down policy and alarm - low CPU - for CPU utilization <=20%  - to remove one oldest instance, save

34. switch to notifications tab and create a new notification

create new notification linked to Dev-1-AutoScalingActivityAlarm
This will alarm whenever there is new launch,  termination, fail to launch or terminate.

35. Create New Cloudwatch Alarms

Load Balancing/Target Groups/ Monitoring Tab / Create New Alarm (Cloudwatch)

Create: Dev-1-Application-High-Average-Latency-Alarm

![alt text](https://github.com/szczepanski/cloud-aws/blob/master/dev-1-architecture/cloudWatchAlarm.png)

Create: Dev-1-Application-High-Average-Latency-Alarm-Recovery-Notice
Trigger it when latency is = < 3s 

36. Add auto scaling actions to previously created Alarms. 

Click on the specific alarm and open directly in CloudWatch/modify it:

- Add Auto Scaling Action - Scale Up -(add one instance) - while the alarm is on - state: Alarm
- Add Auto Scaling Action - Scale Down -(remove one instance) - while alarm recovery notice is on - State: Alarm

37. Configuring SNS Topic Subscriptions 
SNS/Topics/Subscribe to topic

- Set Protocol to email
- set endpoint to mail address

Open the subscribed email and confirm the subscription. 

To test go to 
Auto Scaling/Autoscaling Groups/Details Tab/ Edit 

- temporarily change desired  from 2 to 3; save

# Create and Configure MySql DB instance

## Configure designated security group 

38. Create a new security group 

Ensure when opening ports - here 3306 (MySQL) that any traffic via this port is coming from (source) public subnet - in this architecture - app servers subnet (not from DMZ - internet facing zone). 

Whenever additional ports are needed - set the source to public (app servers / non-internet facing) security group. 

## Setup RDS DB

39. Create new subnet group:
- go to RDS/Subnet groups 

Add two subnets -for 2 availability zones:

![alt text](https://github.com/szczepanski/cloud-aws/blob/master/dev-1-architecture/db%20subnet.png)

## Configure RDS Instances 

40. 
go to RDS/Instances/ Launch
pick DB type - here Amazon Aurora (MySql) and configure all required settings. 

# DNS Management

## Configure Amazon Certificate Manager (Amazon Issued SSl certificate).

SSL certificate needs to be assigned to load balancer by creating HTTPS listeners that forward the traffic to specific target groups. 

41. 
Certificate Manager > request a certificate
- add domain name
- select validation method => email (AWS validation email message to be sent to the domain owner registered address). 


Once approved by the owner certificate will show as issued. 

## Configure Load Balancer HTTPS Load Balancer 
42. 
Load Balancing/ Load Balancers/ Listeners/ 
add https port 443 

## Setting up Route 53

43. Go to Route 53/Create Hosted Zone

provide domain name and type: hosted zone and create. 

In order to have Route 53 managing the DNS, original (GoDaddy, 1&1, etc) nameservers entries need to be changed to the nameservers entries provided by AWS. 

Once this is done, entire DNS can be then managed from within Route 53. 

44. Create A record within Route 53 pointing to the Dev-1 Application Load Balancer (Simple route policy). 
45. Wait some time to test/resolve new A record (pointing to app LB)









# Terraform

Open source by hashicorp 

Use cases: - infrastructure version control and back up if prod config breaks; for multiple environments - minimization of config drift - consistency. 

 3 Basic components:
- config file.tf - written in hashicorp configuration language - HCL 
- cli => terraform plan
- cli => terraform apply

Easy variable declaration 
Good documentation. 
Referencing files  => such as user data. 

### Setup IAM user:
Create a new user in IAM with programmatic access:
dev-1-terraform-user

add it to the group:
dev-1-admin-programmatic-access

### Install AWS cli and configure the profile  


provider "aws" {
  region                  = "us-west-2"
  shared_credentials_file = "/Users/tf_user/.aws/creds"
  profile                 = "customprofile"
}

## To avoid extra costs, stop or terminate instances and go to auto scaling/ auto-scaling group/ set desired (instances) to 0. 
