# Cloud Cost Incinerator

## Overview

An AI agent with access to AWS Cost Explorer and billing data that identifies cost optimization opportunities and provides ready-to-run commands to save money. Not vague suggestions - actual executable commands with estimated savings.

## Problem Statement

- Cloud costs grow unchecked without active monitoring
- Unused resources accumulate (zombie resources)
- Teams lack visibility into their actual spend
- Cost optimization advice is often generic and not actionable
- Engineers don't have time to hunt for savings

## Proposed Solution

### Core Features

1. **Full Cost Visibility**
   - AWS Cost Explorer API integration
   - Billing data analysis
   - Resource inventory across all regions
   - Tag-based cost allocation

2. **Waste Detection**
   - Idle EC2 instances
   - Unattached EBS volumes
   - Unused Elastic IPs
   - Inactive SageMaker notebooks
   - Empty S3 buckets with costs
   - Orphaned snapshots
   - Unused NAT Gateways
   - VPCs in unused regions
   - Over-provisioned RDS instances

3. **Actionable Commands**
   - Ready-to-run AWS CLI commands
   - Terraform destroy snippets
   - CloudFormation delete commands
   - Estimated savings per action

4. **Safety Features**
   - Dry-run mode
   - Impact assessment
   - Rollback instructions
   - Approval workflow

## Technical Architecture

### Data Sources

```
+------------------+     +------------------+     +------------------+
| Cost Explorer    |     | CloudWatch       |     | Resource APIs    |
+------------------+     +------------------+     +------------------+
         |                       |                       |
         +-----------------------+-----------------------+
                                 |
                                 v
                       +------------------+
                       | Data Aggregator  |
                       +------------------+
                                 |
                                 v
                       +------------------+
                       | AI Analyzer      |
                       +------------------+
                                 |
                                 v
                       +------------------+
                       | Command Generator|
                       +------------------+
                                 |
                                 v
                       +------------------+
                       | Action Report    |
                       +------------------+
```

### Detection Rules

| Resource Type | Detection Criteria | Example Command |
|--------------|---------------------|-----------------|
| EC2 Instance | CPU < 5% for 14 days | `aws ec2 stop-instances --instance-ids i-xxx` |
| EBS Volume | Unattached > 30 days | `aws ec2 delete-volume --volume-id vol-xxx` |
| Elastic IP | Not associated | `aws ec2 release-address --allocation-id eipalloc-xxx` |
| SageMaker Notebook | No activity 7 days | `aws sagemaker stop-notebook-instance --notebook-instance-name xxx` |
| RDS Instance | CPU < 10%, connections < 5 | `aws rds modify-db-instance --db-instance-identifier xxx --db-instance-class db.t3.small` |
| NAT Gateway | 0 bytes processed | `aws ec2 delete-nat-gateway --nat-gateway-id nat-xxx` |
| S3 Bucket | Empty, has lifecycle costs | `aws s3 rb s3://bucket-name` |

## Output Format

### Cost Optimization Report

```
AWS COST INCINERATOR REPORT
===========================
Account: 123456789012
Period: Last 30 days
Total Spend: $15,432.50

IMMEDIATE SAVINGS AVAILABLE: $2,847.00/month

HIGH PRIORITY (Save $1,500/month):
----------------------------------

1. Idle SageMaker Notebooks (3 found)
   - ml-experiment-jan (us-east-1): No activity for 45 days
   - data-analysis-old (us-west-2): No activity for 90 days
   - test-notebook (eu-west-1): No activity for 30 days

   Current Cost: $450/month

   Commands:
   aws sagemaker stop-notebook-instance --notebook-instance-name ml-experiment-jan --region us-east-1
   aws sagemaker stop-notebook-instance --notebook-instance-name data-analysis-old --region us-west-2
   aws sagemaker stop-notebook-instance --notebook-instance-name test-notebook --region eu-west-1

   Or run all at once:
   ./generated-scripts/stop-idle-notebooks.sh

2. Unused VPCs in Obscure Regions
   - ap-south-2 (Hyderabad): VPC with NAT Gateway, no resources
   - af-south-1 (Cape Town): VPC with 2 subnets, no resources

   Current Cost: $65/month (NAT Gateway charges)

   Commands:
   aws ec2 delete-nat-gateway --nat-gateway-id nat-xxx --region ap-south-2
   # Wait for NAT Gateway deletion, then:
   aws ec2 delete-vpc --vpc-id vpc-xxx --region ap-south-2

3. Over-provisioned RDS Instances
   - prod-analytics-db: db.r5.2xlarge, avg CPU 8%
   - Recommendation: Downsize to db.r5.large

   Current Cost: $800/month
   Projected Cost: $200/month
   Savings: $600/month

   Command (schedule during maintenance window):
   aws rds modify-db-instance \
     --db-instance-identifier prod-analytics-db \
     --db-instance-class db.r5.large \
     --apply-immediately

MEDIUM PRIORITY (Save $800/month):
----------------------------------

4. Unattached EBS Volumes (12 found)
   Total Size: 2.4 TB
   Current Cost: $240/month

   Commands:
   aws ec2 delete-volume --volume-id vol-0a1b2c3d4e5f --region us-east-1
   aws ec2 delete-volume --volume-id vol-1b2c3d4e5f6 --region us-east-1
   [...]

   Or run:
   ./generated-scripts/delete-unattached-volumes.sh

GENERATED SCRIPTS:
------------------
All commands saved to: ./cost-incinerator-output/
- stop-idle-notebooks.sh
- delete-unattached-volumes.sh
- cleanup-unused-vpcs.sh
- rightsize-rds.sh (requires maintenance window)

TOTAL POTENTIAL SAVINGS: $2,847.00/month ($34,164/year)
```

## Deliverables

1. AWS credential integration (IAM role recommended)
2. Cost analysis engine
3. Resource scanner across all regions
4. AI recommendation engine
5. Command generator with safety checks
6. Web dashboard
7. Scheduled scan capability

## Safety Features

1. **Dry Run Mode**: Preview all commands without execution
2. **Impact Assessment**: Show dependencies before deletion
3. **Exclusion Tags**: Respect `DoNotDelete` or similar tags
4. **Approval Workflow**: Require confirmation for destructive actions
5. **Audit Log**: Track all actions taken

## Success Metrics

- Monthly savings achieved
- Number of zombie resources eliminated
- Time saved vs manual review
- Cost trend improvement

## Permissions Required

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ce:GetCostAndUsage",
        "ce:GetCostForecast",
        "ec2:Describe*",
        "rds:Describe*",
        "sagemaker:List*",
        "sagemaker:Describe*",
        "s3:ListAllMyBuckets",
        "s3:GetBucketLocation",
        "cloudwatch:GetMetricStatistics"
      ],
      "Resource": "*"
    }
  ]
}
```

## Open Questions

- How to handle shared resources across teams?
- What's the approval workflow for production resources?
- How to integrate with existing FinOps tools?
- Should we support GCP and Azure as well?
