# SecureCloudNet — Automated AWS Network Security & Monitoring Lab

![AWS](https://img.shields.io/badge/AWS-Cloud-orange)
![Terraform](https://img.shields.io/badge/Terraform-Infrastructure%20as%20Code-purple)
![Ubuntu](https://img.shields.io/badge/OS-Ubuntu-E95420)
![CloudWatch](https://img.shields.io/badge/Amazon-CloudWatch-blue)
![CloudTrail](https://img.shields.io/badge/AWS-CloudTrail-red)
![Security](https://img.shields.io/badge/Focus-Network%20Security-green)

## Overview

**SecureCloudNet** is an automated AWS network security and monitoring laboratory designed to demonstrate practical cloud networking, infrastructure security, security monitoring, and Infrastructure as Code.

The project builds a secure AWS network using **Terraform** and implements multiple layers of security and monitoring, including:

* AWS VPC networking
* Public and private subnets
* Internet Gateway and route tables
* Security Groups
* Network ACLs
* EC2 security-test server
* IAM and AWS Systems Manager
* VPC Flow Logs
* CloudWatch monitoring and alarms
* CloudTrail activity logging
* Rejected network traffic detection
* Automated infrastructure deployment with Terraform

---

## Project Objectives

The main objectives of SecureCloudNet are to:

1. Build a structured AWS network using Terraform.
2. Apply multiple layers of network security.
3. Monitor network traffic and infrastructure activity.
4. Detect rejected network traffic automatically.
5. Demonstrate practical AWS security concepts.
6. Test security controls using real network traffic.
7. Manage infrastructure using Infrastructure as Code.
8. Produce security evidence suitable for a cloud/networking portfolio.

---

## Architecture

The environment contains a custom AWS VPC with separate public and private network segments.

```text
                         Internet
                            |
                            |
                   +------------------+
                   | Internet Gateway |
                   +------------------+
                            |
                            |
                    +---------------+
                    |   AWS VPC     |
                    |  10.0.0.0/16  |
                    +---------------+
                       /          \
                      /            \
                     /              \
          +----------------+    +----------------+
          | Public Subnet  |    | Private Subnet |
          | 10.0.1.0/24   |    | 10.0.2.0/24    |
          |                |    |                |
          | EC2 Server     |    | Future Private |
          | Nginx          |    | Workloads      |
          +----------------+    +----------------+
                  |
                  |
        +----------------------+
        | Security Group       |
        | Network ACL          |
        +----------------------+
                  |
                  v
        +----------------------+
        | VPC Flow Logs        |
        +----------------------+
                  |
                  v
        +----------------------+
        | CloudWatch Logs      |
        | Metric Filter        |
        | Security Alarms      |
        +----------------------+

        AWS API Activity
               |
               v
        +----------------------+
        | AWS CloudTrail       |
        | S3 Log Storage       |
        +----------------------+
```

### Architecture Screenshot

<img width="1408" height="768" alt="h" src="https://github.com/user-attachments/assets/e53142df-0fa0-412d-bbc2-f1d182740957" />

---

# Infrastructure Components

## 1. AWS VPC

<img width="1906" height="793" alt="2  VPC" src="https://github.com/user-attachments/assets/e4f4e05f-a581-4825-b78b-63fe9c92e922" />

A dedicated VPC is created with:

```text
CIDR: 10.0.0.0/16
Region: ap-south-1
```

The VPC provides an isolated networking environment for the project.

Enabled features:

* DNS support
* DNS hostnames
* Custom routing
* Public subnet
* Private subnet

---

## 2. Public Subnet

```text
CIDR: 10.0.1.0/24
Availability Zone: ap-south-1a
```

The public subnet contains the EC2 security-test server.

Instances launched in this subnet can receive public IPv4 addresses and communicate through the Internet Gateway according to their routing and security rules.

<img width="1907" height="453" alt="3  VPC → Subnets" src="https://github.com/user-attachments/assets/79191751-74b5-4811-bede-404ed663bfcd" />

---

## 3. Private Subnet

```text
CIDR: 10.0.2.0/24
Availability Zone: ap-south-1a
```

The private subnet is reserved for future private workloads.

It does not have a direct Internet Gateway route.

A NAT Gateway was intentionally avoided to reduce unnecessary costs.

---

## 4. Internet Gateway

The public subnet uses an Internet Gateway for Internet connectivity.

Public route:

```text
0.0.0.0/0 → Internet Gateway
```

Local VPC traffic remains available through:

```text
10.0.0.0/16 → local
```

---

# Security Layer

SecureCloudNet uses multiple AWS security controls rather than relying on a single security mechanism.

## 5. Security Group

The EC2 instance uses a dedicated Security Group.

### Inbound Rules

| Protocol | Port | Source                 | Purpose            |
| -------- | ---: | ---------------------- | ------------------ |
| TCP      |   22 | Administrator IP `/32` | SSH administration |
| TCP      |   80 | `0.0.0.0/0`            | HTTP web traffic   |

### Outbound

Normal outbound Internet traffic is permitted.

The SSH rule is restricted to a single administrator IP instead of exposing port 22 publicly.

<img width="1909" height="549" alt="4  EC2 → Security Groups" src="https://github.com/user-attachments/assets/54b323dc-fb22-45e5-b5d3-f91aebe43edb" />

---

## 6. Network ACL

A custom Network ACL provides an additional subnet-level security layer.

Configured rules include:

* HTTP traffic on TCP port 80
* SSH traffic from the administrator IP
* TCP ephemeral ports `1024–65535`
* Outbound traffic
* Default deny behavior

This demonstrates the difference between:

```text
Security Group → Instance-level security
Network ACL    → Subnet-level security
```
<img width="1907" height="635" alt="7  Network ACL inbound" src="https://github.com/user-attachments/assets/f924cea8-dd3f-4d9b-810c-c93dfd063088" />
<img width="1907" height="635" alt="7  Network ACL inbound" src="https://github.com/user-attachments/assets/77508213-31ed-4646-a9eb-19eb5f93e13b" />

---

# EC2 Security-Test Server

## 7. EC2 Instance

The project provisions an Ubuntu EC2 instance using Terraform.

Configuration:

```text
Instance Type: t3.micro
Operating System: Ubuntu
Subnet: Public Subnet
Role: Security-Test-Server
```

The instance is used for:

* Network connectivity testing
* HTTP testing
* Nginx testing
* VPC Flow Log generation
* Security rule validation

<img width="1887" height="584" alt="5  EC2 Instance" src="https://github.com/user-attachments/assets/d3f18f0a-e5cb-4d7e-84ff-9242c4b67ba1" />

---

# Secure Administration with AWS Systems Manager

Instead of depending entirely on SSH access, the EC2 instance is configured with an IAM role containing:

```text
AmazonSSMManagedInstanceCore
```

AWS Systems Manager allows secure management of the instance without requiring SSH to be publicly exposed.

The instance was successfully verified through:

```bash
aws ssm describe-instance-information
```

and an SSM session was established using:

```bash
aws ssm start-session --target <instance-id>
```

This demonstrates practical use of:

* IAM
* EC2
* Systems Manager
* Least-privilege administration concepts

---

# Network Monitoring

## 8. VPC Flow Logs

VPC Flow Logs are enabled for the VPC.

Configuration:

```text
Traffic Type: ALL
Aggregation Interval: 60 seconds
Destination: Amazon CloudWatch Logs
Retention: 3 days
```

Flow Logs capture accepted and rejected network traffic.

Example traffic states:

```text
ACCEPT
REJECT
```

The logs provide visibility into network communication and rejected connection attempts.

<img width="1905" height="649" alt="9  vpc-flow-logs-active" src="https://github.com/user-attachments/assets/782d6358-f2a2-420a-9b1d-b1ee0c75ca71" />
<img width="1882" height="806" alt="10-vpc-flow-log-traffic" src="https://github.com/user-attachments/assets/d3e54042-b938-4a67-b6aa-25f7cd3473c5" />

---

# CloudWatch Monitoring

## 9. EC2 CPU Monitoring

A CloudWatch alarm monitors EC2 CPU utilization.

Configuration:

```text
Metric: CPUUtilization
Threshold: 70%
Evaluation Periods: 2
Period: 5 minutes
Statistic: Average
```

The alarm changes state when sustained CPU utilization exceeds the configured threshold.

<img width="1899" height="524" alt="12 cloudwatch-security-alarms" src="https://github.com/user-attachments/assets/e4fa2697-2e42-4075-9e5e-52b0c9000a16" />
<img width="1869" height="825" alt="13 EC2 CPU Alarm — OK" src="https://github.com/user-attachments/assets/8348d24b-4416-4dab-8bcf-8f00c0f3fb09" />
<img width="1869" height="825" alt="13 EC2 CPU Alarm — OK" src="https://github.com/user-attachments/assets/d81dea16-ee13-4dbd-aedc-a0eb3841c823" />
<img width="1869" height="825" alt="13 EC2 CPU Alarm — OK" src="https://github.com/user-attachments/assets/d6a40d03-2ebd-4ba1-8176-efaf65b210ef" />

---

## 10. Rejected Traffic Detection

SecureCloudNet includes an automated security detection mechanism for rejected network traffic.

The process is:

```text
VPC Flow Logs
      |
      v
CloudWatch Log Group
      |
      v
Metric Filter
      |
      v
Custom Security Metric
      |
      v
CloudWatch Alarm
```

The metric filter identifies flow-log records where:

```text
action = REJECT
```

The CloudWatch alarm is configured to trigger when rejected traffic exceeds the defined threshold during the monitoring period.

This provides a simple example of automated network-security detection.

> A rejected connection does not automatically mean malicious activity. The project uses rejected traffic as a security monitoring signal that requires investigation.

---

# AWS CloudTrail

## 11. CloudTrail Activity Logging

AWS CloudTrail is enabled to record AWS API activity.

Configuration includes:

* Multi-region trail disabled for cost-conscious lab design
* Global service events enabled
* Log file validation enabled
* S3 log storage
* Private S3 bucket configuration

CloudTrail provides an audit trail for AWS management activity.

Example:

```bash
aws cloudtrail get-trail-status \
  --name SecureCloudNet-Trail
```

The trail was verified as actively logging without delivery errors.

---

# Security Testing

The project includes practical validation of the implemented controls.

## Tests Performed

### Security Group

Verified:

```text
TCP/22 → Administrator IP only
TCP/80 → Internet
```

### Network ACL

Verified:

```text
HTTP → Allowed
SSH → Administrator IP
Ephemeral TCP → Allowed
Other traffic → Default deny
```

### Routing

Verified:

```text
10.0.0.0/16 → local
0.0.0.0/0   → Internet Gateway
```

### Nginx

Nginx was installed and verified as running.

The server listens on:

```text
0.0.0.0:80
```

### HTTP Connectivity

External TCP connectivity to port 80 was successfully tested.

<img width="1293" height="256" alt="16 http-test-success" src="https://github.com/user-attachments/assets/0a0f5d9e-800b-4914-8c03-019afd120f18" />

### Flow Log Verification

Both accepted and rejected traffic were observed in VPC Flow Logs.

### Security Alarm

Rejected traffic successfully triggered the CloudWatch security alarm.

---

# Infrastructure as Code

All major infrastructure is managed using Terraform.

The project follows an Infrastructure as Code approach so that the AWS environment can be reproduced and managed through configuration files instead of manual console configuration.

## Terraform Resources

The infrastructure includes resources for:

* VPC
* Subnets
* Internet Gateway
* Route Tables
* Security Groups
* Network ACL
* EC2
* IAM Role
* IAM Instance Profile
* CloudWatch Log Group
* VPC Flow Logs
* CloudWatch Metric Filters
* CloudWatch Alarms
* CloudTrail
* S3 CloudTrail log storage

---

---



# Terraform Deployment

Navigate to the Terraform directory:

```bash
cd terraform
```

Initialize Terraform:

```bash
terraform init
```

Format the configuration:

```bash
terraform fmt
```

Validate the configuration:

```bash
terraform validate
```

Review the deployment plan:

```bash
terraform plan
```

Apply the infrastructure:

```bash
terraform apply
```


---

# Technologies Used

| Technology       | Purpose                      |
| ---------------- | ---------------------------- |
| AWS VPC          | Network isolation            |
| AWS Subnets      | Network segmentation         |
| Internet Gateway | Internet connectivity        |
| Route Tables     | Traffic routing              |
| Security Groups  | Instance-level firewall      |
| Network ACLs     | Subnet-level traffic control |
| EC2              | Security-test server         |
| IAM              | Access control               |
| Systems Manager  | Secure instance management   |
| VPC Flow Logs    | Network traffic monitoring   |
| CloudWatch       | Monitoring and alerting      |
| CloudTrail       | AWS API auditing             |
| S3               | CloudTrail log storage       |
| Terraform        | Infrastructure as Code       |
| Ubuntu           | Server operating system      |
| Nginx            | HTTP testing                 |

---

# Skills Demonstrated

This project demonstrates practical experience in:

* AWS Cloud Networking
* VPC Architecture
* IPv4 Networking
* Subnetting
* Routing
* Internet Gateways
* Security Groups
* Network ACLs
* IAM
* AWS Systems Manager
* Network Traffic Monitoring
* VPC Flow Logs
* CloudWatch
* CloudTrail
* Security Event Detection
* Linux Administration
* Nginx
* Infrastructure as Code
* Terraform
* AWS CLI
* Cloud Security
* Security Testing
* Cost-Aware Cloud Architecture

---

## Author

**Udeesha Jayendra**
