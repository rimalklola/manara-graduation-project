````markdown
# Scalable Web Application on AWS

## Project Overview

This project implements a scalable, secure, and production-inspired web application architecture on AWS.

The infrastructure is deployed inside a custom VPC with public and private subnets distributed across two Availability Zones. The application uses EC2 instances managed by an Auto Scaling Group, an Application Load Balancer for traffic distribution, CloudFront for content delivery, AWS WAF for protection, Amazon RDS PostgreSQL for the database, AWS Systems Manager for secure instance access, and CloudWatch with SNS for monitoring and alerts.

The main objectives of the project are:

- High availability
- Automatic scaling
- Secure network isolation
- Application-layer protection
- Secure instance administration
- Managed database integration
- Monitoring and notifications

---

## Architecture Diagram

<img width="1448" height="1086" alt="lucidchart" src="https://github.com/user-attachments/assets/55147061-98f0-4ddf-abbc-e3c05e1d0e0f" />


### Traffic Flow

```text
Users
  ↓
CloudFront
  ↓
AWS WAF
  ↓
Application Load Balancer
  ↓
Target Group
  ↓
EC2 Auto Scaling Group
  ↓
Amazon RDS PostgreSQL
````

---

## Architecture Workflow

### 1. User Request

Users access the application through the CloudFront distribution URL.

CloudFront acts as the public entry point of the application. It caches content at AWS edge locations, reduces latency, and forwards requests to the Application Load Balancer.

```text
User → CloudFront
```

---

### 2. AWS WAF

AWS WAF is associated with the CloudFront distribution.

It inspects incoming requests before they reach the application infrastructure.

Managed rules are used to help protect the application from:

* SQL injection
* Common web exploits
* Known malicious inputs
* Suspicious IP addresses

```text
CloudFront → AWS WAF
```

---

### 3. Application Load Balancer

Requests accepted through CloudFront are forwarded to the Application Load Balancer.

The ALB is deployed across two public subnets in different Availability Zones.

The load balancer listens on HTTP port 80 and forwards traffic to the application's Target Group.

```text
WAF → ALB → Target Group
```

The Target Group performs health checks and sends traffic only to healthy EC2 instances.

---

## Network Design

The infrastructure is deployed inside a custom VPC.

### VPC CIDR

```text
10.0.0.0/16
```
<img width="1547" height="672" alt="vpc" src="https://github.com/user-attachments/assets/0537fe52-c538-471e-a110-638cb83a535d" />

### Subnets

| Subnet               | CIDR           | Purpose                               |
| -------------------- | -------------- | ------------------------------------- |
| Public Subnet A      | `10.0.1.0/24`  | ALB and NAT Gateway                   |
| Public Subnet B      | `10.0.2.0/24`  | ALB and NAT Gateway                   |
| Private App Subnet A | `10.0.11.0/24` | EC2 instances                         |
| Private App Subnet B | `10.0.12.0/24` | EC2 instances                         |
| Private DB Subnet A  | `10.0.21.0/24` | Amazon RDS                            |
| Private DB Subnet B  | `10.0.22.0/24` | Database subnet / future Multi-AZ use |

The public subnets have access to the Internet Gateway.

The private application subnets use NAT Gateways for outbound internet access.

The database subnets remain private.

---

## EC2 and Auto Scaling

The application runs on EC2 instances located inside the private application subnets.

The instances are created automatically using an EC2 Launch Template.

### Launch Template Configuration

* Amazon Linux 2023
* `t3.micro` instance type
* Nginx web server
* No public IP address
* IAM role for Systems Manager
* IMDSv2 enabled
* Automatic configuration through User Data

The User Data script installs and starts Nginx automatically when an instance launches.

### Auto Scaling Configuration

```text
Minimum capacity: 2
Desired capacity: 2
Maximum capacity: 4
```

The Auto Scaling Group is connected to the Application Load Balancer through the Target Group.

A target tracking policy monitors average CPU utilization and can automatically increase or decrease the number of EC2 instances.

```text
Target Group
    ↓
Auto Scaling Group
   ↙             ↘
EC2 Instance   EC2 Instance
```

The instances are distributed across two Availability Zones to improve availability.

---

## Load Balancing
<img width="932" height="526" alt="lb" src="https://github.com/user-attachments/assets/638f4817-c921-4273-b038-295429a4e381" />

The Application Load Balancer distributes incoming requests between healthy EC2 instances.

### Target Group Configuration

```text
Protocol: HTTP
Port: 80
Health Check Path: /
```
<img width="947" height="622" alt="target groups" src="https://github.com/user-attachments/assets/27826b99-8d0f-4289-ab34-3a34b96c223b" />

Only healthy targets receive traffic.

If an instance becomes unhealthy, the Auto Scaling Group can replace it automatically.

---

## Database Layer

Amazon RDS PostgreSQL is used as the database backend.

The database is deployed inside private database subnets and is not publicly accessible.

Only the application security group is allowed to connect to PostgreSQL.

```text
EC2
 ↓
TCP 5432
 ↓
RDS PostgreSQL
```

### Database Configuration

* Engine: PostgreSQL
* Public access: Disabled
* Port: `5432`
* Private subnet group
* Dedicated database security group

The lab deployment uses a Single-AZ database due to AWS Free Plan limitations.

A production deployment would use Multi-AZ RDS for automatic failover and improved availability.

---

## Security Groups
<img width="1612" height="432" alt="sgs" src="https://github.com/user-attachments/assets/c6e6fdcb-07f8-4007-9664-f9e371b9faf5" />

Three main security groups are used.

### ALB Security Group

Allows public web traffic:

```text
HTTP 80  ← 0.0.0.0/0
HTTPS 443 ← 0.0.0.0/0
```
<img width="915" height="467" alt="thewebapp" src="https://github.com/user-attachments/assets/afaada5a-dc86-4a75-bcf7-fc6626adcb1c" />

### Application Security Group

Allows HTTP traffic only from the ALB security group:

```text
HTTP 80 ← ALB Security Group
```

The EC2 instances are therefore not directly exposed to the internet.

### Database Security Group

Allows PostgreSQL traffic only from the application security group:

```text
PostgreSQL 5432 ← Application Security Group
```

This prevents direct public access to the database.

---

## Systems Manager

The EC2 instances are accessed using AWS Systems Manager Session Manager instead of SSH.

An IAM role containing the following managed policy is attached to the instances:

```text
AmazonSSMManagedInstanceCore
```

This approach provides secure administration without:

* Public IP addresses
* SSH keys
* Opening port 22 to the internet

```text
Administrator
      ↓
Systems Manager
      ↓
Private EC2 Instance
```

---

## CloudFront

Amazon CloudFront is deployed in front of the Application Load Balancer.

The ALB is configured as the CloudFront origin.

CloudFront provides:

* Edge caching
* Lower latency
* HTTPS access for users
* Reduced traffic to the origin
* Integration with AWS WAF

The communication between CloudFront and the ALB uses HTTP port 80 in this lab.

```text
User
 ↓
CloudFront
 ↓
ALB
```
<img width="942" height="427" alt="cloudfront" src="https://github.com/user-attachments/assets/f1ab4ceb-c0aa-453a-a758-fc42daf1c507" />

---

## AWS WAF

AWS WAF is associated with the CloudFront distribution.

Managed rule groups are used to provide protection against common web attacks.

Examples include:

* AWS Managed Common Rules
* Known Bad Inputs
* Amazon IP Reputation List
* SQL Injection Rules
<img width="592" height="492" alt="alarm" src="https://github.com/user-attachments/assets/f32ca338-2d1c-4f41-b3f6-9face9b62295" />

The traffic flow becomes:

```text
User
 ↓
CloudFront
 ↓
AWS WAF
 ↓
Application Load Balancer
```

---

## Monitoring and Notifications

Amazon CloudWatch is used to monitor the infrastructure.

CloudWatch collects metrics from components such as:

* EC2
* Auto Scaling
* Application Load Balancer
* RDS

CloudWatch alarms were created for important events such as:

* High EC2 CPU utilization
* Unhealthy ALB targets

Amazon SNS is used to send email notifications when an alarm is triggered.

```text
AWS Resource
     ↓
CloudWatch
     ↓
CloudWatch Alarm
     ↓
SNS Topic
     ↓
Email Notification
```

A CloudWatch dashboard is also used to monitor infrastructure metrics.

---


```

---

## AWS Services Used

| AWS Service               | Purpose                                          |
| ------------------------- | ------------------------------------------------ |
| Amazon VPC                | Network isolation                                |
| Public / Private Subnets  | Separate public, application, and database tiers |
| NAT Gateway               | Outbound access for private EC2 instances        |
| Amazon EC2                | Application compute                              |
| Auto Scaling Group        | Automatic scaling and instance replacement       |
| Application Load Balancer | Traffic distribution                             |
| Target Groups             | Health checks and backend routing                |
| Amazon CloudFront         | CDN and edge delivery                            |
| AWS WAF                   | Web application security                         |
| Amazon RDS PostgreSQL     | Managed database                                 |
| AWS Systems Manager       | Secure EC2 administration                        |
| Amazon CloudWatch         | Metrics and alarms                               |
| Amazon SNS                | Email notifications                              |

---

## Testing Performed

The following tests were completed successfully:

* Application accessed through the Application Load Balancer
* Target Group reported healthy EC2 instances
* Auto Scaling Group successfully launched EC2 instances
* CloudFront successfully served the application
* AWS WAF was associated with CloudFront
* EC2 instances were accessed using Systems Manager
* EC2 instances successfully communicated with RDS
* CloudWatch collected infrastructure metrics
* SNS email notification subscription was configured

---

## Final Architecture Workflow

```text
                         Users
                           │
                           ▼
                      CloudFront
                           │
                           ▼
                        AWS WAF
                           │
                           ▼
              Application Load Balancer
                    /             \
                   /               \
          EC2 Instance          EC2 Instance
          Private AZ-A          Private AZ-B
                   \               /
                    \             /
                      RDS PostgreSQL
                     Private DB Tier


EC2 / ALB / RDS
       │
       ▼
   CloudWatch
       │
       ▼
      SNS
       │
       ▼
     Email
```

---

## Future Improvements

Possible improvements for a production deployment include:

* Multi-AZ RDS
* Route 53 custom domain
* ACM SSL/TLS certificate on the ALB
* AWS Secrets Manager for database credentials
* Terraform Infrastructure as Code
* GitHub Actions CI/CD
* Docker container deployment

---

## Conclusion

This project demonstrates the deployment of a scalable and secure web application architecture on AWS.

The solution separates the public, application, and database layers using VPC networking, provides high availability with multiple Availability Zones and Auto Scaling, protects the application using AWS WAF, improves performance through CloudFront, and provides monitoring and notifications through CloudWatch and SNS.

```
```
