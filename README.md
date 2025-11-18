# VPC CloudFormation Template

This repository contains a single CloudFormation template (`vpc.yaml`) that builds a production-ready VPC with optional high-availability settings. The stack creates public and private subnets across two Availability Zones, attaches an internet gateway, provisions NAT gateways, configures routing, and associates explicit network ACLs with each subnet.

## Features

- Two public and two private subnets stretched across AZ1 and AZ2
- Internet gateway plus NAT gateway in AZ1 by default, with optional AZ2 NAT
- Public and private route tables (and an additional private RT when the second NAT is enabled)
- Dedicated network ACLs per subnet tier for future fine-grained controls
- Tier-specific security groups for public and private workloads
- Parameterized CIDR ranges, environment naming, and AZ selection

## Parameters

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `EnvironmentName` | `String` | `dev` | Prefix for tagging resources (e.g., `dev-vpc`). |
| `VpcCIDR` | `String` | `10.0.0.0/16` | CIDR block for the VPC. |
| `PublicSubnet1CIDR` | `String` | `10.0.101.0/24` | CIDR for Public Subnet AZ1. |
| `PublicSubnet2CIDR` | `String` | `10.0.102.0/24` | CIDR for Public Subnet AZ2. |
| `PrivateSubnet1CIDR` | `String` | `10.0.1.0/24` | CIDR for Private Subnet AZ1. |
| `PrivateSubnet2CIDR` | `String` | `10.0.2.0/24` | CIDR for Private Subnet AZ2. |
| `CreateSecondaryNatGateway` | `String` (`true`/`false`) | `false` | When `true`, adds a second NAT gateway, EIP, route table, and route for AZ2. |
| `AvailabilityZone1` | `AWS::EC2::AvailabilityZone::Name` | `us-east-1a` | AZ for the `*-a` subnets. |
| `AvailabilityZone2` | `AWS::EC2::AvailabilityZone::Name` | `us-east-1b` | AZ for the `*-b` subnets. |

## Conditions

- `CreateNatInAz2`: Evaluates to `true` when `CreateSecondaryNatGateway` is set to `true`. Controls creation of the secondary NAT, Elastic IP, private route table, and the route association for the AZ2 private subnet.

## Optional High Availability Path

- **Single NAT (default):** Only `NatGateway` in AZ1 is deployed. Both private subnets share the primary private route table; `PrivateSubnet2` uses the AZ1 NAT (cross-AZ).
- **Dual NAT (set parameter to `true`):** Adds `NatGateway2`, `NatEip2`, `PrivateRouteTableAz2`, and `PrivateRouteAz2`. The `PrivateSubnet2RouteTableAssociation` switches to the AZ2 route table, eliminating cross-AZ dependency for outbound traffic.

## Network ACLs

Each subnet tier (public/private) has a dedicated Network ACL resource and subnet associations. The current rules allow all inbound/outbound traffic to mirror default AWS behavior while giving you explicit resources to tighten rules later.

## Security Groups

- `PublicSecurityGroup`: Allows HTTP, HTTPS, and SSH ingress from the internet and all outbound traffic.
- `PrivateSecurityGroup`: Allows all TCP traffic from within the VPC CIDR and allows all outbound traffic.

## Outputs

- `VpcId`: Exported as `<env>-vpc-id`.
- `PublicSubnetIds`: Comma-delimited list of the two public subnets.
- `PrivateSubnetIds`: Comma-delimited list of the two private subnets.


Adjust parameters as needed (for example, set `CreateSecondaryNatGateway=true` when you want per-AZ NAT coverage).
