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
