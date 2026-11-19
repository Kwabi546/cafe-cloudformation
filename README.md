# Cafe app on AWS with CloudFormation

CloudFormation templates that rebuild the highly available cafe application from the
earlier assignments (VPC, NAT gateways, bastion, Multi-AZ RDS, load balancer, Auto Scaling).
They are updated versions of the class templates `lab-network` and `lab-application`.
Every change from the class templates is marked with a `# CHANGE n:` comment in the YAML files.

## Files

| File | What it builds |
|------|----------------|
| `lab-network.yaml` | VPC, 2 public and 2 private subnets in 2 Availability Zones, internet gateway, 2 NAT gateways with Elastic IPs, route tables |
| `lab-application.yaml` | Security groups, bastion, Multi-AZ RDS MariaDB, the 7 `/cafe/*` Secrets Manager secrets, launch template, target group, Application Load Balancer, Auto Scaling group with CPU target tracking |
| `kiro-prompts-cloudformation.md` | The Kiro (AI agent) prompts used to deploy and clean up the stacks |

## Architecture

```
Internet -> ALB (public subnets A and B)
              -> Auto Scaling group (private subnets A and B, min/max are parameters)
                   -> RDS MariaDB Multi-AZ (private subnets)
Your workstation IP only -> Bastion (public subnet A) -> SSH to the private servers
Private servers -> NAT gateway in their own AZ -> internet (package installs)
```

## Parameters

`lab-network.yaml`: `ProjectName`, `VpcCidr`, `PublicSubnetACidr`, `PublicSubnetBCidr`,
`PrivateSubnetACidr`, `PrivateSubnetBCidr` (all have defaults).

`lab-application.yaml` (main ones):

| Parameter | Purpose | Default |
|-----------|---------|---------|
| `WorkstationIp` | Your public IP. The bastion accepts SSH only from `<ip>/32` | none, required |
| `AsgMinSize` / `AsgMaxSize` | Auto Scaling group size | 2 / 4 |
| `CpuTargetValue` | Target average CPU (percent) | 50 |
| `DBPassword` | Database admin password (NoEcho) | none, required |
| `AppBucketName` / `AppBucketPrefix` | S3 bucket and folder holding the course files | none / `cafe-app` |
| `InstanceType`, `KeyName`, `InstanceProfileName` | Servers, key pair, existing IAM instance profile | `t3.micro`, `cafe-app-key-pair`, `cafe-app-role` |
| `DBInstanceClass`, `DBEngineVersion`, `DBAllocatedStorage`, `DBMultiAZ` | Database settings | `db.t3.micro`, `10.6.25`, `20`, `true` |
| `ShowServerInfo`, `TimeZone`, `Currency` | Cafe app settings stored as secrets | `false`, `America/New_York`, `$` |
| `NetworkStackName` | Name of the network stack to import from | `lab-network` |

## Prerequisites

- An S3 bucket with `cafe.zip`, `aws.zip`, `aws.phar` and `db.zip` in a folder named `cafe-app`
  (the course files are not stored in this repository).
- The IAM instance profile `cafe-app-role` and an EC2 key pair.

## Deploy

```
aws cloudformation create-stack --stack-name lab-network --template-body file://lab-network.yaml \
  --tags Key=Project,Value=cafe-cfn-assignment

aws cloudformation create-stack --stack-name lab-application --template-body file://lab-application.yaml \
  --parameters ParameterKey=NetworkStackName,ParameterValue=lab-network \
               ParameterKey=WorkstationIp,ParameterValue=<your-ip> \
               ParameterKey=AppBucketName,ParameterValue=<your-bucket> \
               ParameterKey=DBPassword,ParameterValue="$CAFE_DB_PASSWORD" \
  --tags Key=Project,Value=cafe-cfn-assignment
```

Then open the `URL` output of the application stack. The application stack takes about
20 to 30 minutes because of the Multi-AZ database.

## Clean up

Delete `lab-application`, then force-delete the seven `/cafe/*` secrets
(`aws secretsmanager delete-secret --secret-id <name> --force-delete-without-recovery`),
then delete `lab-network`, then empty and delete the S3 bucket.

## Notes

- No password, IP address or account ID is stored in this repository.
