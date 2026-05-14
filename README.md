# AWS Secure & Monitored Cloud Architecture

Secure AWS infrastructure with EC2, Public &amp; Private Subnets, CloudWatch Monitoring &amp; Alarms, SNS, and S3 Logging.

---

## 1. Project Overview & Objectives

### The Challenge
In a standard cloud setup, servers are often exposed to the internet via public IPs and SSH ports (22), making them vulnerable to brute-force attacks. This project aims to deploy a web infrastructure that is **invisible** to the public internet while remaining **fully manageable** and **monitored**.

### Key Objectives:
* **Infrastructure Hardening**: Isolate the EC2 instance in a private subnet.
* **Zero-Attack Surface**: Eliminate SSH access in favor of AWS Systems Manager (SSM).
* **Advanced Monitoring**: Track internal metrics (RAM/Disk) that AWS doesn't monitor by default.
* **Automated Response**: Setup a notification system for real-time incident response with SNS.
* **Secure Data**: Implement secure log archiving to Amazon S3.

### Expected Results:
1.  A server with **no public IP** and **no open ports**.
2.  A functional **NAT Gateway** for secure outbound updates.
3.  A **CloudWatch Dashboard** showing live RAM usage.
4.  An **automated email alert** triggered during high-load scenarios with SNS.
5.  Verification of **log persistence** in an encrypted S3 bucket.

---

## 2. High-Level Architecture Diagram
The diagram below illustrates the secure flow: traffic stays within the AWS backbone, and the instance remains isolated.

<img width="1176" height="721" alt="AWS VPC Architecture" src="https://github.com/user-attachments/assets/9a04e05a-fc8e-4e47-b067-d0952e3ec59f" />

---

### Step 1 : Secure Network Foundation (VPC, Subnets, Routing, NAT & IGW)

The infrastructure is built on a **segmented VPC architecture** to isolate resources and strictly control traffic flow.

#### Network Configuration

- **VPC**: `10.0.0.0/16`
- **Public Subnet**: `10.0.1.0/24`  
- **Private Subnet**: `10.0.2.0/24`  

#### Internet Access Design

- An **Internet Gateway (IGW)** is attached to the VPC  
- A **NAT Gateway** (with Elastic IP) is deployed in the public subnet  

➡️ This allows outbound internet access for private resources without exposing them.

#### Routing Configuration

Two route tables were configured:

- **Public Route Table**  
  - `0.0.0.0/0 → Internet Gateway`  

- **Private Route Table**  
  - `0.0.0.0/0 → NAT Gateway`  

➡️ This ensures:
- Public subnet has direct internet access  
- Private subnet can only initiate outbound connections  

💡 This routing strategy is key to maintaining a **private-by-default architecture**.

<img width="1809" height="327" alt="Capture d&#39;écran 2026-04-23 070017" src="https://github.com/user-attachments/assets/cb793559-1351-4e77-abde-e459467ff6c0" />

---

### Step 2 : Identity & Access Management (IAM Role)

To securely manage permissions without using static credentials, an **IAM Role** was created and attached to the EC2 instance.

This approach follows AWS best practices by leveraging **temporary credentials automatically provided by AWS**, eliminating the need for hardcoded access keys.

#### Attached Policies

The IAM role includes the following policies:

- **AmazonSSMManagedInstanceCore**  
  → Enables secure remote access via AWS Systems Manager (SSM), removing the need for SSH access  

- **CloudWatchAgentServerPolicy**  
  → Allows the instance to publish system-level metrics (RAM, disk usage) to CloudWatch  

- **Custom S3 Write Policy (Least Privilege)**  
  → Grants write-only access to a dedicated S3 bucket for log storage   

➡️ Permissions were intentionally restricted to the minimum required, following the **Principle of Least Privilege**.

---

#### Security Principles Applied

- **Least Privilege**: Only strictly necessary permissions were granted  
- **No Static Credentials**: No access keys stored on the instance  
- **Secure Administration**: All access is performed via AWS Systems Manager (SSM)  
- **Audit-Friendly Design**: Actions are traceable through AWS logging services 

💡 This design significantly reduces the risk of credential exposure and unauthorized access.

<img width="1221" height="493" alt="image" src="https://github.com/user-attachments/assets/751f0461-4082-4197-8940-5f6111beb265" />

---

### Step 3 : Hardened Compute Deployment (Private EC2)

The EC2 instance was deployed with a **security-first approach**, ensuring it remains completely isolated from the public internet while still fully manageable.

---

### Instance Configuration

- Deployed inside the **Private Subnet only**
- **No Public IP assigned**
- Operating system: Linux (Amazon Linux / Ubuntu)
- Managed exclusively via **AWS Systems Manager (SSM)**

➡️ This design completely removes the need for SSH access.

<img width="586" height="562" alt="Capture d&#39;écran 2026-04-22 104110" src="https://github.com/user-attachments/assets/64f6d64d-b213-47fe-b3d7-290b79936063" />

---

### Security Group Configuration

A dedicated Security Group was created with strict access control:

- **Inbound Rules**:
  - HTTP (port 80) allowed only from the public subnet (`10.0.1.0/24`)
  - No SSH (port 22) allowed

- **Outbound Rules**:
  - All traffic allowed (`0.0.0.0/0`) to enable system updates and AWS service communication

➡️ This ensures controlled application access while maintaining necessary outbound connectivity.

<img width="1645" height="632" alt="Capture d&#39;écran 2026-04-22 100133" src="https://github.com/user-attachments/assets/40aa0664-1a70-4223-adcf-0bf3ddb3913b" />

<img width="1648" height="632" alt="Capture d&#39;écran 2026-04-22 100143" src="https://github.com/user-attachments/assets/daccdf42-5086-472d-a56c-1a3f6d7d5575" />

---

### Instance Access & Management

- No SSH access is used
- All administration is performed via **AWS Systems Manager Session Manager**
- Instance is managed through IAM role permissions defined in Step 2

---

💡 This architecture significantly reduces the attack surface by:
- Removing all inbound internet exposure
- Eliminating SSH as an access method
- Centralizing access through AWS-native secure tooling

---

### Step 4 : Monitoring & Proactive Alerting (CloudWatch & SNS)

To ensure real-time visibility of system performance and enable proactive incident detection, a monitoring and alerting pipeline was implemented using **Amazon CloudWatch and SNS**.

---

### CloudWatch Monitoring

The EC2 instance was integrated with **Amazon CloudWatch** to collect both default and custom metrics.

#### Metrics Collected:

- CPU Utilization (default metric)
- Network activity (default metric)
- **Memory usage (custom metric via CloudWatch Agent)**
- **Disk usage (custom metric via CloudWatch Agent)**

➡️ Since memory and disk are not available by default in EC2 metrics, the **CloudWatch Agent** was installed and configured on the instance.

---

### CloudWatch Agent Configuration

The agent was deployed on the EC2 instance to push system-level metrics to CloudWatch in near real-time.

This enables deeper observability beyond default AWS monitoring capabilities.

---

### CloudWatch Dashboard (Centralized Observability)

A CloudWatch Dashboard was implemented to provide a centralized real-time visualization of EC2 performance metrics.

This dashboard aggregates key system indicators in a single view:

- Memory usage (via CloudWatch Agent)
- Disk usage

➡️ This provides a “single pane of glass” view of the instance health, improving operational visibility and simplifying monitoring.

<img width="1362" height="624" alt="Capture d&#39;écran 2026-04-22 121654" src="https://github.com/user-attachments/assets/6c63be54-934a-4c08-8811-20ace5c1e496" />

---

### Alerting System (Amazon SNS)

To enable proactive incident response, a **CloudWatch Alarm** was configured:

- Trigger condition: High memory usage threshold (> 80%)
- Evaluation: Continuous monitoring over defined period
- Action: Trigger Amazon SNS notification

---

### Notification System (SNS Alerting)

When memory usage exceeds the defined threshold (> 80%), the monitoring pipeline triggers a full alerting workflow.

---

### 1. CloudWatch Alarm Triggered

When the EC2 instance reaches high memory usage, the CloudWatch alarm transitions into the **ALARM state**.

This indicates that the system is under stress and requires immediate attention.

<img width="486" height="287" alt="Capture d&#39;écran 2026-04-22 134211" src="https://github.com/user-attachments/assets/c17c3257-e641-42bf-a871-db42edbb4b32" />

<img width="1021" height="343" alt="Capture d&#39;écran 2026-04-22 134255" src="https://github.com/user-attachments/assets/02ba77ff-d3ca-4137-9f22-9eb6cd463a12" />

---

### 2. SNS Email Notification Sent

Once the alarm is triggered, an **Amazon SNS topic** automatically sends a notification email to the administrator.

The email includes:
- Alarm status (ALARM state)
- Triggered metric (memory usage)
- Threshold breach information

This ensures immediate awareness of system performance issues without manual monitoring.

<img width="833" height="538" alt="Capture d&#39;écran 2026-04-22 131259" src="https://github.com/user-attachments/assets/94a027e4-76b0-44fa-b77e-656a62d872bf" />

---

💡 This monitoring architecture provides:
- Real-time system visibility
- Proactive incident detection
- Automated alerting 
- Centralized observability through dashboards

---

### Step 5 : Secure Data Archiving (Amazon S3 Logging)

To ensure secure and durable log storage, the EC2 instance was configured to archive logs into a dedicated **Amazon S3 bucket**.

This enables long-term data retention, auditability, and secure storage of system-generated logs.

---

### S3 Bucket Configuration

A dedicated S3 bucket was created with a security-first configuration:

- **Block Public Access**: Fully enabled
- **Versioning**: Enabled to prevent accidental deletion or overwrites
- **Server-Side Encryption (SSE-S3)**: Enabled to secure data at rest
- **Access Control**: Restricted via IAM role only

➡️ This ensures that all stored data remains private, secure, and protected against unauthorized access.

---

### Secure Access via IAM Role

The EC2 instance uploads logs to S3 using the IAM role defined in Step 2, following a **least privilege approach**:

- No hardcoded credentials
- Access restricted to a single S3 bucket
- Write-only permissions for log upload operations

➡️ This guarantees secure and controlled interaction between EC2 and S3.

---

### Secure Administration via AWS Systems Manager (SSM)

The EC2 instance is fully managed through **AWS Systems Manager (SSM)**, eliminating the need for SSH or direct network exposure.

All administrative actions and validations (including S3 log uploads) are performed through SSM sessions.

➡️ This ensures that the instance remains completely private and inaccessible from the public internet.

---

### Log Upload Verification

The log transfer process from the EC2 instance to Amazon S3 was validated using AWS CLI through **AWS Systems Manager (SSM) Session Manager**.

This ensures that all interactions remain fully private and are executed without SSH or public internet exposure.

---

### 1. Log Upload Execution via SSM

A log file was successfully uploaded to the S3 bucket directly from the EC2 instance using AWS CLI inside an SSM session.

This confirms:
- Secure execution environment via SSM
- Proper IAM role-based permissions
- Successful S3 write operation from the private instance

<img width="860" height="80" alt="Capture d&#39;écran 2026-04-22 202731" src="https://github.com/user-attachments/assets/289828f7-033b-44cc-ab2b-655192db7643" />

---

### 2. S3 Bucket Verification

The uploaded log file was then verified directly in the **Amazon S3 bucket**, confirming successful persistence and storage.

This validates:
- Data was correctly written to S3
- IAM permissions are properly enforced
- Log persistence pipeline is fully operational

<img width="1913" height="437" alt="Capture d&#39;écran 2026-04-22 202913" src="https://github.com/user-attachments/assets/14ad65a1-8627-4a82-b4ba-8712f1f13e65" />

---

💡 This setup ensures:
- Secure log persistence
- Long-term data retention
- Fully isolated storage architecture
- No public exposure of sensitive data

---

## 3. Architecture Summary

The final architecture is composed of:

- A **VPC with public and private subnets**
- A **NAT Gateway for controlled outbound traffic**
- A **private EC2 instance managed via AWS SSM**
- A **CloudWatch monitoring and alerting system with SNS**
- A **secure S3 bucket for log storage**

➡️ All components are integrated following AWS security best practices.

---

## 4. Project Outcome

This project successfully demonstrates the deployment of a **secure, fully isolated AWS infrastructure** with real-time monitoring and automated alerting.

It replicates a production-like environment where:
- The infrastructure is protected by design (no public exposure)
- Monitoring and alerting are fully automated
- Data is securely stored and controlled

💡 The result is a cloud architecture aligned with modern security and operational best practices.

---

## 5. Key Learnings

Through this project, I gained hands-on experience in designing and implementing a secure AWS cloud architecture.

Key takeaways include:

- Designing a **private-by-default VPC architecture**
- Implementing **IAM roles with least privilege principles**
- Replacing SSH access with **AWS Systems Manager (SSM)**
- Building a full **observability stack (CloudWatch, custom metrics, alarms, dashboards)**
- Setting up **automated alerting with Amazon SNS**
- Implementing secure **S3 log archiving and data retention**

💡 This project strengthened both my cloud architecture understanding and my security-first mindset.

---

## Final Note

This project demonstrates a complete end-to-end AWS cloud architecture combining infrastructure design, security best practices, observability, and automated alerting.

It reflects a production-like mindset focused on reliability, security, and operational efficiency.

---

## Author

Gregory JACQUET

Aspiring Cloud DevOps / Cloud Engineer Security

LinkedIn: https://www.linkedin.com/in/gregoryjacquet/

This project was designed and implemented as part of a personal initiative to develop hands-on skills in AWS Cloud, Security, and Infrastructure Architecture.

---

## Career Focus

This project reflects my growing interest in Cloud Architecture and the design of secure and scalable cloud systems.

My goal is to continue developing expertise in cloud architecture, automation, and security best practices in real-world environments.
