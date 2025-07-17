# MITRE ATT&CK AND ISO27001 
This Lab touches on several MITRE ATT&CK techniques and aligns with multiple ISO 27001 principles, directly addressing how to detect, respond, and manage security incidents.

## MITRE ATT&CK Coverage
The focuses heavily on Defense Evasion, Collection, and aspects of Discovery and Exfiltration, primarily from the defender's perspective. By simulating attacks and collecting logs, we're building capabilities to identify these techniques.

Here are some specific **ATT&CK** techniques that are addressed:

### Discovery (TA0007)
**T1087.001 Account Discovery: Local Account and T1087.002 Account Discovery: Domain Account:**      
When an attacker is on an endpoint, they often try to find user and administrator accounts. Your Wazuh agents collecting Windows security event logs and Limacharlie detecting unusual enumeration attempts directly target these.

**T1046 Network Service Discovery:**     
Attackers scan for open ports. Wazuh can be configured to alert on common port scanning activities from agents.

**T1018 Remote System Discovery:**     
Looking for other machines on the network. Wazuh can log and alert on network connection attempts between endpoints.

### Collection (TA0009)
**T1005 Data from Local System:**    
Attackers collecting sensitive files from the compromised host. Wazuh can detect unauthorized modifications or access to critical files.

**T1020 Automated Collection:**         
Tools that automatically gather data. Both Wazuh and Limacharlie are designed to automatically collect system logs, process information, and network activity, which are the sources an attacker would target.

### Defense Evasion (TA0005)
**T1070.001 Indicator Removal: Clear Windows Event Logs:**      
A common attacker tactic. The Wazuh agents are configured to forward logs in real-time. If an attacker attempts to clear local logs, this event would already be sent to the Wazuh Manager, providing an alert even if the local log is wiped.

**T1562 Impair Defenses:**     
This is a broad category, and the lab aims to detect attempts at disabling security tools or firewalls. Wazuh and Limacharlie can monitor service states and system configurations for changes.

### Exfiltration (TA0010)
**T1041 Exfiltration Over C2 Channel:**     
Data being sent out over the same channel used for command and control. While our lab doesn't build a C2, by monitoring network traffic via VPC Flow Logs ingested into Wazuh, you could detect anomalous outbound data flows.

**T1048 Exfiltration Over Alternative Protocol:**   
VPC Flow Logs combined with Wazuh's analysis capabilities are key to spotting unusual patterns.

### Command and Control (TA0011)
**T1071 Application Layer Protocol:**    
Using common protocols like HTTP, DNS for C2. VPC Flow Logs and Limacharlie's network visibility can help identify unusual connections to known malicious domains or IPs.

## ISO 27001 Coverage 
ISO 27001 is a standard for Information Security Management Systems (ISMS). Our lab, while a technical implementation, directly supports several key principles within an ISMS.

### A.5 Information Security Policies
**A.5.1.1 Policies for Information Security:**    
The very act of designing this lab (defining networks, access controls, logging) is an implementation of implied security policies. The output of the lab (monitoring, alerting) provides data to enforce these policies.

### A.6 Organization of Information Security
**A.6.1.2 Segregation of Duties:**     
Our network design with separate Management, SOC Tooling, and Enterprise subnets, along with a dedicated Jump Box, supports the principle of segregating administrative access to critical components. Not all users have access to all network segments.

### A.9 Access Control
**A.9.1.2 Access Control Policy:**    
Explicitly defined in our Security Groups (SGs). Each SG strictly controls who/what can communicate with the instances.

**A.9.2.1 User Registration:**     
Covered by your Active Directory (AD) setup. You'll create and manage user accounts centrally.

**A.9.4.1 Secure Log-on Procedures:**      
The use of the Jump Box and SSH/RDP with key pairs/passwords provides a more secure way to log onto internal instances than direct public exposure.

**A.9.4.3 Password Management System:**    
AD handles password policies for domain-joined machines.

### A.12 Operations Security
**A.12.1.1 Documented Operating Procedures:**    
While we don't document them in the lab, the entire setup implicitly establishes procedures for deploying, managing, and securing systems.

**A.12.4.1 Event Logging:**     
This is a core pillar of our lab. Wazuh and Limacharlie are dedicated to collecting and storing security-related event logs from across the environment (Windows events, Sysmon, process activity, network flows, AWS service logs).

**A.12.4.2 Protection of Log Information:**    
Logs are centralized on the Wazuh Manager, which should be secured, preventing tampering. Using S3 for AWS logs inherently provides good protection.

**A.12.4.3 Administrator and Operator Logs:**    
All administrative actions performed via the Jump Box and within the instances are logged.

**A.12.4.4 Clock Synchronization:**   
Crucial for accurate log correlation, a fundamental requirement for a functional SOC. Implicitly supported by AWS instances having synchronized clocks.

### A.13 Communications Security
**A.13.1.1 Network Security Management:**    
The entire VPC, Subnets, Route Tables, Internet Gateway, NAT Gateway, and Security Groups represent a robust implementation of network security management, controlling traffic flow and isolation.

**A.13.1.2 Security of Network Services:**    
Defining specific ports and protocols in Security Groups ensures only authorized services are accessible.

**A.13.2.1 Information Transfer Policies:**    
While not explicitly defined, the secure transfer of logs (Wazuh agent to manager) and alerts (Wazuh/Limacharlie to Tines) demonstrates secure information transfer.

### A.16 Information Security Incident Management
**A.16.1.1 Responsibilities and Procedures:**    
This is the ultimate goal of the lab. By simulating attacks and routing alerts to TheHive (for case management) and Tines (for orchestration/response), you're building a system to support incident management procedures.

**A.16.1.2 Reporting Information Security Events:**    
Wazuh and Limacharlie generate the events that are then reported to Tines and TheHive.

**A.16.1.5 Response to Information Security Incidents:**    
Tines playbooks, when configured for automated actions (e.g., isolating a host via Limacharlie, disabling an AD user), directly address this principle.

### A.17 Information Security Aspects of Business Continuity Management
**A.17.2.1 Availability of Information Processing Facilities:**    
While a single-region lab, the use of AWS services provides inherent underlying high availability compared to on-premises solutions, supporting the continuity principle.
