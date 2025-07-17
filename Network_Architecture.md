# Architecture Overview

My SOC lab resides within a meticulously crafted AWS Virtual Private Cloud (VPC), engineered for security, isolation, and scalability. This isn't just a collection of VMs; it's a production-ready network design for a secure enterprise.
Key Architectural Principles Applied:

- **Principle of Least Privilege:**        
 Granular IAM roles and stringent Security Group rules enforce minimal necessary access.

- **Network Segmentation:**       
 Logical isolation via private subnets to contain potential breaches and control traffic flow.

- **Defense in Depth:**     
Multiple layers of security controls, from network ACLs to host-based agents.
 
- **Observability:**      
 Comprehensive logging of all cloud control plane and data plane activities.

# Network Components :

AWS VPC
soc-lab-vpc (10.0.0.0/16): This foundational block dictates the entire IP addressing scheme. It's where every resource lives, meticulously controlled.

Default DHCP Options Set: Essential for DNS resolution and domain information, automatically assigned.

Default Network ACL (NACL): A stateless firewall at the subnet level. While default allows all, custom NACLs provide an additional, optional layer of ingress/egress filtering, complementing Security Groups.

3.2. Availability Zones (AZs) (Resilience & Redundancy)
For this lab, resources are primarily deployed within a single AZ for cost-efficiency and simplified initial setup (e.g., eu-west-1a). 

<img width="801" height="541" alt="FINAL" src="https://github.com/user-attachments/assets/1545f8b7-19bb-413c-a36b-921117602e9c" />

# public-subnet (10.0.1.0/24): 

<img width="801" height="431" alt="Public Subnet" src="https://github.com/user-attachments/assets/7d60aec5-9367-4f84-afb9-b70eaaa2a6c8" />

**Purpose:**   
Houses components requiring direct internet access.

**Components:** 

- **Routing Table:**      
Explicitly routes 0.0.0.0/0 (all internet traffic) to the Internet Gateway (IGW).
  
- **Internet Gateway:**          
The VPC's "front door" to the internet. Attached directly to the VPC, it allows public traffic in and out.

- **NAT Gateway:**    
The controlled egress point for private subnets. Instances in private subnets cannot initiate direct outbound internet connections; they must route through here


# Private subnet 1: Corporate Network 

<img width="631" height="381" alt="Enterprise Subnet" src="https://github.com/user-attachments/assets/9b4e856b-743a-47d5-9ef6-4a83f1bd2a3a" />

**Purpose:**     
Hosts the assets that are found in a typical enterprise environment 

Route Table: Routes 0.0.0.0/0 to the NAT Gateway (NAT GW) for all outbound internet access.

**Components:**       

**Active Directory Domain Controller :**       
- OS: Windows Server 
- Roles:
 Core identity and access management (Active Directory Domain Services and DNS Server).
- Agents:
Instrumented with Wazuh Agent and Limacharlie Agent for deep visibility into AD activities.
- Security Group (SOC-Lab-AD-DC-SG): Inbound: RDP (3389) from Jump Box, LDAP/S (389, 636), Kerberos (88), DNS (53) from other Enterprise Network SGs and SOC Tooling SGs. Outbound: All traffic to Enterprise and SOC Tooling SGs, and NAT GW.

**Windows Endpoints (2x):**     
- OS: Windows Client (e.g., Windows 10/11).
- Domain:      
 Authenticated members of the AD domain
- Agents:      
 Wazuh Agent (event logs) and Limacharlie Agent ( process, network, and file telemetry) installed.
-Security Group (SOC-Lab-Win-Endpoint-SG): Inbound: RDP (3389) from Jump Box, DNS (53) from AD DC. Outbound: To AD DC, Wazuh Manager, and NAT GW.



# Private subnet 2: The Security Operations Center Hub 

<img width="761" height="251" alt="SOC Hub" src="https://github.com/user-attachments/assets/24222d32-f2ef-46d6-8816-c59f875ade58" />

**Purpose:**      
Dedicated to hosting the core security tools, isolated from the simulated enterprise.
Route Table: Routes 0.0.0.0/0 to the NAT Gateway (NAT GW) for all outbound internet access (e.g., pulling threat intelligence, updates).

**Components :**   

 **Wazuh Manager:**
- OS: Linux.
- Components: Wazuh Manager (brain), OpenSearch/Elasticsearch (data lake), Kibana/Dashboards (visualization & alerting UI).
-Security Group (SOC-Lab-Wazuh-Manager-SG): Inbound: Wazuh Agent communication (TCP 1514 - agent enrollment/data, 55000 - agent registration), SSH (22) from Jump Box, Elasticsearch/OpenSearch API (9200) from Tines, Kibana (443) from Jump Box. Outbound: To Tines (for alerts), and NAT GW.

**TheHive:**
- OS: Linux.
- Components:
 TheHive application, its database (e.g., Elasticsearch/Cassandra), and potentially Cortex (for automated analysis of observables).
- Security Group (SOC-Lab-TheHive-SG): Inbound: SSH (22) from Jump Box, Web UI (9000/443) from Jump Box/Tines. Outbound: To Tines (if Tines pulls cases), and NAT GW.

**Tines:**   
- OS: Linux.
- Components: The Tines SOAR platform itself.
- Security Group (SOC-Lab-Tines-SG): (For self-hosted) Inbound: SSH (22) from Jump Box, Webhook/API ingress (443/8080) from Wazuh Manager and Limacharlie Cloud Platform. Outbound: To TheHive, AD DC, and NAT GW (for external APIs).

**Limacharlie Cloud Platform:**    
-OS : SaaS
Interaction: Your Limacharlie agents on your EC2 instances connect securely to your dedicated Limacharlie tenant over the internet (via the NAT Gateway). You configure Limacharlie to send high-fidelity endpoint alerts via webhooks out to your Tines instance.

# Private Subnet 3 : Management

<img width="481" height="241" alt="Management" src="https://github.com/user-attachments/assets/96285c0f-fd5d-4b4a-bc64-6f0652b01365" />

**Purpose:** 
- A highly restricted subnet for administrative access.
- Route Table: Routes 0.0.0.0/0 to the NAT Gateway (NAT GW) for all outbound internet access.

**Component,Management Jump Box:**
- OS: Windows Server (for RDP to other Windows instances) or Linux (for SSH, with RDP client).
- Single, secure entry point into the private network. This drastically reduces internet-facing attack surface.
-Security Group (SOC-Lab-JumpBox-SG): EXTREMELY RESTRICTIVE. Inbound: Only allows SSH (22) or RDP (3389) from YOUR SPECIFIC PUBLIC IP ADDRESS (X.X.X.X/32). Outbound: RDP (3389) to Windows Endpoints/AD DC, SSH (22) to Linux Server/Wazuh/TheHive/Tines.

<a href ="https://github.com/TIN0-m/Home-Lab/blob/main/Logical-Flow.md"><img src="https://img.shields.io/badge/-Next%20Section-FF0000?&style=for-the-badge&logoColor=white" /><a/>
