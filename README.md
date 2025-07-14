# Home-Lab
personal sandbox for mastering the entire incident response lifecycle, from the first byte of telemetry to automated containment and meticulous case management.

# Table of Contents
1. Introduction

2. Architecture Overview

3. Key Components

4. Deployment Guide

5. Usage & Scenarios

# 1. Introduction 

Purpose:     

This repository details the architecture and deployment steps for an AWS-based Security Operations Center (SOC) lab. Its purpose is to provide a practical, hands-on environment to simulate cyberattacks, detect malicious activity, and practice incident response workflows using a suite of open-source and cloud-native security tools. It's my personal sandbox for mastering the entire incident response lifecycle, from the first byte of telemetry to automated containment and meticulous case management. 

# Objectives:  

- To bridge the gap between theoretical cybersecurity knowledge and tangible, hands-on experience in a real-world cloud environment.

- To demonstrate proficiency in designing, deploying, and securing complex cloud infrastructures (AWS VPC).

- To showcase practical skills in Endpoint Detection & Response (EDR), Security Information & Event Management (SIEM), Security Orchestration, Automation, & Response (SOAR), and Incident Response Platform (IRP) integration.

- To cultivate a proactive, blue-team mindset by simulating adversary tactics (Red Team) and building robust defenses (Blue Team).

- To translate theoretical understanding of frameworks like MITRE ATT&CK into actionable detections and automated responses.

# Skills : Detect. Automate. Respond. Analyze.
This lab provides opportunity to:

- Simulate Realistic Attacks: Execute Atomic Red Team playbooks against live Windows endpoints, mirroring sophisticated adversary techniques.

- Achieve Unprecedented Visibility: Collect granular endpoint telemetry with Limacharlie, comprehensive system and cloud logs with Wazuh, and network flow data with AWS VPC Flow Logs.

- Automate Tier 1/2 SOC Functions: Leverage Tines to orchestrate real-time alert triage, threat intelligence enrichment, and automated containment actions against Active Directory or endpoints.

- Master Incident Response: Utilize TheHive for collaborative incident management, from initial alert ingestion to investigation, remediation, and post-mortem analysis.

- Fortify Cloud Security Posture: Integrate AWS CloudTrail and GuardDuty findings into the SIEM, providing holistic visibility across the cloud control plane.

# 2. Architecture Overview

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

<img width="801" height="541" alt="FINAL" src="https://github.com/user-attachments/assets/a0a66fe6-08fb-4ddc-9a69-889478cebd28" />

# public-subnet (10.0.1.0/24): 

<img width="801" height="431" alt="Public Subnet" src="https://github.com/user-attachments/assets/4248011d-0aed-4e0b-a577-fc83e278ea88" />
Purpose: Houses components requiring direct internet access.

Components: 

- **Routing Table:**      
Explicitly routes 0.0.0.0/0 (all internet traffic) to the Internet Gateway (IGW).
  
- **Internet Gateway:**          
The VPC's "front door" to the internet. Attached directly to the VPC, it allows public traffic in and out.

- **NAT Gateway:**    
The controlled egress point for private subnets. Instances in private subnets cannot initiate direct outbound internet connections; they must route through here


# Private subnet 1: Corporate Network 
<img width="631" height="381" alt="Enterprise Subnet" src="https://github.com/user-attachments/assets/ded38939-50fd-4341-99ae-24d4de7ad092" />
Purpose: Hosts the assets that mimic a typical enterprise environment – your "victims."

Route Table: Routes 0.0.0.0/0 to the NAT Gateway (NAT GW) for all outbound internet access.

Components (EC2 Instances):

Active Directory Domain Controller (AD DC):

OS: Windows Server (e.g., 2022).

Roles: Core identity and access management: Active Directory Domain Services (AD DS) and DNS Server. The backbone of the simulated enterprise.

Agents: Instrumented with Wazuh Agent and Limacharlie Agent for deep visibility into AD activities.

Security Group (SOC-Lab-AD-DC-SG): Inbound: RDP (3389) from Jump Box, LDAP/S (389, 636), Kerberos (88), DNS (53) from other Enterprise Network SGs and SOC Tooling SGs. Outbound: All traffic to Enterprise and SOC Tooling SGs, and NAT GW.

Windows Endpoints (2x):

OS: Windows Client (e.g., Windows 10/11).

Domain: Authenticated members of the AD domain. These are your primary targets for Atomic Red Team simulations.

Agents: Wazuh Agent (for system logs, event logs) and Limacharlie Agent (for granular process, network, and file telemetry) installed.

Sysmon (Optional): Deployed for additional, rich Windows event logging if Limacharlie's native telemetry isn't sufficient for specific detection needs (though Limacharlie often supersedes it).

Security Group (SOC-Lab-Win-Endpoint-SG): Inbound: RDP (3389) from Jump Box, DNS (53) from AD DC. Outbound: To AD DC, Wazuh Manager, and NAT GW.



# Private subnet 2: The Security Operations Center Hub 
<img width="761" height="251" alt="SOC Hub" src="https://github.com/user-attachments/assets/37a53847-97b4-4da1-bf73-1e34f947eced" />

Purpose: Dedicated to hosting the core security tools, isolated from the simulated enterprise.

Route Table: Routes 0.0.0.0/0 to the NAT Gateway (NAT GW) for all outbound internet access (e.g., pulling threat intelligence, updates).

Components (EC2 Instances):

Wazuh Manager/Server:

OS: Linux (e.g., Ubuntu LTS).

Components: Wazuh Manager (brain), OpenSearch/Elasticsearch (data lake), Kibana/Dashboards (visualization & alerting UI).

Security Group (SOC-Lab-Wazuh-Manager-SG): Inbound: Wazuh Agent communication (TCP 1514 - agent enrollment/data, 55000 - agent registration), SSH (22) from Jump Box, Elasticsearch/OpenSearch API (9200) from Tines, Kibana (443) from Jump Box. Outbound: To Tines (for alerts), and NAT GW.

TheHive:

OS: Linux.

Components: TheHive application, its database (e.g., Elasticsearch/Cassandra), and potentially Cortex (for automated analysis of observables).

Security Group (SOC-Lab-TheHive-SG): Inbound: SSH (22) from Jump Box, Web UI (9000/443) from Jump Box/Tines. Outbound: To Tines (if Tines pulls cases), and NAT GW.

Tines:

OS: Linux (if self-hosted).

Components: The Tines SOAR platform itself.

Key Detail: While self-hosting is possible (as depicted), for a lab environment, utilizing a Tines Cloud Free/Trial account significantly simplifies deployment and management, allowing you to focus on automation logic. Your agents in AWS would communicate with the cloud instance via the NAT Gateway.

Security Group (SOC-Lab-Tines-SG): (For self-hosted) Inbound: SSH (22) from Jump Box, Webhook/API ingress (443/8080) from Wazuh Manager and Limacharlie Cloud Platform. Outbound: To TheHive, AD DC, and NAT GW (for external APIs).

Limacharlie Cloud Platform (SaaS):

Purpose: This is a cloud-native, managed service. You DO NOT host this.

Interaction: Your Limacharlie agents on your EC2 instances connect securely to your dedicated Limacharlie tenant over the internet (via the NAT Gateway). You configure Limacharlie to send high-fidelity endpoint alerts via webhooks out to your Tines instance.


