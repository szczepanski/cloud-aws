

#  Top layer architecture 

![alt text](https://github.com/szczepanski/cloud-aws/blob/master/dev-1-architecture/architecture%20design.png "Logo Title Text 1")


- Region - geographic
- VPC - private IP space 
- two availability zones - resilence
- 6 subnets - three per each availability zone
# VPC
## Tier Architecture architecture:
- Private subnet
- Public subnet
- DMZ subnet

## VPC components: 

- 2 Elastic load balancers - DMZ Subnet 
- 2 Application servers - public subnet 
- 2 database servers - private subnets

Client can only access ELB DMZ subnet. 

APP servers communicates only with ELB. 
Database servers communicate with App servers only

Security groups act as firewall - restricting access. 

## VPC Set UP

VLSM / CIDR Subnet calculator 
1. www.vlsm-calc.net
2. Vpc->create Vpc [specify name and CIDR range with subnet mask, no dedicated Tenancy]
192.160.0.0/19
3. Set Up 6 Subnets to override the 3 default subnets - size 256 hosts each 
![alt text](cloud-aws/dev-1-architecture/Screen Shot 2018-03-11 at 13.17.37.png "Logo Title Text 1")

# Route table and Gateway 

 
