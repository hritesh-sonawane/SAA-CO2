##VPC
_`Configuring VPC` hands-on walkthrough sessions (3 sessions, 1 hour each)_
_for **Cloud Computing** Class in `SEM 6` of undergrad_

1. Navigate to the VPC section of the AWS console
2. Click Your VPCs
3. Click Create VPC
4. Give it a Name Tag: VPC-demo
5. Give it an IPv6 block if so desired (auto assigns a /56 IPv6 Block)
6. Leave Tenancy to Default

###`Subnet`

```
10.0.0.0/24 - Public Subnet for Web Servers [ vpc-demo-a-public ]
10.0.1.0/25 - Private Subnet for API Servers [ vpc-demo-a-private ]
10.0.1.128/26 - Database Subnet for DBs [ vpc-demo-a-db ]
10.0.1.192/26 - Spare Subnet [ vpc-demo-a-spare ]
```

1. In the VPC console, go to Subnets on the side panel.
2. Click on Create Subnet
3. Give it a Name Tag: vpc-demo-a-public
4. For Availability Zone, choose ap-south-1a
5. For IPv4 CIDR block type 10.0.0.0/24
6. Rinse and repeat for the above ranges

<br />

###`Route Tables`
**The Public Route Table**

1. In the VPC console, click on Route Tables in the side bar
2. Click Create Route Table
3. Give the Name Tag something like: public-roads
4. Select the VPC-demo VPC
5. Click Yes, Create

_Now we need to create the Internet Gateway_

1. In the VPC console, click on Internet Gateways
2. Click Create Internet Gateway
3. Give it a Name Tag, something like: public-roads-igw
4. After it's created click on Attach to VPC

_Now to set it up on the Route Table_

1. In the VPC console, click Route Tables
2. Select the Route Table that we created
3. In the bottom tabs, under Routes click Edit
4. Click Add another route
5. For Destination input 0.0.0.0/0
6. For Target select our newly created internet gateway: public-roads-igw
7. Click Save but don't leave this page yet...
8. In this same page, with our Route Table Selected, click Subnet Associations
9. Click Edit and select our vpc-demo-a-public subnet, or whichever subnet you'd like to be public

**The Private Route Table**
_We could just use the default_

1. In the VPC console, click on Route Tables in the side bar
2. Click Create Route Table
3. Give the Name Tag something like: private-roads
4. Select the VPC-demo VPC
5. Click Yes, Create

_Now we need to create the NAT Gateway_

1. In the VPC console, click on NAT Gateways
2. Click Create NAT Gateway
3. Select our Public Subnets we want our NAT gateway to sit in: vpc-demo-a-public
4. Click Create New EIP
   _NAT Gateways require an Elastic IP Address_
5. Click Create a NAT Gateway and View NAT Gateways (refresh the screen)

**Rinse and repeat!**

1. In the VPC console, click Route Tables
2. Select the Route Table that we created, private-roads
3. In the bottom tabs, under Routes click Edit
4. Click Add another route
5. For Destination input 0.0.0.0/0
6. For Target select our newly created nat gateway
7. Click Save but don't leave this page yet...
8. In this same page, with our Route Table Selected, click Subnet Associations
9. Click Edit and select our vpc-demo-a-private, vpc-demo-a-db and vpc-demo-a-spare subnets.

_And there's our private subnet with access to the internet. For IPv6 private subnets, we'd only need to:_

1. Create an Egress Only Internet Gateway
2. Select our VPC
3. In our Route Table private-roads, click Edit
4. For Destination input ::/0
5. For Target select our Egress Only Internet Gateway

<br />

####`Security group is the firewall of EC2 Instances`

####`Network ACL is the firewall of the VPC Subnets`

_Security groups are stateful_
This means any changes applied to an incoming rule will be automatically applied to the outgoing rule.
e.g. If you allow an incoming port 80, the outgoing port 80 will be automatically opened.

_Network ACLs are stateless_
This means any changes applied to an incoming rule will not be applied to the outgoing rule.
e.g. If you allow an incoming port 80, you would also need to apply the rule for outgoing traffic.

####`Rules: Allow or Deny`
Security group support allow rules only (`by default all rules are denied`).
e.g. You cannot deny a certain IP address from establishing a connection.
Network ACL support allow and deny rules.
By deny rules, you could explicitly deny a certain IP address to establish a connection example:
Block IP address 123.201.57.39 from establishing a connection to an EC2 Instance.

**Rule process order**
All rules in a security group are applied whereas rules are applied in their order
(`the rule with the lower number gets processed first`) in Network ACL.
100, 200, 300, 400, 500

**Defense order**

```
Network ACL first layer of defense
Security group is second layer of the defense for inbound/ingress traffic.
```

```
Security group first layer of defense
Network ACL is second layer of the defense for outbound/egress traffic.
```

_Subnet can have only one NACL, whereas Instance can have multiple Security groups._
