# Café infrastructure with CloudFormation

SEIS 615 Cloud Computing — CloudFormation assignment. The two class lab templates were updated to build the
scalable, highly available café from the previous assignment: VPC with public and private subnets in two
Availability Zones, a NAT gateway per AZ, a bastion host, an Application Load Balancer, an Auto Scaling group
and a Multi-AZ RDS MariaDB database.

| File | What it is |
|---|---|
| `lab-network.yaml` | Updated network stack (VPC, subnets, NAT gateways, bastion) |
| `lab-application.yaml` | Updated application stack (ALB, Auto Scaling group, RDS, security groups, IAM role) |
| `lab-network.orig.yaml`, `lab-application.orig.yaml` | The unmodified class lab templates, for comparison |

**Every change made to the lab files is marked with a `# CHANGE:` comment.**

## Deploy

```bash
aws cloudformation deploy --stack-name lab-network --template-file lab-network.yaml \
  --parameter-overrides WorkstationIp=<your-ip>/32 KeyName=<your-key-pair> EnvironmentName=cafe-cfn

aws cloudformation deploy --stack-name lab-application --template-file lab-application.yaml \
  --capabilities CAPABILITY_IAM \
  --parameter-overrides NetworkStackName=lab-network AsgMinSize=2 AsgMaxSize=4 AppAmiId=<cafe-ami> \
      DbSnapshotIdentifier=<optional-rds-snapshot>
```

Key parameters: `WorkstationIp` (the only address allowed to SSH to the bastion), `AsgMinSize` / `AsgMaxSize`
(Auto Scaling group size), `AppAmiId`, instance and database sizes, CIDR ranges, `DbMultiAZ`,
`CpuTargetValue`. See the `Parameters` section of each file.

Notes: the AMI ID is specific to one account and Region, so pass your own café AMI. No password is stored in the
templates; the app database password is read at boot from an existing SSM SecureString
(`DbAppPasswordParameter`).

## Clean up

```bash
aws cloudformation delete-stack --stack-name lab-application
aws cloudformation wait stack-delete-complete --stack-name lab-application
aws cloudformation delete-stack --stack-name lab-network
```
