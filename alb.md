# Application Load Balancer + ASG Template

This repository now includes `asg-alb-template.yaml`, which provisions an internet-facing Application Load Balancer, a target group, and an EC2 Auto Scaling Group fed by a custom launch template. The ASG uses a target-tracking scaling policy based on the `ALBRequestCountPerTarget` metric so capacity adjusts with incoming web traffic.

## Parameters

| Name | Type | Default | Description |
| --- | --- | --- | --- |
| `VpcId` | `AWS::EC2::VPC::Id` | _none_ | Existing VPC that hosts the ALB and Auto Scaling subnets. |
| `SubnetIds` | `List<AWS::EC2::Subnet::Id>` | _none_ | Shared subnets for both the ALB and the ASG instances. |
| `LaunchTemplateSecurityGroupId` | `AWS::EC2::SecurityGroup::Id` | _none_ | Security group applied to EC2 instances. |
| `AlbSecurityGroupId` | `AWS::EC2::SecurityGroup::Id` | _none_ | Security group applied to the ALB. |
| `AmiId` | `AWS::EC2::Image::Id` | _none_ | AMI used by the launch template. |
| `InstanceType` | `String` | `t3.micro` | EC2 instance size. |
| `KeyName` | `AWS::EC2::KeyPair::KeyName` | _none_ | SSH key pair for instance access. |
| `TargetRequestsPerInstance` | `Number` | `300` | Request count per target that the scaling policy aims to maintain. |

## Resources

1. **ApplicationLoadBalancer** – Internet-facing ALB deployed across the provided subnets with the ALB security group.
2. **AlbTargetGroup** – HTTP target group listening on port 80, health-checking `/`, and expecting instance targets.
3. **HttpListener** – Port 80 listener that forwards all traffic to the target group.
4. **WebLaunchTemplate** – Launch template defining AMI, instance type, key pair, network interface (public IP + SG), tags, and a bootstrap script that installs and starts nginx with a simple status page.
5. **AutoScalingGroup** – Spans the supplied subnets, registers instances with the target group, and sets `MinSize=1`, `DesiredCapacity=2`, `MaxSize=4`.
6. **AlbRequestTargetTrackingPolicy** – Target-tracking scaling policy on `ALBRequestCountPerTarget`, dynamically scaling the ASG to maintain the configured request count per instance.

## Outputs

- **LoadBalancerDNSName** – DNS entry of the ALB for quick testing.
- **TargetGroupArn** – ARN of the target group (useful for listener rules or sharing with other stacks).
- **AutoScalingGroupName** – Logical ASG name for operations/automation.

## Deployment Tips

- Plug in existing networking resource IDs via parameters to keep this stack focused on compute and load balancing.
- Adjust `TargetRequestsPerInstance` to match your workload’s acceptable concurrency; lower values scale out faster.
- Swap the UserData block as needed to bootstrap your actual application instead of the sample nginx page.


