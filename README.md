# Home-Lab
personal sandbox for mastering the entire incident response lifecycle, from the first byte of telemetry to automated containment and meticulous case management.

# Table of Contents
1. Introduction

2. <a href = "https://github.com/TIN0-m/Home-Lab/blob/main/Network_Architecture.md"> Architecture Overview <a/>

3. <a href ="https://github.com/TIN0-m/Home-Lab/blob/main/Logical-Flow.md"> Logical Flow <a/>

4. <a href ="https://github.com/TIN0-m/Home-Lab/blob/main/Deployment-Guide.md"> Deployment Guide <a/>
   
5. <a href ="https://github.com/TIN0-m/Home-Lab/blob/main/Security%20groups%20and%20AWS%20services.md"> Security groups and AWS services <a/>

6. <a href ="https://github.com/TIN0-m/Home-Lab/blob/main/Mitre_Attack_and_IS27001_principles.md"> Mitre Attack and ISO27001 principles <a/>  

7. <a href ="https://github.com/TIN0-m/Home-Lab/tree/main/Usage%20and%20Scenarios"> Usage & Scenarios <a/>

# 1. Introduction 

### Purpose:     

This repository details the architecture and deployment steps for an AWS-based Security Operations Center (SOC) lab. Its purpose is to provide a practical, hands-on environment to simulate cyberattacks, detect malicious activity, and practice incident response workflows using a suite of open-source and cloud-native security tools. It's my personal sandbox for mastering the entire incident response lifecycle, from the first byte of telemetry to automated containment and meticulous case management. 

# Objectives:  

- To bridge the gap between theoretical cybersecurity knowledge and tangible, hands-on experience in a real-world cloud environment.

- To demonstrate proficiency in designing, deploying, and securing complex cloud infrastructures (AWS VPC).

- To showcase practical skills in Endpoint Detection & Response (EDR), Security Information & Event Management (SIEM), Security Orchestration, Automation, & Response (SOAR), and Incident Response Platform (IRP) integration.

- To cultivate a proactive, blue-team mindset by simulating adversary tactics (Red Team) and building robust defenses (Blue Team).

- To translate theoretical understanding of frameworks like MITRE ATT&CK into actionable detections and automated responses.

# Skills : Detect. Automate. Respond. Analyze.
This lab provides opportunity to showcase the following:

- **Simulate Realistic Attacks:**       
 Execute Atomic Red Team playbooks against live Windows endpoints, mirroring sophisticated adversary techniques.

- **Achieve Unprecedented Visibility:**       
 Collect granular endpoint telemetry with Limacharlie, comprehensive system and cloud logs with Wazuh, and network flow data with AWS VPC Flow Logs.

- **Automate Tier 1/2 SOC Functions:**      
 Leverage Tines to orchestrate real-time alert triage, threat intelligence enrichment, and automated containment actions against Active Directory or endpoints.

- **Master Incident Response:**       
 Utilize TheHive for collaborative incident management, from initial alert ingestion to investigation, remediation, and post-mortem analysis.

- **Fortify Cloud Security Posture:**       
 Integrate AWS CloudTrail and GuardDuty findings into the SIEM, providing holistic visibility across the cloud control plane.

# Tools and Components used :
<div>
    <img src="https://img.shields.io/badge/-Wazuh-blue?&style=for-the-badge&logo=wazuh&logoColor=white" />
    <img src="https://img.shields.io/badge/-LimaCharlie-orange?&style=for-the-badge&logo=limacharlie&logoColor=white" />
    <img src="https://img.shields.io/badge/-TheHive-272E3F?&style=for-the-badge&logo=thehive&logoColor=white" />
    <img src="https://img.shields.io/badge/-AWS-FF9900?&style=for-the-badge&logo=amazonaws&logoColor=white" />
    <img src="https://img.shields.io/badge/-Windows%20Active%20Directory-0078D4?&style=for-the-badge&logo=microsoft&logoColor=white" />
    <img src="https://img.shields.io/badge/-Tines-1996D7?&style=for-the-badge&logo=tines&logoColor=white" />
    <img src="https://img.shields.io/badge/-Atomic%20Red%20Team-D81E05?&style=for-the-badge&logo=redhat&logoColor=white" />
</div>

## AWS VPC: 
<img src="https://img.shields.io/badge/-AWS-FF9900?&style=for-the-badge&logo=amazonaws&logoColor=white" />
This is where the entire simulated infrastructure will live. It will consist of subnets, security groups, routing, and access control lists to mimic a real corporate network, giving easy control and allow traffic observation.

## Active Directory (Identity and Access Management):
<img src="https://img.shields.io/badge/-Windows%20Active%20Directory-0078D4?&style=for-the-badge&logo=microsoft&logoColor=white" />
Critical for simulating a realistic enterprise environment. It will have a domain controller, user accounts, groups, and policies. This will allow me to:

- Simulate user activity.
- Generate authentication logs that your SIEM will ingest.
- Practice response actions.
- Provide authentication.

## Limacharlie (EDR and Foundational Telemetry): 
<img src="https://img.shields.io/badge/-LimaCharlie-orange?&style=for-the-badge&logo=limacharlie&logoColor=white" />
This tool is for endpoint visibility and initial detection.

Limacharlie's lightweight, robust agent provides extremely rich endpoint telemetry (process execution, network connections, file system changes, registry, etc.) far beyond what basic Windows Event Logs offer. It's purpose-built for threat detection and response at the endpoint level. It essentially replaces the need for a separate Sysmon agent as its data is similar but often more extensive and easier to consume.

When the Atomic Red Team attacks are ran, Limacharlie will be the primary sensor detecting these activities. Its rule engine allows me to write custom detections for ATT&CK techniques.

## Wazuh (SIEM and HIDS):
<img src="https://img.shields.io/badge/-Wazuh-blue?&style=for-the-badge&logo=wazuh&logoColor=white" />
This tool will work as a SIEM and Host Intrusion Detection System (HIDS).

It's excellent for collecting, normalizing, and analyzing logs from diverse sources beyond just endpoints (e.g., Active Directory logs, AWS CloudTrail, VPC Flow Logs, firewall logs). It has a strong rule set for common attack patterns and compliance.

Limacharlie alerts can be forwarded to Wazuh for broader correlation, and Wazuh will also collect AD logs directly from the domain controller.

## TheHive (Case Management & Incident Response):
<img src="https://img.shields.io/badge/-TheHive-272E3F?&style=for-the-badge&logo=thehive&logoColor=white" />
This tool will be my central hub for the incident response process.

It provides structure for investigations, allows analysts to collaborate, manage observables, and track the status of incidents from creation to closure. 

## Tines (Dedicated SOAR - Automation & Orchestration):
<img src="https://img.shields.io/badge/-Tines-1996D7?&style=for-the-badge&logo=tines&logoColor=white" />
This tool will be used for automating complex workflows. It allows you to create intricate playbooks that connect all your tools. Limacharlie and Wazuh will be configured to send alerts to Tines. Tines will then orchestrates actions like:

- Automatically creating cases in TheHive.
- Enriching alerts with external threat intelligence.
- Performing automated response actions on Active Directory (e.g., disabling users).
- Notifying team members.

## Atomic Red Team (Attack Simulation):
<img src="https://img.shields.io/badge/-Atomic%20Red%20Team-D81E05?&style=for-the-badge&logo=redhat&logoColor=white" />
The primary tool for generating realistic attack data.

It provides a structured way to simulate specific MITRE ATT&CK techniques, allowing me to test the detection capabilities in Limacharlie and Wazuh, and the automated responses in Tines.

<a href ="https://github.com/TIN0-m/Home-Lab/blob/main/Network_Architecture.md"><img src="https://img.shields.io/badge/-Next%20Section-FF0000?&style=for-the-badge&logoColor=white" /><a/>

