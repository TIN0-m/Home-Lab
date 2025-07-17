Phase 1: AWS Account Preparation ⚙️
Sign Up for AWS & Set Up Billing Alarms (Crucial!):

If you don't have an AWS account, sign up at aws.amazon.com.

Set up a billing alarm: Go to CloudWatch -> Alarms -> Create alarm. Select "Billing" metric "EstimatedCharges" and set a threshold (e.g., $5 USD). Configure it to notify you via email or SMS. This is your primary safeguard against unexpected costs.

Select Your AWS Region:

In the AWS Management Console, look at the top right corner. Ensure you select "Europe (Frankfurt) eu-central-1" or "Africa (Cape Town) eu-west-1" if available and you prefer local. For this guide, I'll use eu-west-1. Consistency in region is important.

Create an EC2 Key Pair:

Go to EC2 dashboard.

In the left navigation pane, under "Network & Security," click "Key Pairs."

Click "Create key pair."

Name: soc-lab-keypair

Key pair type: RSA

Private key file format: .pem

Click "Create key pair."

Download the .pem file immediately. Keep it secure; you'll need it to connect to your instances via SSH/RDP.

Phase 2: AWS VPC Network Infrastructure Setup 🌐
This is the backbone of your lab. We'll build the VPC, subnets, route tables, and NAT Gateway.

Create Your Virtual Private Cloud (VPC):

Go to the VPC dashboard.

In the left navigation pane, click "Your VPCs."

Click "Create VPC."

Name tag: SOC-Lab-VPC

IPv4 CIDR block: 10.0.0.0/16 (This gives you a large, private IP range for all your resources).

Leave other settings as default.

Click "Create VPC."

Create Subnets:

In the VPC dashboard, click "Subnets."

Click "Create subnet."

Repeat this process four times for the following subnets. Ensure you select your SOC-Lab-VPC for each.

Name tag: SOC-Lab-Public-Subnet

Availability Zone: Choose one (e.g., eu-west-1a).

IPv4 CIDR block: 10.0.1.0/24

Name tag: SOC-Lab-Enterprise-Subnet

Availability Zone: Use the same AZ as your Public Subnet (e.g., eu-west-1a) for simplicity in a lab.

IPv4 CIDR block: 10.0.10.0/24

Name tag: SOC-Lab-SOC-Tooling-Subnet

Availability Zone: Same AZ (e.g., eu-west-1a).

IPv4 CIDR block: 10.0.20.0/24

Name tag: SOC-Lab-Management-Subnet

Availability Zone: Same AZ (e.g., eu-west-1a).

IPv4 CIDR block: 10.0.30.0/24

Click "Create subnet" after configuring each one.

Create and Attach an Internet Gateway (IGW):

In the VPC dashboard, click "Internet Gateways."

Click "Create internet gateway."

Name tag: SOC-Lab-IGW

Click "Create internet gateway."

Once created, select it and click "Actions" -> "Attach to VPC."

Select your SOC-Lab-VPC from the dropdown.

Click "Attach internet gateway."

Create Route Tables:

In the VPC dashboard, click "Route Tables."

a. Create Public Route Table:

Click "Create route table."

Name tag: SOC-Lab-Public-RT

VPC: Select SOC-Lab-VPC.

Click "Create route table."

Select SOC-Lab-Public-RT. Go to the "Routes" tab -> "Edit routes."

Click "Add route."

Destination: 0.0.0.0/0 (this means "all internet traffic")

Target: Select "Internet Gateway" and choose your SOC-Lab-IGW.

Click "Save changes."

Go to the "Subnet associations" tab -> "Edit subnet associations."

Select SOC-Lab-Public-Subnet.

Click "Save associations."

b. Create Private Route Table:

Click "Create route table."

Name tag: SOC-Lab-Private-RT

VPC: Select SOC-Lab-VPC.

Click "Create route table."

Do NOT add the 0.0.0.0/0 route yet. We'll do that after the NAT Gateway.

Go to the "Subnet associations" tab -> "Edit subnet associations."

Select SOC-Lab-Enterprise-Subnet, SOC-Lab-SOC-Tooling-Subnet, and SOC-Lab-Management-Subnet.

Click "Save associations."

Create NAT Gateway:

In the VPC dashboard, click "NAT Gateways."

Click "Create NAT gateway."

Name tag: SOC-Lab-NAT-GW

Subnet: Select SOC-Lab-Public-Subnet (the NAT GW must be in a public subnet).

Connectivity Type: Public.

Elastic IP allocation ID: Click "Allocate an Elastic IP address" (this gives it a static public IP).

Click "Create NAT gateway." This might take a few minutes to become available.

Update Private Route Table to Use NAT Gateway:

Once the NAT Gateway status is "Available" (check in the NAT Gateway section), go back to "Route Tables" in the VPC dashboard.

Select SOC-Lab-Private-RT. Go to the "Routes" tab -> "Edit routes."

Click "Add route."

Destination: 0.0.0.0/0

Target: Select "NAT Gateway" and choose your SOC-Lab-NAT-GW.

Click "Save changes."

Phase 3: AWS Security Groups (Firewalls for Instances) 🔒
We'll create the necessary Security Groups (SGs) to control traffic to and from your EC2 instances.

Create Security Groups:

Go to the EC2 dashboard.

In the left navigation pane, under "Network & Security," click "Security Groups."

Click "Create security group."

Repeat this for each SG listed below. Ensure you select your SOC-Lab-VPC for each.

a. SOC-Lab-JumpBox-SG (Critical for Initial Access)

Description: Allows secure RDP/SSH access to the Jump Box.

Inbound rules:

Type: RDP (Port 3389)

Source: "My IP" (AWS will auto-detect your current public IP. If your IP changes, you'll need to update this, or use a CIDR for your home network).

Add another rule if using Linux Jump Box: Type: SSH (Port 22), Source: "My IP".

Outbound rules: All traffic (0.0.0.0/0) to All (::/0) - this is temporary, we'll refine it later.

b. SOC-Lab-AD-DC-SG

Description: Security Group for Active Directory Domain Controller.

Inbound rules:

Type: RDP (Port 3389), Source: SOC-Lab-JumpBox-SG (select by SG ID).

Type: LDAP (Port 389), Source: SOC-Lab-Enterprise-Subnet CIDR (10.0.10.0/24).

Type: LDAPS (Port 636), Source: SOC-Lab-Enterprise-Subnet CIDR (10.0.10.0/24).

Type: Kerberos (Port 88), Source: SOC-Lab-Enterprise-Subnet CIDR (10.0.10.0/24).

Type: DNS (Port 53), Source: SOC-Lab-Enterprise-Subnet CIDR (10.0.10.0/24).

Later, you'll add rules from SOC Tooling SGs if they need to query AD.

Outbound rules: All traffic (0.0.0.0/0) to All (::/0) - refine later.

c. SOC-Lab-Win-Endpoint-SG

Description: Security Group for Windows Endpoints.

Inbound rules:

Type: RDP (Port 3389), Source: SOC-Lab-JumpBox-SG.

Type: DNS (Port 53), Source: SOC-Lab-AD-DC-SG.

Outbound rules: All traffic (0.0.0.0/0) to All (::/0) - refine later.

d. SOC-Lab-Lin-Server-SG (If including Linux Server)

Description: Security Group for Linux Server.

Inbound rules:

Type: SSH (Port 22), Source: SOC-Lab-JumpBox-SG.

Outbound rules: All traffic (0.0.0.0/0) to All (::/0) - refine later.

e. SOC-Lab-Wazuh-Manager-SG

Description: Security Group for Wazuh Manager components.

Inbound rules:

Type: SSH (Port 22), Source: SOC-Lab-JumpBox-SG.

Type: Custom TCP (Port 1514 - Wazuh agent registration), Source: SOC-Lab-Enterprise-Subnet CIDR (10.0.10.0/24).

Type: Custom TCP (Port 55000 - Wazuh agent communication), Source: SOC-Lab-Enterprise-Subnet CIDR (10.0.10.0/24).

Type: Custom TCP (Port 9200 - OpenSearch/Elasticsearch API), Source: SOC-Lab-Tines-SG (you'll create this later).

Type: HTTPS (Port 443 - Kibana Web UI), Source: SOC-Lab-JumpBox-SG.

Outbound rules: All traffic (0.0.0.0/0) to All (::/0) - refine later.

f. SOC-Lab-TheHive-SG

Description: Security Group for TheHive.

Inbound rules:

Type: SSH (Port 22), Source: SOC-Lab-JumpBox-SG.

Type: Custom TCP (Port 9000 - TheHive Web UI), Source: SOC-Lab-JumpBox-SG and SOC-Lab-Tines-SG (for Tines to connect).

Outbound rules: All traffic (0.0.0.0/0) to All (::/0) - refine later.

g. SOC-Lab-Tines-SG (If self-hosting Tines)

Description: Security Group for Tines.

Inbound rules:

Type: SSH (Port 22), Source: SOC-Lab-JumpBox-SG.

Type: HTTPS (Port 443 or specific Tines webhook port, e.g., 8080), Source: SOC-Lab-Wazuh-Manager-SG.

If using Limacharlie Cloud Platform's direct webhook, you might need to allow HTTPS from 0.0.0.0/0 on a specific Tines webhook port, but this is less secure. Better to route through an authenticated proxy or use Tines cloud which handles this.

Outbound rules: All traffic (0.0.0.0/0) to All (::/0) - refine later.

Phase 4: Launching EC2 Instances (Servers) 🖥️
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

Phase 5: Initial Access & Internal Networking 🔌
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

Set Up Tines (Cloud or Self-Hosted):

Option A (Recommended - Tines Cloud):

Go to tines.com and sign up for their free/trial cloud instance.

Familiarize yourself with creating "Stories" and "Agents."

Get the Webhook URL for an incoming HTTP Agent in Tines; you'll use this for integration.

Option B (Self-Hosted - Advanced):

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

Phase 7: SOC Tool Integration & Automation 🔗
This is where the magic happens – connecting your tools for a seamless incident response workflow.

Wazuh to Tines Alerting:

Connect to Wazuh-Manager via SSH.

Configure the Wazuh webhook integration. Add a new integration block in ossec.conf that points to your Tines webhook URL. You'll need to specify what alerts (e.g., level 10 and higher) should be sent.

XML

<integration>
  <name>custom-webhook</name>
  <level>10</level>
  <api_key>YOUR_TINES_API_KEY_IF_NEEDED</api_key> <url>YOUR_TINES_WEBHOOK_URL</url>
  <extra_args>
    <alert_format>json</alert_format>
  </extra_args>
</integration>
Restart Wazuh Manager.

Limacharlie to Tines Alerting:

Log into your Limacharlie Cloud Platform console.

Go to "Detections" or "Outputs."

Create a new output that sends alerts to a Webhook.

Paste your Tines webhook URL here. Configure which types of detections you want to forward (e.g., high-fidelity detections).

Tines Playbooks (Orchestration):

Log into your Tines instance/cloud console.

Create a new "Story" (Playbook).

Agent 1: Incoming Webhook (HTTP Agent): Configure this agent to listen for alerts from Wazuh and Limacharlie.

Agent 2: TheHive Create Case:

Add an action to call TheHive's API to create a new case.

You'll need TheHive's API key (generated in TheHive) and its private IP/URL.

Map the incoming alert data from Wazuh/Limacharlie to TheHive case fields (title, description, observables).

Optional Agents:

Threat Intelligence Enrichment: Add an agent to query VirusTotal (requires a VirusTotal API key) for hashes/IPs from the alert.

Active Directory Response: Add an agent (e.g., SSH Agent or PowerShell Remoting Agent if Tines supports it directly or via a proxy) to connect to your AD-DC and execute commands (e.g., disable user suspect_user). Ensure Tines has necessary credentials securely stored.

Limacharlie Response: Add an agent to call Limacharlie's API to perform actions like isolate_host on a compromised endpoint.

TheHive Configuration:

Log into TheHive web UI.

Configure Responders if you want to integrate Cortex (for automated analysis) or other external tools.

Familiarize yourself with creating cases, adding observables, and managing tasks.

Phase 8: Attack Simulation & Validation 😈
Time to test your SOC!

Prepare Atomic Red Team:

Connect to your Win-Endpoint-01 or Win-Endpoint-02 from Jump Box.

Install the Atomic Red Team PowerShell module.

Download the Atomic Red Team atoms (Invoke-AtomicTest).

Do not run everything at once! Start small.

Run a Simple Atomic Test:

Example: Execute a simple command to create a suspicious file or run a common PowerShell technique.

PowerShell

# Example: T1059.001 - PowerShell Command Line
Invoke-AtomicTest T1059.001 -ShowDetailsBrief -CheckPrereqs -GetPrereqs -Execute
Validate Detections:

Immediately check your Limacharlie dashboard for new alerts from the endpoint.

Check your Wazuh Kibana dashboard for new alerts.

Verify Alert Flow to Tines: Log into Tines and check the "Events" or "Activity" feed to confirm it received the alerts from Limacharlie and Wazuh.

Verify Case Creation in TheHive: Log into TheHive and confirm a new case was created by Tines based on the alert.

Test Response Actions:

If you configured an automated response (e.g., "disable user in AD" or "isolate host in Limacharlie"), trigger that action in Tines (or manually simulate it).

Verify the outcome (e.g., log into AD to confirm user status, check Limacharlie console for host isolation).


<a href ="https://github.com/TIN0-m/Home-Lab/blob/main/Security%20groups%20and%20AWS%20services.md"><img src="https://img.shields.io/badge/-Next%20Section-FF0000?&style=for-the-badge&logoColor=white" /><a/>

