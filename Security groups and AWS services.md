# Security Groups and AWS services

## Security Groups

The Security Groups act as a gatekeeper right in front of the EC2 instances. It inspects every single piece of network traffic trying to reach or leave that instance and decides whether to allow or deny it based on a set of rules you define.
**Here's a breakdown of their characteristics:**

### Instance-Level Control
The Security Groups are associated directly with each of the subnets that are in the VPC. This gives you granular control over each specific instance's traffic as the Subnets have the overal instances that are being used.

### Inbound Traffic and Outbound Traffic
By adding rules to allow and deny any connections and specifying which ports can do this adheres to the **principle of least privilege**, which is a core security best practice.

### Stateful
This is a very important characteristic. Security Groups are stateful. This means if you allow an inbound connection (e.g., SSH on port 22), the return traffic for that connection is automatically allowed back out, even if you don't have an explicit outbound rule for it. Similarly, if you allow an outbound connection (e.g., an instance initiating a web request on port 443), the return traffic for that request is automatically allowed back in. This simplifies rule management significantly.

### Rule Evaluation
When an instance has multiple Security Groups associated with it, all the rules from all associated Security Groups are aggregated to form one logical set of rules. AWS evaluates all these rules before deciding whether to allow the traffic. There's no specific order of precedence for rules within a Security Group, as all allow rules are considered.

### Security Groups in the Lab
These are the Security groups that are being implemented into the lab

**JumpBox Security Group:**        
Allows inbound SSH or RDP allows outbound SSH/RDP to your private instances.

**Active Directory Security Group:**       
Allows inbound LDAP, Kerberos, DNS from your Windows/Linux endpoints, and potentially outbound access for updates via the NAT Gateway.

**Coperate Network Security Group:**     
Allows inbound domain traffic from Active Directory and outbound traffic for agent communication (Wazuh, Limacharlie) and updates.

**Wazuh Security Group:**      
Allows inbound agent communication from your endpoints and outbound communication for alerts to Tines or external services.

**By meticulously configuring these Security Groups, a robust instance-level firewalling system that strictly controls the communication pathways within and out of the VPC is created.**

## AWS Services
The AWS services are there to provide with extra security measures. By combining these native services with the tools i have chosen (Wazuh, TheHive, Tines), it creates a comprehensive environment capable of robust detection, analysis, and response.

### AWS CloudTrail

AWS CloudTrail is an audit log of all actions that happen in the AWS environment. Every time someone logs into the AWS console, runs a command via the AWS CLI, or an AWS service makes an API call, CloudTrail logs it automatically and captures management events. It provides a 90-day event history in the console, allowing you to quickly look up recent activities. It can also enable log file integrity validation, using hashing and digital signatures to ensure that your log files haven't been tampered with.

#### Why it's crucial: 

- **Forensics and Incident Response:**    
If a breach occurs, CloudTrail logs are the go-to source to answer critical questions: "Who launched that suspicious instance?", "When was that security group modified?".

- **Compliance:**     
Many regulatory frameworks require detailed audit trails of activity. CloudTrail helps meet these requirements.

- **Operational Troubleshooting:**     
 Debugging why a service isn't working or why a user can't access a resource.

- **Threat Detection:**      
 Unusual API calls can be indicators of compromise. These logs are fed into the Wazuh Manager for analysis and correlation with other log sources.


### Amazon GuardDuty: Intelligent Threat Detection 

Amazon GuardDuty is a managed threat detection service that continuously monitors your AWS accounts and workloads for malicious activity and unauthorized behavior. It uses machine learning, anomaly detection, and integrated threat intelligence to identify potential threats. Once enabled, GuardDuty continuously analyzes data from several AWS log sources. GuardDuty applies a combination of pre-built threat intelligence feeds (known malicious IPs, domains) and machine learning models. It builds a baseline of normal behavior.


#### Why it's crucial:

- **Automated Threat Detection:**      
GuardDuty is always on and provides immediate alerts without need to deploy or manage any agents or infrastructure.

- **Broad Coverage:**      
It covers a wide range of potential threats.

- **Reduced Noise:**
Its ML models help reduce false positives by learning the environment's typical behavior.

- **Quick Identification of IOCs:**
It can quickly identify Indicators of Compromise (IOCs) based on its threat intelligence.

### AWS VPC Flow Logs: The "Traffic Cop" of Your Network 

VPC Flow Logs is a feature that enables the capture of information about the IP traffic going to and from network interfaces in your Virtual Private Cloud (VPC). It's like a traffic recorder for your network, logging metadata about every connection. It enables flow logs at the VPC or subnet level.It also gives the choice to accept or reject traffic.

#### Why it's crucial:

- **Network Visibility:**   
 Provides deep insights into who is talking to whom, on what ports, and how much data is being transferred within your VPC and with external networks.

- **Security Monitoring:**    
Crucial for detecting:

- **Port scanning:**    
 Many rejected connections from a single source to multiple ports.

- **Policy violations:**    
 Instances communicating with unauthorized internal or external systems can be stopped.

- **Troubleshooting:**     
Essential for diagnosing connectivity issues, misconfigured security groups, or route table problems.

<a href ="https://github.com/TIN0-m/Home-Lab/blob/main/Mitre_Attack_and_IS27001_principles.md"><img src="https://img.shields.io/badge/-Next%20Section-FF0000?&style=for-the-badge&logoColor=white" /><a/>
