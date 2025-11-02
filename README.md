# AWS Ansible Automation

This repository contains Ansible playbooks for managing and reporting on AWS infrastructure.

## 📋 Prerequisites

- Python 3.x
- Ansible 2.9+
- AWS CLI
- boto3 and botocore Python libraries

### Installation

```bash
# Install Ansible
pip install ansible

# Install AWS collections
ansible-galaxy collection install amazon.aws
ansible-galaxy collection install community.aws

# Install boto3 and botocore
pip install boto3 botocore
```

## 🔐 AWS Credentials Setup

### Option 1: Using AWS Credentials File (Recommended)

Create a credentials file:

```bash
mkdir -p aws_credentials
```

Create `aws_credentials/aws_credentials.yml`:

```yaml
---
# For SSO Mode
aws_mode: sso
aws_profile: "your-sso-profile"
aws_region: "us-east-1"

# For Static Credentials Mode
# aws_mode: static
# static_aws_access_key: "YOUR_ACCESS_KEY"
# static_aws_secret_key: "YOUR_SECRET_KEY"
# static_aws_session_token: "YOUR_SESSION_TOKEN"  # Optional, for temporary credentials
# aws_region: "us-east-1"
```

### Option 2: Using Environment Variables

```bash
export AWS_ACCESS_KEY_ID="your_access_key"
export AWS_SECRET_ACCESS_KEY="your_secret_key"
export AWS_SESSION_TOKEN="your_session_token"  # Optional
export AWS_DEFAULT_REGION="us-east-1"
```

### Option 3: Using AWS CLI Configuration

```bash
aws configure
```

## 📂 Project Structure

```
aws-ansible/
├── ansible.cfg                 # Ansible configuration
├── inventory/
│   └── aws_ec2.yml            # Dynamic EC2 inventory
├── aws_credentials/           # AWS credentials (gitignored)
│   └── aws_credentials.yml
├── aws_ec2_state.yml          # EC2 instances report
├── aws_s3_list.yml            # S3 buckets report
├── aws_sg_list.yml            # Security groups report
├── aws_vpc_list.yml           # VPC and networking report
├── aws_alb_list.yml           # Application Load Balancers report
├── aws_iam_list.yml           # IAM roles, policies, users report
├── aws_cloudwatch_list.yml    # CloudWatch alarms report
└── README.md
```

## 🚀 Usage

### Running Playbooks

```bash
# List EC2 instances
ansible-playbook aws_ec2_state.yml -v

# List S3 buckets
ansible-playbook aws_s3_list.yml -v

# List Security Groups
ansible-playbook aws_sg_list.yml -v

# List VPCs and Subnets
ansible-playbook aws_vpc_list.yml -v

# List Application Load Balancers
ansible-playbook aws_alb_list.yml -v

# List IAM resources
ansible-playbook aws_iam_list.yml -v

# List CloudWatch Alarms
ansible-playbook aws_cloudwatch_list.yml -v
```

### Using Dynamic Inventory

The dynamic inventory automatically discovers EC2 instances:

```bash
# List all discovered hosts
ansible-inventory -i inventory/aws_ec2.yml --list

# Ping all running instances
ansible all -i inventory/aws_ec2.yml -m ping
```

## 📊 Playbook Descriptions

### `aws_ec2_state.yml`
Lists all EC2 instances with:
- Instance ID
- Name tag
- Current state (running, stopped, etc.)

### `aws_s3_list.yml`
Lists all S3 buckets with:
- Bucket name
- Creation date
- Optional: Objects in specific buckets

### `aws_sg_list.yml`
Lists all Security Groups with:
- Security Group ID and Name
- VPC association
- Ingress (inbound) rules
- Egress (outbound) rules

### `aws_vpc_list.yml`
Lists networking resources:
- VPCs with CIDR blocks
- Subnets with availability zones
- Internet Gateways and attachments

### `aws_alb_list.yml`
Lists Application Load Balancers with:
- ALB details (DNS, scheme, state)
- Listeners (protocols, ports)
- Target groups
- CloudWatch metrics (last hour)

### `aws_iam_list.yml`
Lists IAM resources:
- IAM Roles
- IAM Policies
- IAM Users
- Permission summary

### `aws_cloudwatch_list.yml`
Lists CloudWatch Alarms with:
- Alarm name and state
- Metric configuration
- Alarm actions
- Summary by state

## 🔒 Security Best Practices

1. **Never commit credentials** - All credential files are in `.gitignore`
2. **Use IAM roles** when running from EC2 instances
3. **Use temporary credentials** with session tokens when possible
4. **Rotate credentials** regularly
5. **Use Ansible Vault** for encrypting sensitive data:

```bash
# Encrypt credentials file
ansible-vault encrypt aws_credentials/aws_credentials.yml

# Run playbook with vault
ansible-playbook aws_ec2_state.yml --ask-vault-pass
```

## 🐛 Troubleshooting

### "Unable to locate credentials" Error

Ensure credentials are properly configured:
1. Check `aws_credentials/aws_credentials.yml` exists and contains valid credentials
2. Or set environment variables
3. Or run `aws configure`

### "Access Denied" Error

Your IAM user/role needs these permissions:
- `ec2:DescribeInstances`
- `s3:ListAllMyBuckets`
- `ec2:DescribeSecurityGroups`
- `ec2:DescribeVpcs`
- `elasticloadbalancing:DescribeLoadBalancers`
- `iam:ListRoles`, `iam:ListPolicies`, `iam:ListUsers`
- `cloudwatch:DescribeAlarms`

### Dynamic Inventory Not Working

1. Check `inventory/aws_ec2.yml` has valid credentials
2. Test with: `ansible-inventory -i inventory/aws_ec2.yml --graph`
3. Ensure EC2 instances are running and tagged properly

## 📝 Configuration

### ansible.cfg

```ini
[defaults]
inventory = ./inventory/aws_ec2.yml
remote_user = ec2-user
private_key_file = ~/.ssh/my_aws_key.pem
host_key_checking = False
```

### Dynamic Inventory (inventory/aws_ec2.yml)

```yaml
plugin: amazon.aws.aws_ec2
regions:
  - us-east-1
filters:
  instance-state-name: running
keyed_groups:
  - key: tags.Type
    prefix: tag_Type
```

## 📄 License

MIT License

## 🔗 Resources

- [Ansible AWS Guide](https://docs.ansible.com/ansible/latest/collections/amazon/aws/)
- [AWS CLI Documentation](https://docs.aws.amazon.com/cli/)
- [Boto3 Documentation](https://boto3.amazonaws.com/v1/documentation/api/latest/index.html)