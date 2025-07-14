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

<img width="801" height="541" alt="Net Overview" src="https://github.com/user-attachments/assets/f25da4f9-1a90-409a-a3c1-9484179cb15e" />

