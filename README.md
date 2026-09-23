# manara-graduation-project
please find the screenshots in the file 

AWS Services Used/
VPC with public and private subnets/
EC2 + Auto Scaling Group/
Application Load Balancer/
CloudFront/
AWS WAF/
RDS PostgreSQL/
Systems Manager Session Manager/
CloudWatch/
SNS/
Network Design
VPC: 10.0.0.0/16
Public subnets: 10.0.1.0/24, 10.0.2.0/24
Private app subnets: 10.0.11.0/24, 10.0.12.0/24
Private DB subnets: 10.0.21.0/24, 10.0.22.0/24

The ALB is deployed in public subnets, while EC2 and RDS remain private.

Auto Scaling
Minimum capacity: 2
Desired capacity: 2
Maximum capacity: 4
Target tracking based on CPU utilization
Security
EC2 instances have no public IP
SSH port 22 is not exposed
EC2 access uses Systems Manager
RDS is private
ALB is the only public application entry point
AWS WAF protects CloudFront
Monitoring

CloudWatch monitors the infrastructure and triggers SNS email notifications for alarms such as:

High EC2 CPU usage
Unhealthy ALB targets
Database

Amazon RDS PostgreSQL is used as the backend database.

