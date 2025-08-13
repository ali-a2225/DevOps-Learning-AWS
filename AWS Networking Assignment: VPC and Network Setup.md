**Objective:**  
Create a custom VPC with both public and private subnets, configure internet access, and deploy EC2 instances with secure segmentation.

**Requirements:**

1. **VPC & Subnets:**
    - Create a custom VPC with at least one public and one private subnet.

2. **Internet & NAT Gateway:**
    - Attach an Internet Gateway (IGW).
    - Deploy a NAT Gateway (NGW) in the public subnet.
 
3. **EC2 Instances:**
    - Launch one EC2 instance in the public subnet (with internet access).
    - Launch another EC2 instance in the private subnet (no direct internet access).

4. **Routing & Security:**
    - Configure route tables for correct routing.
    - Restrict direct access to the private instance.

5. **Bonus Challenge (Optional):**
    - Set up a Bastion Host (Jump Box).
    - Enable CloudWatch monitoring and logging.


---


## Initial thoughts

- Create a custom VPC with at least one public and one private subnet - use CIDR notation to create a private subnet, will probably start with 172., assign public IP
- Attach an Internet Gateway (IGW) - internet gateway for accessing the internet
- Deploy a NAT Gateway (NGW) in the public subnet - NAT Gateway is required due to IPv4 addresses having a private and public split
- Launch one EC2 instance in the public subnet (with internet access) - need to figure this out
- Launch another EC2 instance in the private subnet (no direct internet access) - need to figure this out
- Configure route tables for correct routing
- Restrict direct access to the private instance - allow traffic through to internet but block replies
- Set up a Bastion Host (Jump Box) - need to figure this out
- Enable CloudWatch monitoring and logging - need to figure this out

## Tasks
- [x] create a customer VPC with 1 private and 1 public subnet
- [x] create an IGW, watch video on IGW
- [x] attach IGW to the VPC, watch video on how to attach IGW
- [x] figure out how to deploy NAT Gateway in the public subnet
- [x] figure out how to launch and EC2 instance in a public subnet
- [x] review notes on routing tables to see how they are meant to be configured
- [ ] figure out how to restrict traffic to private subnet
- [x] figure out how to set up a bastion host or jump box
- [ ] figure out how to enable cloud watch monitoring




## Action  


# Create VPC and Subnets
- created a VPC + 1 private subnet (IPv4 + IPv6 (also public))



- place subnets in different AZs for high availability
- created 3 subnets in north1a 1b and 1c respectively and expanded the VPC's CIDR block as /28 was too restrictive
- I deleted the VPC and the subnet I created and then remade it



# Attach IGW
- all the subnet above have no access to the internet
- I need to attach IGW to allow internet access for at least 1 subnet
- created IGW and attached to VPC



# Private Route
- create Private route associate with private subnet




# Assign Public Route to Public Subnet
- creating a public route and assign to public subnets
- target is IGW for internet access


- edit auto-assign IP settings


# Launch an Instance 
## Public 



# Private



# Assign NAT Gateway to Private Subnet


# Bastion Host
## Allow Access from Bastion Host (in the Public Subnet)
- attach security group to instance in the private subnet that allows SSH from the public subnet instances or a Bastion Host



## Test connection from Bastion Host to Private Instance
1. copy private key to a file on the instance
2. change permissions chmod 400
3. ssh -i prikey.pem ec2-user@10.0.0.157



- top ping is with NAT gateway attached to private subnet
- bottom ping is without NAT gateway






# Steps
1. Create VPC
2. Create 2 Subnets in VPC (Public and Private) within the new VPC
3. Set up 2 Route Tables (Public & Private)
4. Associate each Route Table with its corresponding Subnet
5. Configure the Public Subnet’s Route Table to route traffic through the Internet Gateway (IGW)
6. Configure the Private Subnet’s Route Table to route traffic through the NAT Gateway
7. Create a Security Group (SG) to allow secure communication between Create SG between the Private Instance (Private Subnet Instance) and the Bastion Host (Public Subnet Instance)









# Key Takeaways
## CIDR
- AWS does not allow modifying the CIDR block of an existing subnet or VPC. Once created, CIDR blocks are immutable
- Subnets cannot be smaller than `/28` (16 IPs). AWS reserves 5 IPs per subnet (e.g., for routing/DNS), so a `/28` subnet gives you 11 usable IPs.
## Public and Private Subnets
- If a subnet is associated with a route table that has a route to an internet gateway, it's known as a public subnet. 
- If a subnet is associated with a route table that does not have a route to an internet gateway, it's known as a private subnet.  https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html

- The "Enable auto-Assign Public IPv4 address" button in the subnet means the instance automatically get assigned a Public IP on launch, if it is disabled you must manually assign a public IP
- Without a Public IP assigned to the instance, the IGW has no public IP to map to the instance's private IP address
## Internet Gateways
- IGW - allows any IP in that subnet send traffic and receive new inbound connections from the internet (unless security groups/NACLs block it)

## NAT Gateways
- NAT gateway - a managed device inside your VPC that rewrites source IPs to its Elastic IP and forwards packets to the internet via an IGW.
- a NAT gateway is for letting private instances _initiate_ outbound IPv4 connections and receive replies while the internet cannot initiate new inbound connections to them, new inbound connections from the internet are dropped.
- A NAT gateway itself must:
      - live in a **public subnet** 
      - use an **Elastic IP (EIP)**
      - route traffic through the IGW to reach the internet.
- NAT gateway + public subnet + EIP + Internet Gateway (IGW) = outbound internet for private IPv4 instances
- For high availability across two multi-AZ setups, create one NAT gateway per AZ (each in a public subnet in that AZ) and point each private subnet’s route table to the NAT gateway in its AZ.
- NAT Gateway → only allows _replies_ to connections your instances initiate. Inbound new connections from internet are dropped.
- IGW only vs NAT + IGW
1. instances get public IP addresses and talk straight to the internet, they can send and receive connections from the internet, this is a public subnet 
2. instances don't get an IP address, they direct traffic to the NAT gateway which has a public IP address. Instances can **initiate** outbound connections and get replies, but cannot accept new inbound connections from the internet, this is a private subnet

## IPv6
- IPv6 addresses are globally unique, and therefore public by default.
- IPv4 + NAT -> NAT's public IP
- IPv6 + Egress-Only IGW -> keeps its unique IPv6 address
### Egress-Only Internet Gateway
- Use an **Egress-Only Internet Gateway** for IPv6 outbound-only access, it allows outbound communication over IPv6 from instances in your VPC to the internet, and prevents the internet from initiating an IPv6 connection with your instances.
- An Egress-Only IG (serves the same purpose as a NAT Gateway (IPv4) for IPv6.
- it forwards traffic from the instances in the subnet to the internet or other AWS services, and then sends the response back to the instances.
- Unlike NAT (which hides IPv4 addresses), this gateway doesn’t hide IPv6 - it just blocks unsolicited inbound traffic.

## Security Groups
- Security groups are like a stateful firewall rulebook attached directly to your cloud server (instance)
1. blocking everything by default 
2. only allowing what you explicitly permit based on IPs/other SGs, ports, and protocols.
- Rules can reference other security groups instead of raw IP ranges
### Scenario
- Imagine you have a fleet of web servers (SG-Web) talking to a fleet of database servers (SG-DB). 
- Instead of listing _every single web server IP_ in the SG-DB rules (which would be a nightmare to update when servers start/stop or scale), you simply put a rule in SG-DB: `Allow TCP port 3306 FROM SG-Web`. 
- any instance with SG-Web attached can talk to the DB. 
- if you add a new web server, just attach SG-Web, it automatically gets access.














