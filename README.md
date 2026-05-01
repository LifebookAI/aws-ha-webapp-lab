# AWS Multi-AZ Three-Tier Web Architecture Lab

## Overview

This project documents a hands-on AWS cloud engineering lab where I designed, built, troubleshot, validated, documented, and cleaned up a production-style three-tier web architecture.

The design uses an internet-facing Application Load Balancer in public subnets, private EC2 application instances across multiple Availability Zones, NAT-based outbound access for private instances, and a private RDS MySQL database layer protected by security groups.

The goal of this lab was not just to deploy resources, but to understand how AWS networking, routing, security groups, load balancing, private compute, and private databases work together in a real architecture.

---

## Architecture

Internet -> Application Load Balancer -> Private EC2 App Layer -> Private RDS Database Layer

Only the Application Load Balancer is public-facing. The EC2 instances and RDS database are private.

---

## AWS Services Used

- Amazon VPC
- Public and private subnets
- Internet Gateway
- NAT Gateway
- Route tables
- Security groups
- Application Load Balancer
- Target groups
- Amazon EC2
- Amazon RDS for MySQL
- RDS DB subnet group

---

## Network Design

The VPC was divided into three logical tiers:

| Tier | Purpose |
|---|---|
| Public subnets | Host the internet-facing Application Load Balancer |
| Private app subnets | Host EC2 application instances with no direct public access |
| Private DB subnets | Host the RDS database layer |

Routing design:

| Route Table | Purpose |
|---|---|
| Public route table | Routes 0.0.0.0/0 to the Internet Gateway |
| App route table | Routes 0.0.0.0/0 to the NAT Gateway for outbound access |
| DB route table | Local VPC routing only; no direct internet route |

Evidence:

- [Public route table to Internet Gateway](screenshots/03-public-route-table-igw-routes.jpeg)
- [Public subnet associations](screenshots/03b-public-route-table-subnet-associations.jpeg)
- [Private app route table to NAT Gateway](screenshots/04a-app-route-table-nat-route.png)
- [Private DB route table local-only](screenshots/05-db-route-table-private-only.jpeg)
- [Private DB subnet associations](screenshots/05b-db-route-table-subnet-associations.jpeg)

---

## Security Design

Security groups were configured using layered access:

| Security Group | Inbound Rule |
|---|---|
| ALB security group | Allows HTTP/HTTPS from the internet |
| App security group | Allows HTTP only from the ALB security group |
| DB security group | Allows MySQL only from the App security group |

This means users can reach the load balancer, but cannot directly reach the EC2 instances or RDS database.

Evidence:

- [App security group allows HTTP only from ALB SG](screenshots/07-app-security-group.jpeg)
- [RDS private network/security settings](screenshots/17-rds-private-network-settings.png)

---

## Application Load Balancer and EC2 Validation

The ALB was configured with an HTTP listener forwarding traffic to an EC2 target group.

The app instances were launched in private app subnets and served a simple Apache test page using EC2 user data.

Evidence:

- [ALB listener forwarding to target group](screenshots/12-alb-listener-forwarding.png)
- [Working ALB DNS page - app-b](screenshots/13-alb-dns-working-app-b.png)
- [Working ALB DNS page - app-a](screenshots/14-alb-dns-working-app-a.png)

---

## Troubleshooting Highlight

During the build, the ALB targets initially showed as unhealthy.

Root cause:

The EC2 instances were launched in private app subnets before NAT-based outbound internet access was configured. Their user data script attempted to install Apache using dnf, but the instances had no outbound internet route to reach package repositories.

Because Apache did not install or start, the ALB health check on port 80 failed.

Fix:

1. Created a NAT Gateway.
2. Added a 0.0.0.0/0 route from the private app route table to the NAT Gateway.
3. Relaunched the EC2 app instances so user data could complete successfully.
4. Registered the new instances with the target group.
5. Confirmed both targets became healthy.

Evidence:

- [Targets unhealthy before NAT](screenshots/09-targets-unhealthy-before-nat.png)
- [NAT Gateway available](screenshots/10-nat-gateway-available.png)
- [Targets healthy after NAT fix](screenshots/11-targets-healthy-after-nat.png)

This troubleshooting process demonstrated the difference between private subnet inbound isolation and outbound dependency access.

---

## RDS Private Database Layer

An RDS MySQL database was created in a private DB subnet group. Public accessibility was disabled, and database access was restricted through the DB security group.

The RDS instance was intentionally kept small for cost control. For a production system, Multi-AZ RDS would be enabled for database failover.

Evidence:

- [RDS DB subnet group](screenshots/15-rds-db-subnet-group.png)
- [RDS instance available](screenshots/16-rds-available.png)
- [RDS private network/security settings](screenshots/17-rds-private-network-settings.png)

---

## Cost Control and Cleanup

After validation and screenshots, cost-incurring resources were deleted.

Resources cleaned up:

- RDS database
- NAT Gateway
- EC2 instances
- Application Load Balancer
- Target group
- Security groups
- Route tables
- Subnets
- Internet Gateway
- VPC

Cleanup was documented in:

- [Cleanup notes](notes/cleanup.md)

---

## What I Learned

- How to design a multi-AZ VPC architecture
- How public and private subnets differ
- How route tables control traffic paths
- How Internet Gateway and NAT Gateway serve different purposes
- Why private EC2 instances need outbound access for package installs and bootstrapping
- How ALB target groups and health checks work
- How to isolate EC2 and RDS with security group references
- How to troubleshoot unhealthy ALB targets
- How to validate a working AWS architecture
- How to document and clean up AWS resources professionally

---

## Future Improvements

- Rebuild this architecture using Terraform
- Add an architecture diagram PNG
- Add CloudWatch alarms for unhealthy target count
- Add HTTPS with ACM and Route 53
- Add AWS Systems Manager Session Manager for private instance access
- Enable Multi-AZ RDS for a production version
