# Deployment Guide
##### This is a step by step guide to how the VPC was set up and all the components it has have been set up.

## AWS Account Preparation

1. #### If you don't have an AWS account, sign up at aws.amazon.com.
   
2. #### Select Your AWS Region:
In the AWS Management Console, look at the top right corner. Ensure you select "Africa (Cape Town) " if available and you prefer local. For this guide, I'll use eu-west-1. Consistency in region is important.

3. #### Create an EC2 Key Pair:

https://github.com/user-attachments/assets/f79bf1eb-f265-4621-87f2-0cf00a47c52e
   
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

https://github.com/user-attachments/assets/7af03112-cb34-4f10-a0c9-aae2de900bdd

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

https://github.com/user-attachments/assets/637a4806-e256-4996-9ca1-d40a8ca2967e

5.1 - In the VPC dashboard, click "Subnets."            
5.2 - Click "Create subnet."                
5.3 - Name tag: Public-Subnet       
5.4 - Availability Zone: Choose one (e.g., eu-west-1a).     
5.3 - IPv4 CIDR block: 10.0.1.0/24     

**For the rest of the subnets please ensure that the IPv4 CIDR blocks are set as follows:**
- Coperate Network : 10.0.10.0/24
- SOC Hub          : 10.0.20.0/24
- Management       : 10.0.30.0/24

6. ## Internet Gateway Setup

https://github.com/user-attachments/assets/2516cb86-9a75-4426-8474-07577421d3dd

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

### JumpBox Security Group        
    
9.5 - **Inbound rules:**         
- Type: RDP (Port 3389)            
- Source: "My IP".      
- Add another rule if using Linux Jump Box: Type: SSH (Port 22), Source: "My IP".             
  
9.6 - **Outbound rules:**       
- All traffic (0.0.0.0/0) to All (::/0) - this is temporary, we'll refine it later.             

### Active Directory Security Group         
9.5 - **Inbound rules:**       
- Type: RDP (Port 3389)        
- Source: JumpBox Security Group (select by SG ID).          
- Type: LDAP (Port 389)         
- Source: Coperate Subnet CIDR (10.0.10.0/24).         
- Type: LDAPS (Port 636)        
- Source: Coperate Subnet CIDR (10.0.10.0/24).       
- Type: Kerberos (Port 88)      
- Source: Coperate Subnet CIDR (10.0.10.0/24).     
- Type: DNS (Port 53)      
- Source: Coperate Subnet CIDR (10.0.10.0/24).     

9.6 - **Outbound rules:**      
- All traffic (0.0.0.0/0) to All (::/0) - refine later     

### Endpoint Security Group     

9.5 - **Inbound rules:**     
- Type: RDP (Port 3389)
- Source: JumpBox Security Group.       
- Type: DNS (Port 53)      
- Source: Active Directory Security group.        
  
9.6 - **Outbound rules:**        
-All traffic (0.0.0.0/0) to All (::/0) - refine later.      

### Wazuh Security Group    

9.5 - **Inbound rules:**       
- Type: SSH (Port 22)       
- Source: JumpBox Security Group.        
- Type: Custom TCP (Port 1514 - Wazuh agent registration)      
- Source: Coperate Subnet CIDR (10.0.10.0/24).        
- Type: Custom TCP (Port 55000 - Wazuh agent communication)     
- Source: Coperate Subnet CIDR (10.0.10.0/24).      
- Type: Custom TCP (Port 9200 - OpenSearch/Elasticsearch API)      
- Source: Tines Security Group (you'll create this later).     
- Type: HTTPS (Port 443 - Kibana Web UI)     
- Source: JumpBox Securrity Group.     
9.6 - **Outbound rules:**      
-All traffic (0.0.0.0/0) to All (::/0) - refine later.     

### The Hive Security Group     

9.5 - **Inbound rules:**      
- Type: SSH (Port 22)     
- Source: JumpBox Security Group.     
- Type: Custom TCP (Port 9000 - TheHive Web UI)    
- Source: JumpBox Security Group and Tines Security Group.    
9.6 - **Outbound rules:**    
- All traffic (0.0.0.0/0) to All (::/0) - refine later.    

### Tines Security Group     

9.5 - **Inbound rules:**     
- Type: SSH (Port 22)     
- Source: JumpBox Security Group.     
- Type: HTTPS (Port 443 or specific Tines webhook port, e.g., 8080)     
- Source: Wazuh Security Group.      
9.6 - **Outbound rules:**         
- All traffic (0.0.0.0/0) to All (::/0) - refine later.       

## Launching IAM Dashboard          

10.1 - Go to IAM dashboard.          
10.2 - In the left navigation pane, click "Roles."           
10.3 - Click "Create role."           
10.4 - Trusted entity type: AWS service.         
10.5 - Use case: EC2. Click "Next."        
10.6 - Permissions policies: Search for and attach AmazonS3ReadOnlyAccess. (In a production environment, you'd create a custom policy allowing read access only to specific S3 buckets where logs are stored). Click "Next."           
10.7 - Role name: Wazuh-S3-Reader-Role         
10.8 - Click "Create role."          

## Launch EC2 Instances      
11.1 - Go to EC2 dashboard.           
11.2 - Click "Launch instances."         

### Management Jump Box:              
11.3 - Name: Management-Jump-Box           
11.4 - AMI: Windows Server AMI          
11.5 - Instance type: t3.medium         
11.6 - Key pair: lab-keypair.          

**Network settings:**
11.7 - VPC: Lab VPC.      
11.8 - Subnet: Management Subnet.       
11.9 - Auto-assign public IP: Enable.        
11.10 - Firewall (security groups): Select an existing security group and choose Jumpbox Security Group.        

11.11 - Storage: 30 GiB      
11.12 - Click "Launch instance."        

### Active Directory                
11.3 - Name: Active Directory
11.4 - AMI: Windows Server AMI.      
11.5 - Instance type: t3.medium        
11.6 - Key pair: lab-keypair.        

**Network settings:**         
11.7 - VPC: Lab VPC.         
11.8 - Subnet: Coperate Subnet.       
11.9 - Auto-assign public IP: Disable.          
11.10 - Firewall (security groups): Select an existing security group and choose Active Directory Security Group.          
11.11 - Storage: 50 GiB       
11.12 - Click "Launch instance."          

### Windows Endpoints (x2):
11.3 - Name: Win-Endpoint-01, then repeat for Win-Endpoint-02.          
11.4 - AMI: Windows Client AMI.         
11.5 - Instance type: t3.small.          
11.6 - Key pair: lab-keypair.         

**Network settings:**         
11.7 - VPC: Lab VPC.            
11.8 - Subnet: Coperate Subnet.          
11.9 - Auto-assign public IP: Disable.          
11.10 - Firewall (security groups): Select an existing security group and choose Endpoint Security Group.           
11.11 - Storage: 30 GiB.           
11.12 - Click "Launch instance."           

### Wazuh Manager:       
11.3 - Name: Wazuh-Manager         
11.4 - AMI: Ubuntu Server LTS          
11.5 - Instance type: t3.large            
11.6 - Key pair: Select soc-lab-keypair.       

**Network settings:**            
11.7 - VPC: Lab VPC.           
11.8 - Subnet: SOC hub Subnet.         
11.9 - Auto-assign public IP: Disable.          
Firewall (security groups): Select an existing security group choose Wazuh Security Group.            

**Advanced details:**
11.10 - IAM instance profile: Select Wazuh-S3-Reader-Role.         
11.11 - Storage: 100 GiB (for logs).        
11.11 - Click "Launch instance."       

### TheHive:
11.3 - Name: TheHive            
11.4 - AMI: Ubuntu Server LTS.        
11.5 - Instance type: t3.large.         
11.6 - Key pair: lab-keypair.          

**Network settings:**            
11.7 - VPC: Lab VPC.          
11.8 - Subnet: SOC Hub Subnet.          
11.9 - Auto-assign public IP: Disable.            
11.10 - Firewall (security groups): Select an existing security group and choose The Hive Security Group.         
11.11 - Storage: 60 GiB.       
11.12 - Click "Launch instance."          

### Tines       
11.3 - Name: Tines      
11.4 - AMI: Ubuntu Server LTS.       
11.5 - Instance type: t3.medium     
11.6 - Key pair: lab-keypair.         

**Network settings:**        
11.7 - VPC: Lab VPC.       
11.8 - Subnet: SOC hub Subnet.     
11.9 - Auto-assign public IP: Disable.     
11.10 - Firewall (security groups): Select an existing security group and choose Tines Security Group.         
11.11 - Storage: 30 GiB.     
11.12 - Click "Launch instance."     

## Initial Access and Internal Networking 

12.1 - Connect to Your Management Jump Box:    
12.2 - Go to EC2 dashboard -> "Instances."     
12.3 - Select your Management-Jump-Box instance. 

**For Windows Jump Box:**     
12.4 - Click "Connect" -> "RDP client" -> "Get password."     
12.5 - Browse to your soc-lab-keypair.pem file, click "Decrypt password." Copy the password.     
12.6 - Download the Remote Desktop file and open it. Use the provided username (Administrator) and the decrypted password.    

**For Linux Jump Box:**     
12.7 - Click "Connect" -> "SSH client."      
12.8 - Copy the example SSH command (e.g., ssh -i "soc-lab-keypair.pem" ec2-user@<public-ip-address>).      
12.9 - Open your terminal on your local machine, navigate to where your .pem file is, and paste/run the command.      
12.10 - Retrieve Private IP Addresses of All Instances:      
12.11 - While in the EC2 console (from your local machine), make a note of the Private IPv4 Address for every instance you just launched. You'll need these to connect from the Jump Box.       

### Connect from Jump Box to Private Instances:        
12.12 -  **From your Windows Jump Box:**     
12.12.1 - Use the Remote Desktop Connection application.      
12.12.2 - Enter the Private IP Address of your AD-DC, Win-Endpoint-01, Win-Endpoint-02.      
12.12.3 - Use the EC2 soc-lab-keypair.pem file to get their initial passwords.       

12.13 - **From your Linux Jump Box:**      
12.13.1 - Use the ssh command with the .pem key for Linux servers.      
12.13.2 - Verify Connectivity: Try ping commands between instances to ensure basic network connectivity is working.        

## Software Installation and Configuration.

13.1 - Configure Active Directory:     
13.2 - Connect to AD-DC from Jump Box.      
13.3 - Open Server Manager.      
13.4 - Click "Add Roles and Features."      
13.5 - Select "Active Directory Domain Services" and "DNS Server."        
13.6 - After installation, promote the server to a Domain Controller.       
13.7 - Choose "Add a new forest."       
13.8 - Root domain name: soclab.local (or a similar internal domain).          
13.9 - Set a Directory Services Restore Mode (DSRM) password.       

**Follow the prompts to complete the promotion. The server will restart.**         

**Join Windows Endpoints to the Domain:**              
13.10 - From each Win-Endpoint, go to System Properties -> Computer Name -> Change -> Domain, and join soclab.local. Restart.      
13.11 - Install Wazuh Stack (Wazuh-Manager):           
13.12 - Connect to Wazuh-Manager (Linux) from Jump Box via SSH.         
**Follow the official Wazuh documentation for installing the Wazuh Manager, OpenSearch (or Elasticsearch), and Kibana/Dashboards.**         

13.13 - Install Wazuh Agents (AD-DC, Win-Endpoint-01, Win-Endpoint-02, Linux-Server):        
13.14 - On the Wazuh-Manager server, generate agent keys for each endpoint.        
13.15 - Connect to each endpoint from the Jump Box.        

**Install Limacharlie Agents (AD-DC, Win-Endpoint-01, Win-Endpoint-02):**         
13.16 - Sign up for a free Limacharlie account (limacharlie.io) and create an "Organization."          
13.17 - From the Limacharlie console, go to "Sensors" and get the installation command/script for Windows. It will include your organization ID and API key.       
13.18 - Connect to your Windows instances from the Jump Box.          
13.19 - Run the Limacharlie agent installation script (often a PowerShell command) on each Windows instance.          

**Install TheHive (TheHive):**          
13.20 - Connect to TheHive (Linux) from Jump Box via SSH.            
**Follow the official TheHive documentation for installation. This typically involves installing a Java Runtime Environment (JRE), setting up a database (like Elasticsearch or Cassandra), and then deploying TheHive**        

**Set Up Tines**           
13.21 - Connect to Tines (Linux) from Jump Box via SSH.             
**Follow the official Tines self-hosting documentation. This is more involved and might require Docker/Kubernetes.**           
13.22 - Once running, identify your Tines webhook URL.           

<a href ="https://github.com/TIN0-m/Home-Lab/blob/main/Security%20groups%20and%20AWS%20services.md"><img src="https://img.shields.io/badge/-Next%20Section-FF0000?&style=for-the-badge&logoColor=white" /><a/>
