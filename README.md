# Build Your VPC and Launch a Web Server
 

## Objectives
After completing this lab, you should be able to:

- Create a virtual private cloud (VPC)
- Create subnets
- Configure a security group
- Launch an Amazon Elastic Compute Cloud (Amazon EC2) instance into a VPC

## Scenario
In this lab, you use Amazon Virtual Private Cloud (VPC) to create your own VPC and add additional components to produce a customized network for a Fortune 100 customer. You also create security groups for your EC2 instance. You then configure and customize an EC2 instance to run a web server and launch it into the VPC that looks like the following customer diagram:
## Customer diagram
![image](https://github.com/GowriAyyanar/Networking-Lab-2/assets/152156151/ce1fed86-5323-4a29-b429-ded68ca44358)

## Task 1: Create your VPC
In this task, you use the VPC Wizard to create a VPC, an internet gateway, and two subnets in a single Availability Zone. An internet gateway is a VPC component that allows communication between instances in your VPC and the internet.

After creating a VPC, you can add subnets. Each subnet resides entirely within one Availability Zone and cannot span zones. If a subnet's traffic is routed to an internet gateway, the subnet is known as a public subnet. If a subnet does not have a route to the internet gateway, the subnet is known as a private subnet.

The wizard also creates a NAT gateway, which is used to provide internet connectivity to EC2 instances in private subnets.

In the AWS Management Console, select the  Services menu, and then select VPC under Networking & Content Delivery.

In the left navigation menu, choose Elastic IPs.

Choose Allocate Elastic IP address.

On the Allocate Elastic IP address page, leave the settings as is, and choose Allocate.

In the left navigation menu, choose VPC Dashboard.

Choose Launch VPC Wizard.

For Step 1: Select a VPC Configuration, choose VPC with Public and Private Subnets.

Choose Select.

For Step 2: VPC with Public and Private Subnets, configure the following options:

VPC name: Enter Lab VPC
Availability Zone: From the dropdown list, choose the first Availability Zone.
Public subnet name: Enter Public Subnet 1
Availability Zone: From the dropdown list, choose the first Availability Zone (the same as used above).
Private subnet name: Enter Private Subnet 1
Elastic IP Allocation ID: Select the box, and select the displayed IP address.
Choose Create VPC. It may take a few minutes for the VPC to become available.

On the VPC Successfully Created page, choose OK.

The wizard has provisioned a VPC with a public subnet and a private subnet in the same Availability Zone together with route tables for each subnet:

![image](https://github.com/GowriAyyanar/Networking-Lab-2/assets/152156151/3dcac4ca-daf9-4e75-9fb1-7e5a71976fed)

Figure: The VPCs, CIDRs, and the creation of public and private route tables. You created these options using the VPC Wizard.

The public subnet has a Classless Inter-Domain Routing (CIDR) of 10.0.0.0/24, which means that it contains all IP addresses starting with 10.0.0.x.

The private subnet has a CIDR of 10.0.1.0/24, which means that it contains all IP addresses starting with 10.0.1.x.

 

## Task 2: Create additional subnets
In this task, you create two additional subnets in a second Availability Zone. This option is useful for creating resources in multiple Availability Zones to provide high availability.

In the left navigation pane, choose Subnets.
To configure the second public subnet, choose Create subnet and configure the following options:

VPC ID: From the dropdown list, choose Lab VPC.
Subnet name: Enter Public Subnet 2    
Availability Zone: From the dropdown list, choose the second Availability Zone.
IPv4 CIDR block: Enter 10.0.2.0/24
Choose Create subnet.

The subnet will have all IP addresses starting with 10.0.2.x.

To configure the second private subnet, choose Create subnet and configure the following options:

VPC ID: From the dropdown list, choose Lab VPC.
Subnet name: Enter Private Subnet 2
Availability Zone: From the dropdown list, choose the second Availability Zone.
IPv4 CIDR block: Enter 10.0.3.0/24
Choose Create subnet.

The subnet will have all IP addresses starting with 10.0.3.x.

 

## Task 3: Create a route table
You now configure the private subnets to route internet-bound traffic to the NAT gateway so that resources in the private subnet are able to connect to the internet while still keeping the resources private. To do this, you configure a route table.

Recall that a route table contains a set of rules, called routes, that are used to determine where network traffic is directed. Each subnet in a VPC must be associated with a route table; the route table controls routing for the subnet.

In the left navigation pane, choose Route Tables.

Select the check box for the route table with Yes in the Main column and Lab VPC in the VPC column. (Expand the VPC column if necessary to view the VPC name.)

In the lower pane, choose the Routes tab.

Recall that Destination 0.0.0.0/0 is set to Target nat-xxxxxxxx. This means that traffic destined for the internet (0.0.0.0/0) will be sent to the NAT gateway. The NAT gateway will then forward the traffic to the internet.

This route table is therefore being used to route traffic from private subnets.

In the Name column for this route table, choose the pencil , enter Private Route Table and then choose Save.

 

## Task 4: Associate the subnets and add routes
In the lower pane, choose the Subnet associations tab.

Under Subnets without explicit associations, choose Edit subnet associations.

Select the check boxes for both Private Subnet 1 and Private Subnet 2.

Choose Save associations.

 You now configure the route table that is used by the public subnets.

Select the check box for the route table with No in the Main column and Lab VPC in the VPC column, and clear the check boxes for any other route tables.

In the Name column for this route table, choose the pencil , enter Public Route Table and then choose Save.

In the lower pane, choose the Routes tab.

Note that Destination 0.0.0.0/0 is set to Target igw-xxxxxxxx, which is the internet gateway. This means that internet-bound traffic will be sent straight to the internet via the internet gateway.

You now associate this route table to the public subnets.

Choose the Subnet associations tab.

In the Subnets without explicit associations section, choose Edit subnet associations.

Select the check boxes for both Public Subnet 1 and Public Subnet 2.

Choose Save associations.

Your VPC now has public and private subnets configured in two Availability Zones:

![image](https://github.com/GowriAyyanar/Networking-Lab-2/assets/152156151/7abc3edd-aaa4-42b3-80b0-964aedb3f6ef)

Figure: The creation of the networking resources and routing components and attachment of these resources that make the VPC functional as a network.

 

## Task 5: Create a VPC security group
In this task, you create a VPC security group, which acts as a virtual firewall for your instance. When you launch an instance, you associate one or more security groups with the instance. You can add rules to each security group that allow traffic to or from its associated instances.

In the left navigation pane, choose Security Groups.

Choose Create security group.

Configure the security group with the following options:

Security group name: Enter Web Security Group
Description: Enter Enable HTTP access
VPC: Choose Lab VPC.
Choose Create security group.

You now add a rule to the security group to permit inbound web requests.

Choose the Inbound rules tab.

Choose Edit inbound rules

Choose Add rule.

Configure the following options:

Type: Choose HTTP.
Source: Choose Anywhere.
Description: Enter Permit web requests
Choose Save rules.

You use this security group in the next task when launching an EC2 instance.

 

Task 6: Launch a web server instance
In this task, you launch an EC2 instance into the new VPC. You configure the instance to act as a web server.

In the AWS Management Console, select the  Services menu, and then select EC2 under Compute.

Choose Launch instances

First, you choose an Amazon Machine Image (AMI), which contains the desired operating system.

For Step 1: Choose an Amazon Machine Image (AMI), choose Select for Amazon Linux 2 AMI (HVM) - Kernel 5.10, SSD Volume Type.

The instance type defines the hardware resources assigned to the instance.

For Step 2: Choose an Instance Type, choose the check box for t2.micro.

Choose Next: Configure Instance Details

You now configure the instance to launch in a public subnet of the new VPC.

For Step 3: Configure Instance Details, configure the following settings:

Network: Choose Lab VPC.
Subnet: Choose Public Subnet 2. (Be careful not to choose the private subnet.)
Auto-assign Public IP: Choose Enable.
At the bottom of the page, expand the  Advanced Details section.

Copy and paste the following code into the User data box:

#!/bin/bash
# Install Apache Web Server and PHP
yum install -y httpd mysql php
# Download Lab files
wget https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RESTRT-1/267-lab-NF-build-vpc-web-server/s3/lab-app.zip
unzip lab-app.zip -d /var/www/html/
# Turn on web server
chkconfig httpd on
service httpd start
This script runs automatically when the instance launches for the first time. The script loads and configures a PHP web application.

Choose Next: Add Storage

For Step 4: Add Storage, you use the default settings for storage. Choose Next: Add Tags to move onto the next step.

Tags can be used to identify resources. You use a tag to assign a name to the instance.

For Step 5: Add Tags, choose Add Tag

Configure the following options:

Key: Enter Name
Value: Enter Web Server 1
Choose Next: Configure Security Group

 You configure the instance to use the Web Security Group that you created earlier.

For Step 6: Configure Security Group, for Assign a security group, select  Select an existing security group.

Select the check box for the  vockey | RSA security group.

 This is the security group that you created in the previous task. It permits HTTP access to the instance.

Choose Review and Launch

When prompted with a warning that you will not be able to connect to the instance through port 22, choose Continue

For Step 7: Review Instance Launch, review the instance information, and choose Launch

In the Select an existing key pair or create a new key pair window, select the check box next to  I acknowledge that I have access to the corresponding private key file, and that without this file, I won't be able to log into my instance.

Choose Launch Instances

Choose View Instances

Wait until the Web Server 1 shows 2/2 checks passed in the Status check column.

  This may take a few minutes. To update the page, choose refresh  at the top of the page.

 You now connect to the web server running on the EC2 instance.

Select the check box for the instance, and choose the Details tab.

Copy the Public IPv4 DNS value.

Open a new web browser tab, paste the Public IPv4 DNS value, and press Enter.

When successful, the page should look like the following:

![image](https://github.com/GowriAyyanar/Networking-Lab-2/assets/152156151/9f440124-7050-41a0-be64-58118cc0a5a4)

The following is the complete architecture that you deployed:

![image](https://github.com/GowriAyyanar/Networking-Lab-2/assets/152156151/2c005d5c-a434-408f-9c18-d14ae17db76e)



