# AWS VPC Setup

## Instructions on how to set up infrastructure behind a secure virtual private cloud on Amazon web services.

I'm going to start with an outline based on my most recent notes. Once I've documented the overall process, I'll take some time and go back and flesh it out with more detail. A lot of this comes directly from the AWS docs. 

So, one thing to keep in mind here is there are countless ways to set these things up. This may not be the perfect way for every system, but it's a solid config that I use a lot. I'm documenting it because it works for me in most cases and the AWS docs take so long to study and analyze. I just wanted a quick way to share this process and get feedback from the community.

### Ultimately, we're going to set up these items:
- Virtual Private Cloud to secure all of the system's infrastructure.
- Security Group to control access to all the instances on the vpc.
- Network ACL (Access Control List)
- Private Subnets to allow the instances to communicate behind the vpc.
- Public Subnets 
- Route Tables
- Elastic IP
- NAT (Network Access Translation) Gateway 
- IGW (Internet Gateway)
- Elastic Network Interfaces (ENI)

This will set up everything we need to secure application instances behind a VPC with a single bastion instance that is publicly accessible for outside access. 

### I. Create a VPC
>A VPC is an isolated portion of the AWS cloud populated by AWS objects, such as Amazon EC2 instances. You must specify an IPv4 address range for your VPC. Specify the IPv4 address range as a Classless Inter-Domain Routing (CIDR) block; for example, 10.0.0.0/16. You cannot specify an IPv4 CIDR block larger than /16. You can optionally associate an Amazon-provided IPv6 CIDR block with the VPC.
1. Go to the VPC console and select "Your VPCs".
2. Click the "Create VPC" button.
3. Use a /16 subnet.
4. Select default tenancy.

### II. Network ACL
>A network ACL is an optional layer of security that acts as a firewall for controlling traffic in and out of a subnet.
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
