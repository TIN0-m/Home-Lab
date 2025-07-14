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

## AWS VPC: 
This is where the entire simulated infrastructure will live. It will consist of subnets, security groups, routing, and access control lists to mimic a real corporate network, giving easy control and allow traffic observation.

## Active Directory (Identity and Access Management):     
Critical for simulating a realistic enterprise environment. It will have a domain controller, user accounts, groups, and policies. This will allow me to:

- Simulate user activity.
- Generate authentication logs that your SIEM will ingest.
- Practice response actions.
- Provide authentication.

## Limacharlie (EDR and Foundational Telemetry):    
This is your most powerful tool for endpoint visibility and initial detection.

Why it's best here: Limacharlie's lightweight, robust agent provides extremely rich endpoint telemetry (process execution, network connections, file system changes, registry, etc.) far beyond what basic Windows Event Logs offer. It's purpose-built for threat detection and response at the endpoint level. It essentially replaces the need for a separate Sysmon agent as its data is similar but often more extensive and easier to consume within Limacharlie's platform.

Key benefit: When you run Atomic Red Team attacks, Limacharlie will be your primary sensor detecting these activities. Its rule engine allows you to write custom detections for ATT&CK techniques.

Wazuh (SIEM & HIDS): While Limacharlie offers some SIEM-like capabilities, Wazuh excels as an open-source SIEM and Host Intrusion Detection System (HIDS).

Why it's best here: It's excellent for collecting, normalizing, and analyzing logs from diverse sources beyond just endpoints (e.g., Active Directory logs, AWS CloudTrail, VPC Flow Logs, firewall logs). It has a strong rule set for common attack patterns and compliance. It also provides file integrity monitoring (FIM) and vulnerability detection on the host.

Integration: Limacharlie alerts can be forwarded to Wazuh for broader correlation, and Wazuh can collect AD logs directly from your domain controller.

TheHive (Case Management & Incident Response): The central hub for your incident response process.

Why it's best here: It provides structure for investigations, allows analysts to collaborate, manage observables, and track the status of incidents from creation to closure. It's an indispensable tool for a functional SOC.

Tines (Dedicated SOAR - Automation & Orchestration): This will be your powerhouse for automating complex workflows.

Why it's best here: While Limacharlie has automation, Tines is a dedicated SOAR platform with a highly visual, no-code/low-code builder. It allows you to create intricate playbooks that connect all your tools.

Key benefit: You can configure Limacharlie and Wazuh to send alerts to Tines. Tines then orchestrates actions like:

Automatically creating cases in TheHive.

Enriching alerts with external threat intelligence.

Performing automated response actions on Active Directory (e.g., disabling users).

Notifying team members via chat (e.g., Slack if you add it).

Simplicity vs. Power: Limacharlie's automation is powerful for its own data. Tines is powerful for orchestrating across all your different tools. For a realistic simulation, having a dedicated SOAR tool like Tines is invaluable to practice automating response actions.

Atomic Red Team (Attack Simulation): Your primary tool for generating realistic attack data.

Why it's best here: It provides a structured way to simulate specific MITRE ATT&CK techniques, allowing you to test your detection capabilities in Limacharlie and Wazuh, and then your automated responses in Tines.

