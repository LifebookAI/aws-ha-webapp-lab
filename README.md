# AWS Multi-AZ Three-Tier Web Architecture Lab

## Overview

This project documents a hands-on AWS lab where I built a production-style three-tier web architecture.

The architecture includes:

- Public Application Load Balancer
- Private EC2 app instances
- NAT Gateway for outbound access
- Private RDS MySQL database
- Route table isolation
- Security group layered access
- ALB target group health checks
- Troubleshooting and cleanup documentation

## Architecture

Internet -> ALB -> Private EC2 App Layer -> Private RDS Database Layer

## Status

Built, validated, documented with screenshots, and cleaned up to control cost.
