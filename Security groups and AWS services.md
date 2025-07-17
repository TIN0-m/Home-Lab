# Security Groups and AWS services

The Security Groups act as a gatekeeper right in front of the EC2 instances. It inspects every single piece of network traffic trying to reach or leave that instance and decides whether to allow or deny it based on a set of rules you define.
**Here's a breakdown of their characteristics:**

## Instance-Level Control
The Security Groups are associated directly with each of the subnets that are in the VPC. This gives you granular control over each specific instance's traffic as the Subnets have the overal instances that are being used.

## Inbound Traffic and Outbound Traffic
By adding rules to allow and deny any connections and specifying which ports can do this adheres to the **principle of least privilege**, which is a core security best practice.

## Stateful
This is a very important characteristic. Security Groups are stateful. This means if you allow an inbound connection (e.g., SSH on port 22), the return traffic for that connection is automatically allowed back out, even if you don't have an explicit outbound rule for it. Similarly, if you allow an outbound connection (e.g., an instance initiating a web request on port 443), the return traffic for that request is automatically allowed back in. This simplifies rule management significantly.

## Rule Evaluation
When an instance has multiple Security Groups associated with it, all the rules from all associated Security Groups are aggregated to form one logical set of rules. AWS evaluates all these rules before deciding whether to allow the traffic. There's no specific order of precedence for rules within a Security Group, as all allow rules are considered.

## Security Groups in the Lab
These are the Security groups that are being implemented into the lab

**JumpBox Security Group:**        
Allows inbound SSH or RDP allows outbound SSH/RDP to your private instances.

**Active Directory Security Group:**       
Allows inbound LDAP, Kerberos, DNS from your Windows/Linux endpoints, and potentially outbound access for updates via the NAT Gateway.

**Coperate Network Security Group:**     
Allows inbound domain traffic from Active Directory and outbound traffic for agent communication (Wazuh, Limacharlie) and updates.

**Wazuh Security Group:**      
Allows inbound agent communication from your endpoints and outbound communication for alerts to Tines or external services.

By meticulously configuring these Security Groups, a robust instance-level firewalling system that strictly controls the communication pathways within and out of the VPC is created.

## AWS CloudTrail:

Purpose: The definitive record of all API activity within your AWS account. Who did what, when, and from where. Crucial for detecting unauthorized configuration changes, privilege escalation, or resource manipulation.

Configuration: Enabled account-wide, logs sent to a dedicated S3 bucket (soc-lab-cloudtrail-logs-yourid).

Integration: Wazuh is configured to poll this S3 bucket for real-time ingestion and analysis of cloud control plane events.

## VPC Flow Logs:

Purpose: Captures detailed IP traffic information for network interfaces within your VPC. Think of it as NetFlow for your AWS network. Essential for network anomaly detection, traffic analysis, and identifying suspicious connections.

Configuration: Enabled on the soc-lab-vpc, logs sent to a dedicated S3 bucket (soc-lab-vpc-flow-logs-yourid) or CloudWatch Log Group.

Integration: Wazuh is configured to ingest these logs, providing crucial network-level context for investigations.

## AWS GuardDuty (Highly Recommended for Advanced Detections):

Purpose: An intelligent, managed threat detection service that continuously monitors your AWS accounts for malicious activity and unauthorized behavior. It uses machine learning, anomaly detection, and integrated threat intelligence.

Configuration: Simply enable it in your AWS account.

Integration: GuardDuty findings can be automatically sent to CloudWatch Events, which can then trigger a Lambda function to push to Wazuh or directly into Tines, enriching your SIEM with high-fidelity, AWS-specific threat intelligence.

## IAM Roles & Policies (The Gatekeepers of Access)
Purpose: Defines granular permissions for EC2 instances to interact with other AWS services. This prevents instances from having overly permissive credentials.

Key Roles:

Wazuh-S3-Reader-Role: Attached to the Wazuh Manager EC2 instance, granting it s3:GetObject and s3:ListBucket permissions only on the specific S3 buckets where CloudTrail and VPC Flow Logs are stored.

Tines-AWS-Responder-Role (Optional): If Tines is configured to perform automated response actions (e.g., isolating an EC2 instance, revoking temporary credentials), it would need a role with the absolute minimum necessary permissions for those specific actions.

Principle of Least Privilege (PoLP): Every IAM role and associated policy adheres strictly to PoLP, ensuring no component has more permissions than it absolutely needs to function.
