# Logical Flow 
This section will provide a clear overview of the event and incident lifecycle as it unfolds in the simulated Security Operations Center (SOC) environment.

## Attack Simulation (Atomic Red Team):

**Action:**      
You execute an Atomic Red Team test (e.g. command and control simulation) on a Windows endpoint within your VPC.
**Output:**       
Generates malicious activity and associated telemetry.

## Endpoint Telemetry & Initial Detection (Limacharlie):

**Action:**       
The Limacharlie agent on the endpoint captures rich telemetry (process creation, network connections, file changes, etc.) related to the Atomic Red Team activity. Limacharlie's built-in rules perform initial detection.
**Output:**           
High-fidelity endpoint events, logs, and potential alerts.

## Log Ingestion & SIEM (Wazuh):

**Action:**      
- Wazuh agents on endpoints and servers (including Active Directory) collect system logs, application logs, and any additional logs not covered by Limacharlie (or could be configured to send Limacharlie alerts).
- Wazuh also ingests AWS CloudTrail, VPC Flow Logs, and Active Directory security logs.
- Wazuh's rules engine processes these logs for broader correlation and threat detection.

**Output:**    
Correlated events, normalized logs, and SIEM alerts.

## Alert Aggregation & Orchestration (Tines):

**Action:**      
- Both Limacharlie and Wazuh are configured to send their critical alerts to Tines.
- Tines receives these alerts and triggers pre-defined automation playbooks.

**Output:**         
Automated workflows initiated.

## Automated Response & Enrichment (Tines orchestrates):

**Action:**         
- Automatically creating a new alert/case in TheHive, populating it with details from the initial alert.
- Threat Intelligence Enrichment: Querying external threat intelligence sources for indicators (IPs, hashes) from the alert.
- Interacting with Active Directory to disable a compromised user account or reset a password if the alert warrants it.
- Sending a notification to a security team channel.

**Output:**       
Enriched incident data, automated containment/response actions.

## Incident Case Management (TheHive):

**Action:**     
Analysts use TheHive to review the automatically created cases, investigate further, add observables, collaborate with team members, and document the investigation and resolution steps.

**Output:**      
Managed incident lifecycle, documented investigation.
Analyst Feedback/Refinement: Based on investigations, analysts might refine detection rules in Limacharlie or Wazuh, or improve automation playbooks in Tines.
