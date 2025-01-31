# Prerequisites

## AWS Marketplace subscriptions

### [Zebrunner Selenium Grid Agent](https://aws.amazon.com/marketplace/pp/prodview-qykvcpnstrlzi?sr=0-2&ref_=beagle&applicationId=AWSMPContessa)

Linux based ECS optimized instance with embedded Zebrunner tuning for scalable and reliable browser images usage

### [Zebrunner Selenium Grid Agent - Windows](https://aws.amazon.com/marketplace/pp/prodview-wmwdyq54i36jy?sr=0-4&ref_=beagle&applicationId=AWSMPContessa)

Windows based ECS optimized instance with embedded Zebrunner tuning for scalable and reliable browser images usage

## E3S instance

### Hardware

* Network optimized instance m5n.large+
* Configured IMDSv2 (HttpPutResponseHopLimit=2)

### Software

* Installed Docker v19+
* Installed Docker compose plugin v2+
* [Optional] Installed jq and aws cli for ./scripts support

### Attached IAM Role/Policies:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "WithoutConstraints",
            "Effect": "Allow",
            "Action": [
                "ecs:RegisterTaskDefinition",
                "ecs:ListTasks",
                "ec2:DescribeInstances",
                "ec2:DescribeInstanceStatus",
                "ec2:DescribeInstanceTypes",
                "elasticloadbalancing:DescribeTargetGroups",
                "autoscaling:DescribeAutoScalingGroups",
                "autoscaling:DescribeAutoScalingInstances"
            ],
            "Resource": "*"
        },
        {
            "Sid": "ECS",
            "Effect": "Allow",
            "Action": [
                "ecs:DescribeContainerInstances",
                "ecs:DescribeTasks",
                "ecs:StopTask",
                "ecs:DescribeClusters",
                "ecs:ListContainerInstances",
                "ecs:RunTask",
                "ecs:DescribeCapacityProviders",
                "ecs:UpdateContainerInstancesState"
            ],
            "Resource": [
                "arn:aws:ecs:${Region}:${Account}:container-instance/e3s-${env}/*",
                "arn:aws:ecs:${Region}:${Account}:task/e3s-${env}/*",
                "arn:aws:ecs:${Region}:${Account}:cluster/e3s-${env}",
                "arn:aws:ecs:${Region}:${Account}:task-definition/${env}-*",
                "arn:aws:ecs:${Region}:${Account}:capacity-provider/e3s-${env}-*"
            ]
        },
        {
            "Sid": "Autoscaling",
            "Effect": "Allow",
            "Action": [
                "autoscaling:TerminateInstanceInAutoScalingGroup",
                "autoscaling:SetInstanceProtection",
                "autoscaling:SetDesiredCapacity"
            ],
            "Resource": "arn:aws:autoscaling:${Region}:${Account}:autoScalingGroup:*:autoScalingGroupName/e3s-${env}-*"
        },
        {
            "Sid": "ELB",
            "Effect": "Allow",
            "Action": [
                "elasticloadbalancing:RegisterTargets",
                "elasticloadbalancing:DeregisterTargets"
            ],
            "Resource": "arn:aws:elasticloadbalancing:${Region}:${Account}:targetgroup/e3s-${env}-*"
        },
        {
            "Sid": "S3",
            "Effect": "Allow",
            "Action": [
                "s3:ListBucket",
                "s3:GetObject"
            ],
            "Resource": [
                "arn:aws:s3:::zebrunner.${env}-engine",
                "arn:aws:s3:::zebrunner.${env}-engine/*"
            ]
        },
        {
            "Sid": "IAM",
            "Effect": "Allow",
            "Action": [
                "iam:passRole"
            ],
            "Resource": "arn:aws:iam::${Account}:role/e3s-${env}-task-role"
        }
    ]
}
```

## Agent instance

### Hardware

* Compute optimized instance c5a.2xlarge +
* Configured IMDSv2 (HttpPutResponseHopLimit=1)

### Attached IAM Role/Policies:

#### Agent role
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "WithoutConstraints",
            "Effect": "Allow",
            "Action": [
                "ecs:DiscoverPollEndpoint",
                "logs:CreateLogStream",
                "logs:PutLogEvents"
            ],
            "Resource": "*"
        },
        {
            "Sid": "ECS",
            "Effect": "Allow",
            "Action": [
                "ecs:DeregisterContainerInstance",
                "ecs:RegisterContainerInstance",
                "ecs:Submit*",
                "ecs:StartTelemetrySession",
                "ecs:UpdateContainerInstancesState",
                "ecs:Poll"
            ],
            "Resource": [
                "arn:aws:ecs:${Region}:${Account}:cluster/e3s-${env}",
                "arn:aws:ecs:${Region}:${Account}:container-instance/e3s-${env}/*"
            ]
        }
    ]
}
```

#### Task role
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::zebrunner.${env}-engine/*"
            ]
        }
    ]
}
```
