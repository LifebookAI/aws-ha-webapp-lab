# Cleanup Notes

After validation and screenshots, cost-incurring resources were deleted.

## Cleanup Order Used

1. Delete RDS database
2. Delete NAT Gateway
3. Release Elastic IP if allocated separately
4. Terminate EC2 instances
5. Delete Application Load Balancer
6. Delete target group
7. Delete RDS subnet group
8. Delete security groups
9. Delete route tables
10. Delete subnets
11. Detach and delete Internet Gateway
12. Delete VPC

## Reason

NAT Gateway, RDS, EC2, and ALB can continue generating charges while running.

Cleaning up after validation demonstrates cost awareness and responsible cloud engineering practice.
