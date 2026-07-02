# terraform-aws-eks-module

A reusable Terraform module for deploying production-ready Amazon EKS clusters with VPC networking, ALB ingress controller support, and CloudWatch logging.

## What It Does

- Provisions a VPC with public/private subnets across 3 AZs
- Deploys an EKS cluster with managed node groups
- Configures NAT Gateway for outbound connectivity
- Sets up IAM roles for ALB Controller (IRSA)
- Creates CloudWatch Log Group for cluster audit logs
- Uses Spot instances in dev to reduce costs by ~60%

## Tech Stack

- **IaC**: Terraform 1.0+
- **Cloud**: AWS (EKS, VPC, ALB, CloudWatch, IAM)
- **Kubernetes**: 1.29
- **Modules**: terraform-aws-modules/vpc, terraform-aws-modules/eks

## Architecture Diagram

```
                    ┌─────────────────────────────┐
                    │         Internet            │
                    └─────────────┬───────────────┘
                                  │
                    ┌─────────────▼───────────────┐
                    │     Application LB          │
                    │    (ALB Controller)         │
                    └─────────────┬───────────────┘
                                  │
                    ┌─────────────▼───────────────┐
                    │      EKS Cluster            │
                    │  ┌─────────────────────┐    │
                    │  │  Managed Node Group │    │
                    │  │  (t3.medium, SPOT)  │    │
                    │  │  - 2 desired nodes  │    │
                    │  │  - 1-5 auto-scaling │    │
                    │  └─────────────────────┘    │
                    └─────────────┬───────────────┘
                                  │
        ┌─────────────────────────┼─────────────────────────┐
        │                         │                         │
   ┌────▼────┐              ┌──────▼──────┐           ┌──────▼──────┐
   │  VPC    │              │   VPC       │           │   VPC       │
   │ us-east-1a│            │ us-east-1b  │           │ us-east-1c  │
   │         │              │             │           │             │
   │Public   │              │Public       │           │Public       │
   │Private  │              │Private      │           │Private      │
   └─────────┘              └─────────────┘           └─────────────┘
        │                         │                         │
   ┌────▼────┐              ┌────▼────┐              ┌────▼────┐
   │ NAT GW  │              │ NAT GW  │              │ NAT GW  │
   └─────────┘              └─────────┘              └─────────┘

CloudWatch Logs: /aws/eks/cluster-01/cluster
```

## How to Run It

### Prerequisites

- AWS CLI configured (`aws configure`)
- Terraform >= 1.0 installed
- kubectl installed (optional, for cluster verification)

### Deployment Steps

```bash
# Clone and enter directory
cd terraform-aws-eks-module

# Initialize Terraform
terraform init

# Review the plan
terraform plan

# Apply (takes ~15 minutes)
terraform apply

# Configure kubectl
aws eks update-kubeconfig --region us-east-1 --name cluster-01

# Verify nodes
kubectl get nodes
```

### Cost Optimization Notes

- Dev environment uses **SPOT instances** (saves ~60%)
- Single NAT Gateway in dev (multi-AZ in prod)
- CloudWatch logs retention set to 7 days (adjust in prod)

## Inputs

See `variables.tf` for all configurable options including:
- `aws_region` (default: us-east-1)
- `cluster_name` (default: cluster-01)
- `node_instance_types` (default: t3.medium)
- `environment` (default: dev)

## Outputs

- `cluster_name` — EKS cluster identifier
- `cluster_endpoint` — Kubernetes API endpoint
- `vpc_id` — VPC identifier for additional resource attachment
- `node_security_group_id` — Security group for node-to-node communication

