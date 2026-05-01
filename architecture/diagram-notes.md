# Architecture Diagram Notes

## Final Architecture

Internet -> Application Load Balancer -> Private EC2 App Layer -> Private RDS Database Layer

## Components

- VPC CIDR: 10.0.0.0/16
- Public subnets across two Availability Zones
- Private app subnets across two Availability Zones
- Private DB subnets across two Availability Zones
- Internet Gateway for public subnet internet access
- NAT Gateway for outbound internet access from private app subnets
- Application Load Balancer in public subnets
- EC2 app instances in private app subnets
- RDS MySQL database in private DB subnet group
- Security group chain:
  - Internet -> ALB SG
  - ALB SG -> App SG
  - App SG -> DB SG

## Design Goal

Only the load balancer is publicly reachable. EC2 and RDS remain private.
