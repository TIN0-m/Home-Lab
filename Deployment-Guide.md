# Deployment Guide
##### This is a step by step guide to how the VPC was set up and all the components it has have been set up.

## AWS Account Preparation

1. #### If you don't have an AWS account, sign up at aws.amazon.com.
   
2. #### Select Your AWS Region:
In the AWS Management Console, look at the top right corner. Ensure you select "Africa (Cape Town) " if available and you prefer local. For this guide, I'll use eu-west-1. Consistency in region is important.

3. #### Create an EC2 Key Pair:
3.1   - Go to EC2 dashboard.     
3.2   - In the left navigation pane, under "Network & Security," click "Key Pairs."     
3.3   - Click "Create key pair."      
3.4   - Name: "Something suitable"     
3.5   - Key pair type: RSA      
3.6   - Private key file format: .pem     
3.7   - Click "Create key pair."     
3.8   - Download the .pem file immediately. Keep it secure; you'll need it to connect to your instances via SSH/RDP.     

4. ## AWS VPC Network Infrastructure Setup
**Creating the Virtual Private Cloud (VPC):**        
4.1 - Go to the VPC dashboard.    
4.2 - In the left navigation pane, click "Your VPCs."     
4.3 - Click "Create VPC."     
4.4 - Name tag: Lab     
4.5 - IPv4 CIDR block: 10.0.0.0/16 (This gives you a large, private IP range for all your resources).     
4.6 - Leave other settings as default.     
4.7 - Click "Create VPC."     
4.8 - Create Subnets:       
4.9 - In the VPC dashboard, click "Subnets."      
4.10 - Click "Create subnet."      

5. ## Subnets Setup
**Repeat this process four times for the following subnets.**

5.1 - Name tag: Public-Subnet      
5.2 - Availability Zone: Choose one (e.g., eu-west-1a).     
5.3 - IPv4 CIDR block: 10.0.1.0/24     

**For the rest of the subnets please ensure that the IPv4 CIDR blocks are set as follows:**
- Coperate Network : 10.0.10.0/24
- SOC Hub          : 10.0.20.0/24
- Management       : 10.0.30.0/24

6. ## Internet Gateway Setup

6.1 - In the VPC dashboard, click "Internet Gateways."     
6.2 - Click "Create internet gateway."     
6.3 - Name tag: SOC-Lab-IGW       
6.4 - Click "Create internet gateway."       
6.5 - Once created, select it and click "Actions" -> "Attach to VPC."     
6.6 - Select your SOC-Lab-VPC from the dropdown.      
6.7 - Click "Attach internet gateway."      

7. ## Create Route Tables:
**Repeat this twice for the Private and Public route tables** 

7.1 - In the VPC dashboard, click "Route Tables."   
7.2 - Create Public Route Table:    
7.3 - Click "Create route table."    
7.4 - Name tag: SOC-Lab-Public-RT    
7.5 - VPC: Select SOC-Lab-VPC.    
7.6 - Click "Create route table."     
7.7 - Select SOC-Lab-Public-RT. Go to the "Routes" tab -> "Edit routes."     
7.8 - Click "Add route."      
7.9 - Destination: 0.0.0.0/0 (this means "all internet traffic")     
7.10 - Target: Select "Internet Gateway" and choose your SOC-Lab-IGW.    
7.11 - Click "Save changes."      
7.12 - Go to the "Subnet associations" tab -> "Edit subnet associations."    
7.13 - Select SOC-Lab-Public-Subnet.     
7.14 - Click "Save associations."      

**For the Private route table please take note of the following:**       
- Do NOT add the 0.0.0.0/0 route yet. We'll do that after the NAT Gateway.     
- Select SOC-Lab-Enterprise-Subnet, SOC-Lab-SOC-Tooling-Subnet, and SOC-Lab-Management-Subnet.        


8. ## Create NAT Gateway:

8.1 - In the VPC dashboard, click "NAT Gateways."     
8.2 - Click "Create NAT gateway."     
8.3 - Name tag: SOC-Lab-NAT-GW      
8.4 - Subnet: Select SOC-Lab-Public-Subnet (the NAT GW must be in a public subnet).      
8.5 - Connectivity Type: Public.      
8.6 - Elastic IP allocation ID: Click "Allocate an Elastic IP address" (this gives it a static public IP).     
8.7 - Click "Create NAT gateway." This might take a few minutes to become available.     
8.8 - Update Private Route Table to Use NAT Gateway:     
8.9 - Once the NAT Gateway status is "Available" (check in the NAT Gateway section), go back to "Route Tables" in the VPC dashboard.     
**This is the part we update the route table for the Private route table**
8.10 - Select SOC-Lab-Private-RT. Go to the "Routes" tab -> "Edit routes."     
8.11 - Click "Add route."     
8.12 - Destination: 0.0.0.0/0     
8.13 - Target: Select "NAT Gateway" and choose your SOC-Lab-NAT-GW.     
8.14 - Click "Save changes."      

9. ## Security Groups
**Repeat this twice for each of the Security Groups**
9.1 - Create Security Groups:
9.2 - Go to the EC2 dashboard.
9.3 - In the left navigation pane, under "Network & Security," click "Security Groups."
9.4 - Click "Create security group."
9.5 - SOC-Lab-JumpBox-SG (Critical for Initial Access)
9.6 - Description: Allows secure RDP/SSH access to the Jump Box.
9.7 - Inbound rules:
- Type: RDP (Port 3389)
- Source: "My IP".
- Add another rule if using Linux Jump Box: Type: SSH (Port 22), Source: "My IP".
9.8 - Outbound rules:
All traffic (0.0.0.0/0) to All (::/0) - this is temporary, we'll refine it later.

b. SOC-Lab-AD-DC-SG
9.7 - Inbound rules:

Type: RDP (Port 3389)
Source: SOC-Lab-JumpBox-SG (select by SG ID).
Type: LDAP (Port 389)
Source: SOC-Lab-Enterprise-Subnet CIDR (10.0.10.0/24).
Type: LDAPS (Port 636)
Source: SOC-Lab-Enterprise-Subnet CIDR (10.0.10.0/24).
Type: Kerberos (Port 88)
Source: SOC-Lab-Enterprise-Subnet CIDR (10.0.10.0/24).
Type: DNS (Port 53)
Source: SOC-Lab-Enterprise-Subnet CIDR (10.0.10.0/24).
9.8 - Outbound rules: 
All traffic (0.0.0.0/0) to All (::/0) - refine later.

c. SOC-Lab-Win-Endpoint-SG

9.7 - Inbound rules:
Type: RDP (Port 3389)
Source: SOC-Lab-JumpBox-SG.
Type: DNS (Port 53)
Source: SOC-Lab-AD-DC-SG.
9.8 - Outbound rules: 
All traffic (0.0.0.0/0) to All (::/0) - refine later.

e. SOC-Lab-Wazuh-Manager-SG
Description: Security Group for Wazuh Manager components.
9.7 - Inbound rules:
Type: SSH (Port 22)
Source: SOC-Lab-JumpBox-SG.
Type: Custom TCP (Port 1514 - Wazuh agent registration)
Source: SOC-Lab-Enterprise-Subnet CIDR (10.0.10.0/24).
Type: Custom TCP (Port 55000 - Wazuh agent communication)
Source: SOC-Lab-Enterprise-Subnet CIDR (10.0.10.0/24).
Type: Custom TCP (Port 9200 - OpenSearch/Elasticsearch API)
Source: SOC-Lab-Tines-SG (you'll create this later).
Type: HTTPS (Port 443 - Kibana Web UI)
Source: SOC-Lab-JumpBox-SG.
9.8 - Outbound rules:
All traffic (0.0.0.0/0) to All (::/0) - refine later.

f. SOC-Lab-TheHive-SG

9.7 - Inbound rules:
Type: SSH (Port 22)
Source: SOC-Lab-JumpBox-SG.
Type: Custom TCP (Port 9000 - TheHive Web UI)
Source: SOC-Lab-JumpBox-SG and SOC-Lab-Tines-SG (for Tines to connect).
9.8 - Outbound rules:
All traffic (0.0.0.0/0) to All (::/0) - refine later.

g. SOC-Lab-Tines-SG 

9.7 - Inbound rules:
Type: SSH (Port 22)
Source: SOC-Lab-JumpBox-SG.
Type: HTTPS (Port 443 or specific Tines webhook port, e.g., 8080)
Source: SOC-Lab-Wazuh-Manager-SG.
9.8 - Outbound rules:
All traffic (0.0.0.0/0) to All (::/0) - refine later.

## Phase 4: Launching EC2 Instances (Servers) 🖥️
Now we'll launch your virtual servers in their respective subnets.
Create IAM Role for Wazuh Manager (for AWS Log Ingestion):
Go to IAM dashboard.
In the left navigation pane, click "Roles."
Click "Create role."
Trusted entity type: AWS service.
Use case: EC2. Click "Next."
Permissions policies: Search for and attach AmazonS3ReadOnlyAccess. (In a production environment, you'd create a custom policy allowing read access only to specific S3 buckets where logs are stored). Click "Next."
Role name: Wazuh-S3-Reader-Role
Click "Create role."

Launch EC2 Instances (Repeat for each type):

Go to EC2 dashboard.

Click "Launch instances."

a. Management Jump Box:

Name: Management-Jump-Box

AMI: Choose a Windows Server AMI (e.g., "Microsoft Windows Server 2022 Base"). This makes RDP easier for other Windows machines.

Instance type: t3.medium (or t3.small to save cost, but medium is more responsive).

Key pair: Select soc-lab-keypair.

Network settings:

VPC: Select SOC-Lab-VPC.

Subnet: Select SOC-Lab-Management-Subnet.

Auto-assign public IP: Enable (the Jump Box needs a public IP for you to connect initially).

Firewall (security groups): Select "Select an existing security group" and choose SOC-Lab-JumpBox-SG.

Storage: 30 GiB (default is usually fine).

Click "Launch instance."

b. Active Directory Domain Controller (AD DC):

Name: AD-DC

AMI: Windows Server AMI (e.g., "Microsoft Windows Server 2022 Base").

Instance type: t3.medium (AD requires a bit more resources).

Key pair: Select soc-lab-keypair.

Network settings:

VPC: Select SOC-Lab-VPC.

Subnet: Select SOC-Lab-Enterprise-Subnet.

Auto-assign public IP: Disable.

Firewall (security groups): Select SOC-Lab-AD-DC-SG.

Storage: 50 GiB (recommended for AD).

Click "Launch instance."

c. Windows Endpoints (x2):

Name: Win-Endpoint-01, then repeat for Win-Endpoint-02.

AMI: Windows Client AMI (e.g., "Windows 10/11" if available, or "Windows Server 2019/2022 Base" as a fallback).

Instance type: t3.small or t3.micro.

Key pair: Select soc-lab-keypair.

Network settings:

VPC: Select SOC-Lab-VPC.

Subnet: Select SOC-Lab-Enterprise-Subnet.

Auto-assign public IP: Disable.

Firewall (security groups): Select SOC-Lab-Win-Endpoint-SG.

Storage: 30 GiB.

Click "Launch instance."

d. Linux Server (Optional):

Name: Linux-Server

AMI: Ubuntu Server LTS (e.g., "Ubuntu Server 22.04 LTS").

Instance type: t3.micro or t3.small.

Key pair: Select soc-lab-keypair.

Network settings:

VPC: Select SOC-Lab-VPC.

Subnet: Select SOC-Lab-Enterprise-Subnet.

Auto-assign public IP: Disable.

Firewall (security groups): Select SOC-Lab-Lin-Server-SG.

Storage: 30 GiB.

Click "Launch instance."

e. Wazuh Manager:

Name: Wazuh-Manager

AMI: Ubuntu Server LTS (e.g., "Ubuntu Server 22.04 LTS").

Instance type: t3.large (Wazuh with OpenSearch needs decent RAM/CPU).

Key pair: Select soc-lab-keypair.

Network settings:

VPC: Select SOC-Lab-VPC.

Subnet: Select SOC-Lab-SOC-Tooling-Subnet.

Auto-assign public IP: Disable.

Firewall (security groups): Select SOC-Lab-Wazuh-Manager-SG.

Advanced details: IAM instance profile: Select Wazuh-S3-Reader-Role.

Storage: 100 GiB (for logs).

Click "Launch instance."

f. TheHive:

Name: TheHive

AMI: Ubuntu Server LTS.

Instance type: t3.large (for TheHive + its DB).

Key pair: Select soc-lab-keypair.

Network settings:

VPC: Select SOC-Lab-VPC.

Subnet: Select SOC-Lab-SOC-Tooling-Subnet.

Auto-assign public IP: Disable.

Firewall (security groups): Select SOC-Lab-TheHive-SG.

Storage: 60 GiB.

Click "Launch instance."

g. Tines (Self-Hosted - Optional/Advanced):

Name: Tines

AMI: Ubuntu Server LTS.

Instance type: t3.medium or t3.large (check Tines requirements).

Key pair: Select soc-lab-keypair.

Network settings:

VPC: Select SOC-Lab-VPC.

Subnet: Select SOC-Lab-SOC-Tooling-Subnet.

Auto-assign public IP: Disable.

Firewall (security groups): Select SOC-Lab-Tines-SG.

Storage: 30 GiB.

Click "Launch instance."

## Phase 5: Initial Access & Internal Networking 🔌
Now that your servers are running, we'll connect to them and set up basic internal communication.

Connect to Your Management Jump Box:

Go to EC2 dashboard -> "Instances."

Select your Management-Jump-Box instance.

For Windows Jump Box:

Click "Connect" -> "RDP client" -> "Get password."

Browse to your soc-lab-keypair.pem file, click "Decrypt password." Copy the password.

Download the Remote Desktop file and open it. Use the provided username (Administrator) and the decrypted password.

For Linux Jump Box:

Click "Connect" -> "SSH client."

Copy the example SSH command (e.g., ssh -i "soc-lab-keypair.pem" ec2-user@<public-ip-address>).

Open your terminal on your local machine, navigate to where your .pem file is, and paste/run the command.

Retrieve Private IP Addresses of All Instances:

While in the EC2 console (from your local machine), make a note of the Private IPv4 Address for every instance you just launched. You'll need these to connect from the Jump Box.

Connect from Jump Box to Private Instances:

From your Windows Jump Box: Use the Remote Desktop Connection application. Enter the Private IP Address of your AD-DC, Win-Endpoint-01, Win-Endpoint-02. Use the EC2 soc-lab-keypair.pem file to get their initial passwords (you'll set better ones later).

From your Linux Jump Box: Use the ssh command with the .pem key for Linux servers and potentially install an RDP client (like Remmina) for Windows servers.

Verify Connectivity: Try ping commands between instances (e.g., from Win-Endpoint-01 to AD-DC's private IP) to ensure basic network connectivity is working.

Phase 6: Core Software Installation & Configuration 💻
This is where you'll install the actual SOC tools and configure them to talk to each other.

Configure Active Directory Domain Controller (AD-DC):

Connect to AD-DC from Jump Box.

Open Server Manager.

Click "Add Roles and Features."

Select "Active Directory Domain Services" and "DNS Server."

After installation, promote the server to a Domain Controller.

Choose "Add a new forest."

Root domain name: soclab.local (or a similar internal domain).

Set a Directory Services Restore Mode (DSRM) password.

Follow the prompts to complete the promotion. The server will restart.

Crucial: Once AD is up, go to your Windows Endpoints and Linux Server (if Linux is part of the domain) and set their Primary DNS server to the private IP address of your AD-DC.

Join Windows Endpoints to the Domain: From each Win-Endpoint, go to System Properties -> Computer Name -> Change -> Domain, and join soclab.local. Restart.

Create a few test user accounts in Active Directory Users and Computers.

Install Wazuh Stack (Wazuh-Manager):

Connect to Wazuh-Manager (Linux) from Jump Box via SSH.

Follow the official Wazuh documentation for installing the Wazuh Manager, OpenSearch (or Elasticsearch), and Kibana/Dashboards. This usually involves adding Wazuh/OpenSearch repositories, installing packages, and configuring initial passwords.
*

Verify: Access the Kibana dashboard from your Jump Box's web browser (using the Wazuh-Manager's private IP, port 443/HTTPS) to confirm it's running. Log in with your configured credentials.

Install Wazuh Agents (AD-DC, Win-Endpoint-01, Win-Endpoint-02, Linux-Server):

On the Wazuh-Manager server, generate agent keys for each endpoint.

Connect to each endpoint from the Jump Box.

Follow the official Wazuh documentation for installing agents on Windows and Linux.

During installation, specify the private IP address of your Wazuh-Manager when prompted for the server IP.

Verify: Check the Wazuh Manager dashboard (Kibana) to see if agents are "Active."

Install Limacharlie Agents (AD-DC, Win-Endpoint-01, Win-Endpoint-02):

Sign up for a free Limacharlie account (limacharlie.io) and create an "Organization."

From the Limacharlie console, go to "Sensors" and get the installation command/script for Windows. It will include your organization ID and API key.

Connect to your Windows instances from the Jump Box.

Run the Limacharlie agent installation script (often a PowerShell command) on each Windows instance.

Verify: Check your Limacharlie dashboard to see if sensors are connecting and reporting.

Install TheHive (TheHive):

Connect to TheHive (Linux) from Jump Box via SSH.

Follow the official TheHive documentation for installation. This typically involves installing a Java Runtime Environment (JRE), setting up a database (like Elasticsearch or Cassandra), and then deploying TheHive.

Verify: Access TheHive web UI from your Jump Box's web browser (using TheHive's private IP, usually port 9000/HTTP or 443/HTTPS) to confirm it's running. Complete the initial setup wizard.

Set Up Tines


Connect to Tines (Linux) from Jump Box via SSH.

Follow the official Tines self-hosting documentation. This is more involved and might require Docker/Kubernetes.

Once running, identify your Tines webhook URL.

Configure AWS Logging Services (CloudTrail, VPC Flow Logs):

Go to CloudTrail in the AWS Console.

Ensure CloudTrail is enabled for all regions and logging to an S3 bucket. If you need to create a new trail, make sure it logs to a new S3 bucket (e.g., soc-lab-cloudtrail-logs-youruniqueid).

Go to VPC -> "VPC Flow Logs."

Click "Create flow log."

Filter: All.

Destination: Send to an S3 bucket. Choose to create a new S3 bucket (e.g., soc-lab-vpc-flow-logs-youruniqueid).

Click "Create."

IAM Role Adjustment: Go to IAM -> Roles -> Wazuh-S3-Reader-Role. Add permissions to read from these newly created S3 buckets for CloudTrail and VPC Flow Logs. Specifically, add s3:ListBucket and s3:GetObject on the specific resource ARNs of your new buckets.

Wazuh AWS Module Configuration:

Connect to Wazuh-Manager via SSH.

Edit the ossec.conf file (usually /var/ossec/etc/ossec.conf).

Add the AWS module configuration to pull logs from your CloudTrail and VPC Flow Logs S3 buckets. Refer to the Wazuh AWS Module documentation.

Restart the Wazuh Manager service: sudo systemctl restart wazuh-manager.

Verify: Check Wazuh Kibana dashboard for ingested CloudTrail and VPC Flow Logs.


<a href ="https://github.com/TIN0-m/Home-Lab/blob/main/Security%20groups%20and%20AWS%20services.md"><img src="https://img.shields.io/badge/-Next%20Section-FF0000?&style=for-the-badge&logoColor=white" /><a/>

