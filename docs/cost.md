# Cost Model

## Overview

This is a reference cost model. Actual costs vary by usage, region, and configuration.

## Key Cost Drivers

- NAT Gateway per AZ (recommended for HA) costs approximately $100/month per gateway in data processing fees alone [inferred from AWS public pricing]; a single shared NAT cuts cost but creates a hidden single point of failure for all private subnet egress [editorial].
- Aurora Serverless v2 would cut idle RDS cost to near zero for low-traffic periods, but the current provisioned Aurora cluster in this stack runs approximately $150-200/month minimum regardless of load [inferred from AWS RDS pricing for the instance class configured]; that trade-off is only justified if baseline traffic is consistently high enough to saturate a provisioned instance [editorial].
- CloudFront in front of S3 adds approximately $0.0085 per 10k HTTPS requests [inferred from AWS CloudFront public pricing] but eliminates S3 egress charges and provides edge caching [from-code via OAC configuration]; skipping CloudFront saves construct complexity but causes the S3 bill to scale linearly with traffic [editorial].
- Fargate removes EC2 patching overhead but costs 20-30% more per vCPU-hour than equivalent EC2 capacity [inferred from AWS Fargate vs EC2 public pricing]; at low task counts this premium is irrelevant, but at 50+ tasks running continuously it becomes a material cost conversation [editorial].
- KMS CMK for RDS adds approximately $1/month per key plus $0.03 per 10k API calls [inferred from AWS KMS public pricing]; cost is negligible and the compliance value is non-negotiable [editorial], but a drifted key policy that removes the Lambda rotation function's decrypt permission will silently break rotation rather than throwing a deployment error [from-code structure, inferred failure mode].

## Estimated Monthly Cost

| Component | Dev (₹) | Staging (₹) | Production (₹) |
|-----------|---------|-------------|-----------------|
| Compute   | ₹2,000–5,000 | ₹8,000–15,000 | ₹25,000–60,000 |
| Database  | ₹1,500–3,000 | ₹5,000–12,000 | ₹15,000–40,000 |
| Networking| ₹500–1,000   | ₹2,000–5,000  | ₹5,000–15,000  |
| Monitoring| ₹200–500     | ₹1,000–2,000  | ₹3,000–8,000   |
| **Total** | **₹4,200–9,500** | **₹16,000–34,000** | **₹48,000–1,23,000** |

> Estimates based on ap-south-1 (Mumbai) pricing. Actual costs depend on traffic, data volume, and reserved capacity.

## Cost Optimization Strategies

- Use Savings Plans or Reserved Instances for predictable workloads
- Enable auto-scaling with conservative scale-in policies
- Use DynamoDB on-demand for dev, provisioned for production
- Leverage S3 Intelligent-Tiering for infrequently accessed data
- Review Cost Explorer weekly for anomalies
