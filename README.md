# AWS VPC Setup

## Instructions on how to set up infrastructure behind a secure virtual private cloud on Amazon web services.

### So what good is all this?
- Protects all of your infrastructure behind a secure network, so it's an added layer of security.
- Forces all access to your instances to go through a limited number of manageable points.
- Allows you to interact quickly and more freely on a private network while staying secure.

*SPECIAL NOTE: There are a couple of notes in this document were I'm unclear on how it actually came together. Please read over the notes first to make sure you know what you're geting yourself into. Maybe You can help me clear these parts up.*

I'm going to start with an outline based on my most recent notes. Once I've documented the overall process, I'll take some time and go back and flesh it out with more detail. A lot of this comes directly from the AWS docs. 

So, one thing to keep in mind here is there are countless ways to set these things up. This may not be the perfect way for every system, but it's a solid config that I use a lot. I'm documenting it because it works for me in most cases and the AWS docs take so long to study and analyze. I just wanted a quick way to share this process and get feedback from the community.

### Ultimately, we're going to set up these items:
|Service|Usage|
|--|--|
|Virtual Private Cloud|Secures all of the system's infrastructure.|
|Security Group|Controls access to all the instances on the vpc.|
|Network ACL (Access Control List)||
|Private Subnets|Allows the instances to communicate behind the vpc.|
|Public Subnets|Opens up access for your NAT Gateway and your IGW| 
|Route Tables|To manage your subnets and gateways.|
|Elastic IP|A Static IP to point your DNS records at while you swap your instances in and out.|
|NAT (Network Access Translation) Gateway|So that your |
|IGW (Internet Gateway)||
|Elastic Network Interfaces (ENI)||

This will set up everything we need to secure application instances behind a VPC with a single bastion instance that is publicly accessible for outside access. 

### I. Create a Virtual Private Cloud (VPC)
>A [VPC](https://docs.amazonaws.cn/en_us/vpc/latest/userguide/what-is-amazon-vpc.html) is an isolated portion of the AWS cloud populated by AWS objects, such as Amazon EC2 instances. You must specify an IPv4 address range for your VPC. Specify the IPv4 address range as a Classless Inter-Domain Routing (CIDR) block; for example, 10.0.0.0/16. You cannot specify an IPv4 CIDR block larger than /16. You can optionally associate an Amazon-provided IPv6 CIDR block with the VPC.
1. Go to the VPC console and select "Your VPCs".
2. Click the "Create VPC" button.
3. Use a /16 subnet.
4. Select default tenancy.
 
### II. Network ACL
>A [Network ACL](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-network-acls.html) is an optional layer of security that acts as a firewall for controlling traffic in and out of a subnet.
1. From the VPC console, select "Network ACLs"
2. Click the "Create network ACL" button.
3. Name it.
4. Add it to the new VPC.
5. Once it's created, go back and add rules to allow inbound and outbound traffic
 
### III. Create Subnets
> Specify your subnet's IP address block in CIDR format; for example, 10.0.0.0/24. IPv4 block sizes must be between a /16 netmask and /28 netmask, and can be the same size as your VPC. An IPv6 CIDR block must be a /64 CIDR block.
You're going to need at least four subnets to make this work. Two public and two private. They are all going to be 
1. Make each name tag descriptive, so they will be easy to work with later. For example, `public_1` and `public_2`. Then `private_1` and `private_2`.
2. Select the new VPC.
3. Alternate the availability zones so that both public subnets are not in the same zone and both of the private ones are in different zones as well. I like to use `us-east-1`. Just use whatever makes sense for your system,
4. Set the IPv4 CIDR block to `10.0.x.0/24`
5. Click create. You'll have to do all this for each one until you have created all four.
6. 

### IV. Edit Subnet Associations
> Now that you have all the subnets you need, you can associate them with the ACL you created.
1. From the VPC console, select Network ACLs.
2. Click on your new ACL to highlight it.
3. In the "Actions" dropdown, select "Edit subnet associations".
4. Add all four subnets and save it.

### V. Create Route Tables
>A [Route Table](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Route_Tables.html) specifies how packets are forwarded between the subnets within your VPC, the internet, and your VPN connection.
1. Go to the "Route Tables" tab. You should already have a route table for the VPC. Name this something descriptive that follows the same convention as the rest of the infrastructure so you know what goes with what if you or the client builds more systems on this AWS account.
2. Create a new route table. Click the "Create route table" button. Name it for the private subnet, so you can easily select it when working with private subnets.
3. Select the VPC from the dropdown.
4. Click "Create".
5. Now repeat the 2 and 3 for the public route table, naming it for the public subnet so you can tell them apart easily.

### VI. Explicitly Associate Your Subnets with Your Route Tables. 
1. Click on the public route table to select it. 
2. In the "Actions" dropdown, click "Edit subnet associations".
3. Click on the two public subnets to associate them. They should be easy to spot if you named them descriptively.
4. Save your associations.
5. Repeat 1--4 for your private route table.
6. Set the private route table as the main route table. Click it to highlight it. In the "Actions" dropdown, click "Set Main Route Table".

### VII. Create an Elastic IP
>An  [Elastic IP](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)  address is a static IPv4 address designed for dynamic cloud computing. An Elastic IP address is associated with your AWS account. With an Elastic IP address, you can mask the failure of an instance or software by rapidly remapping the address to another instance in your account.
1. From the Elastic IPs tab, click "Allocate new address".
2. Just let Amazon pull one from their pool, unless you have one you insist on using.
3. Click "Allocate".

### VIII. Create a NAT Gateway
>You can use a network address translation (NAT) gateway [network address translation (NAT) gateway](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html) to enable instances in a private subnet to connect to the internet or other AWS services, but prevent the internet from initiating a connection with those instances.
1. From the NAT Gateway tab, click "Create NAT Gateway".
2. Select one of your new public subnets.
3. Select your new elastic ip.
4. Click "Create a NAT Gateway"
 
### XI. Add a Route to Your Private Route Table for Your NAT Gateway
1. From the "Route Tables" tab, click on the private subnet to highlight it. (Do you see now why we named them descriptively). 
2. Click the "Routes" tab below.
3. Click "Edit Routes".
4. Click "Add route".
5. Set the Destination to `0.0.0.0/0`
6. Set the Target to the new NAT Gateway.
7. Save it.

*NOTE: For some reason that last step failed the first time I tried it. I'll update this the next time I follow this process. I'm not sure what fixed it.*

### X. Internet Gateway (IGW)
>An [internet gateway](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html) is a virtual router that connects a VPC to the internet. To create a new internet gateway specify the name for the gateway below.
1. From the Internet Gateways tab, click "Create Internet Gateway"
2. Give it a name. Remember, name everything descriptively with a unique convention.
3. Save it.
3. Attach it to the VPC. Click on the new IGW to highlight it in the list.
4. In the actions droplist, click "Attach to VPC".
5. Select your new VPC.
6. Give it a route in the public route table the same way you added the NAT Gateway to the private route table. 
 
### XI. Create a Security Group
>
1. Finally, you're going to bounce over to the [EC2 Console](https://console.aws.amazon.com/ec2/v2/home?region=us-east-1#Home:) and create a security group if you haven't already.
2. From the "Security Groups" tab, click "Create Security Group".
3. Name it. (I'm going to go ahead and keep repeating this. Be descriptive. You're not a robot yet.)
4. Add a description while you're at it.
5. Select the new VPC.
6. You can open up all traffic in the rules for now, but make sure you come back and adjust it to only allow the ports you are using and the ips of the people that will be working on it. For example, you'll want to add ports 80 and 443 for HTTP and HTTPS. You'll also want 22 for SSH and 3306 for database. There could be more for other services you might use and these could be different depending on how you set up your particular system. The point is, only allow what you need an only while you need it.

### XII. Create a Network Interface and Associate It with Your Elastic IP
>An [Elastic Network Interface (ENI)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html) (referred to as a network interface in this documentation) is a logical networking component in a VPC that represents a virtual network card.
1. Also from the EC2 Console
2. From the "Network Interfaces" tab, click "Create Network Interface"
3. Add a description.
4. Add one of your public subnets.
5. Select your new security group from the list.
6. Click "Create".
7. Click on your new ENI in the list to highlight it.
8. From the "Actions" dropdown, click associate address.

*NOTE: That last step is documented as failing. Looking back, it appears that this step may be automated. It looks like AWS generated on for me that ended up being attached to the elastic IP for me. Again, I'll watch more closely next time I follow these docs so I can improve this part.*

# That's It!
You now have a secure VPC with all of the infrastructure you need to make your instances public on the web, while also protecting them from access but allowing your team to tunnel into any of them through a single bastion. 

Next, I'll be adding a docs about Load Balancing, AutoScaling and CodeDeploy with BitBucket Pipelines.
