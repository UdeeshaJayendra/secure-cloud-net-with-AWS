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

The project is designed to remain **cost-conscious**, avoiding expensive components such as NAT Gateways and unnecessary always-on services.

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

![AWS Architecture](docs/screenshots/aws-architecture.png)

> Add your final architecture diagram to `docs/screenshots/aws-architecture.png`.

---

# Infrastructure Components

## 1. AWS VPC

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

![VPC and Subnets](docs/screenshots/vpc-subnets.png)

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

![Route Tables](docs/screenshots/route-tables.png)

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

![Security Group](docs/screenshots/phase5-security-group.png)

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

![Network ACL](docs/screenshots/phase5-network-acl.png)

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

![EC2 Instance](docs/screenshots/ec2-instance.png)

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

![VPC Flow Logs](docs/screenshots/phase5-flow-logs.png)

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

![CPU Alarm](docs/screenshots/cloudwatch-cpu-alarm.png)

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

![Rejected Traffic Alarm](docs/screenshots/phase5-rejected-traffic-alarm.png)

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

![HTTP Test](docs/screenshots/phase5-http-test-success.png)

### Flow Log Verification

Both accepted and rejected traffic were observed in VPC Flow Logs.

### Security Alarm

Rejected traffic successfully triggered the CloudWatch security alarm.

![Nginx Running](docs/screenshots/phase5-nginx-running.png)

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

# Project Structure

```text
SecureCloudNet/
│
├── .gitignore
├── README.md
├── PROJECT_PROGRESS.md
│
├── terraform/
│   ├── provider.tf
│   ├── variables.tf
│   ├── main.tf
│   ├── outputs.tf
│   └── .terraform.lock.hcl
│
├── scripts/
│
├── docs/
│   └── screenshots/
│       ├── aws-architecture.png
│       ├── vpc-subnets.png
│       ├── route-tables.png
│       ├── ec2-instance.png
│       ├── cloudwatch-cpu-alarm.png
│       ├── phase5-security-group.png
│       ├── phase5-network-acl.png
│       ├── phase5-http-test-success.png
│       ├── phase5-nginx-running.png
│       ├── phase5-flow-logs.png
│       └── phase5-rejected-traffic-alarm.png
│
└── architecture/
```

---

# Deployment

## Prerequisites

Install:

* AWS CLI
* Terraform
* Git
* An AWS account

Verify the installations:

```bash
aws --version
terraform version
git --version
```

---

## Configure AWS CLI

Configure your AWS credentials locally:

```bash
aws configure
```

Set the AWS region to:

```text
ap-south-1
```

Verify the authenticated identity:

```bash
aws sts get-caller-identity
```

> Never commit AWS access keys, secret keys, session tokens, or private keys to GitHub.

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

Confirm the deployment when Terraform asks for approval.

---

# Verify Infrastructure

Check Terraform state:

```bash
terraform state list
```

Check EC2 instances:

```bash
aws ec2 describe-instances \
  --query "Reservations[*].Instances[*].[InstanceId,State.Name,PrivateIpAddress,PublicIpAddress]" \
  --output table
```

Check VPC Flow Logs:

```bash
aws ec2 describe-flow-logs \
  --query "FlowLogs[*].[FlowLogId,FlowLogStatus,TrafficType]" \
  --output table
```

Check CloudWatch alarms:

```bash
aws cloudwatch describe-alarms \
  --output table
```

Check CloudTrail:

```bash
aws cloudtrail get-trail-status \
  --name SecureCloudNet-Trail
```

---

# Destroy Infrastructure

When the lab is no longer required, destroy the resources to avoid unnecessary AWS charges.

From the Terraform directory:

```bash
terraform destroy
```

Review the resources carefully before confirming.

> AWS billing depends on the resources and usage in your account. The project intentionally avoids expensive always-on components such as NAT Gateway, but users should still monitor their AWS billing and Free Tier usage.

---

# Cost-Conscious Design

The project was designed with a low-cost laboratory environment in mind.

Cost-control decisions include:

* `t3.micro` EC2 instance
* No NAT Gateway
* Short CloudWatch log retention
* Limited monitoring resources
* No unnecessary load balancers
* No managed Kubernetes cluster
* No expensive security appliances
* Manual destruction after lab usage

An AWS Budget/Cost Alert can also be configured to notify the account owner when spending approaches a chosen threshold.

**Important:** AWS Budgets provide alerts but do not guarantee that spending will never exceed the configured amount.

---

# Security Considerations

This project intentionally demonstrates several security principles.

### Least Privilege

Administrative access is restricted where practical, particularly SSH access.

### Defense in Depth

Multiple controls are used:

```text
Security Group
      +
Network ACL
      +
VPC Flow Logs
      +
CloudWatch Monitoring
      +
CloudTrail
```

### Visibility

Network and AWS API activity are logged for investigation.

### Secure Administration

AWS Systems Manager provides an alternative management mechanism to direct SSH access.

### Credential Protection

Sensitive credentials and private keys are excluded from Git using `.gitignore`.

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

# Project Evidence

Screenshots documenting the implementation are stored in:

```text
docs/screenshots/
```

Recommended evidence includes:

1. AWS VPC and subnet configuration
2. Route tables
3. Internet Gateway
4. EC2 instance
5. Security Group
6. Network ACL
7. Nginx running
8. HTTP connectivity test
9. VPC Flow Logs
10. CloudWatch CPU alarm
11. Rejected traffic alarm
12. CloudTrail configuration
13. Final Terraform plan

---

# Final Terraform Validation

Before completing the project, Terraform was verified with:

```bash
terraform plan
```

Result:

```text
No changes. Your infrastructure matches the configuration.
```

This confirms that the deployed infrastructure is synchronized with the Terraform configuration.

---

# Future Improvements

Possible future enhancements include:

* Automated security testing
* AWS Config rules
* GuardDuty integration
* SNS security notifications
* CloudWatch dashboards
* More advanced traffic analysis
* Private EC2 workloads
* Bastion-free administration using SSM
* Automated incident-response scripts
* Additional Terraform modules
* CI/CD validation using GitHub Actions

These features can be added while maintaining the project's cost-conscious design.

---

# Conclusion

**SecureCloudNet** demonstrates how AWS networking, security controls, monitoring, logging, and Infrastructure as Code can be combined into a practical cloud security laboratory.

The project goes beyond simply creating an AWS VPC by implementing:

```text
Infrastructure
      ↓
Network Security
      ↓
Traffic Monitoring
      ↓
Security Detection
      ↓
Audit Logging
      ↓
Security Testing
      ↓
Terraform Automation
```

The result is a reproducible AWS environment that demonstrates practical **Cloud Networking, Cybersecurity, AWS, Linux, Monitoring, and DevOps skills**.

---

## Author

**Udeesha Jayendra**

GitHub: [UdeeshaJayendra](https://github.com/UdeeshaJayendra)

Project Repository:

[SecureCloudNet — Automated AWS Network Security & Monitoring Lab](https://github.com/UdeeshaJayendra/secure-cloud-net-with-AWS)
