

#  Top layer architecture 

![alt text](https://github.com/szczepanski/cloud-aws/blob/master/dev-1-architecture/architecture%20design.png "Logo Title Text 1")


- Region - geographic
- VPC - private IP space 
- two availability zones - resilence
- 6 subnets - three per each availability zone
# VPC set up
## Three Tier Architecture:
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

## Steps

VLSM / CIDR Subnet calculator 
1. www.vlsm-calc.net
2. Vpc->create Vpc [specify name and CIDR range with subnet mask, no dedicated Tenancy]
192.160.0.0/19
3. Set Up 6 Subnets to override the 3 default subnets - size 256 hosts each 
![alt text](https://github.com/szczepanski/cloud-aws/blob/master/dev-1-architecture/Screen%20Shot%202018-03-11%20at%2013.17.37.png "Logo Title Text 1")

## Route table and Internet Gateway IGW
### Route Table
Basic route table has been configured automatically once the initial VPC setup has been done. 
### Gateway config 
4. Configure new Gateway and attach to VPC
5. Set Up Main Route Table - RT:
 Go to routes/ routes tab / edit/ and add another route:
0.0.0.0/0 - all web;  select taget IGW: Dev-1-IGW; save
6. Set up new RT - Subnet specific - DMZ:
Add internet address and target Dev-1-IGW. 
save and switch to subnet associations tab and add two dmz subnets
7. Set up new RT Subnet specific  - Public:
Add relevant subnets
8. Set up new RT Subnet specific  - Private:
Add relevant subnets





 
