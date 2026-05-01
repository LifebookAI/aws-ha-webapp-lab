# Troubleshooting Notes

## Issue

The Application Load Balancer target group initially showed EC2 targets as unhealthy.

## Root Cause

The EC2 instances were launched in private app subnets before NAT-based outbound access was configured.

The user data script attempted to install Apache using dnf, but the instances had no outbound internet route to reach package repositories.

Because Apache did not install or start, the ALB health check on port 80 failed.

## Fix

1. Created a NAT Gateway.
2. Added a 0.0.0.0/0 route from the private app route table to the NAT Gateway.
3. Relaunched the EC2 app instances so user data could run successfully.
4. Registered the new instances with the target group.
5. Confirmed both targets became healthy.

## Lesson

Private subnets protect instances from direct inbound internet access, but private instances may still need outbound access for updates, packages, or bootstrap scripts.

NAT Gateway provides outbound internet access without allowing inbound public access to private EC2 instances.
