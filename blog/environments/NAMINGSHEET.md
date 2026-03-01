# AWS Resource Naming Cheatsheet

Quick reference guide for naming AWS resources following the Derrops conventions. All examples assume **account-segregated environments** (preferred approach).

**Core Format:** `{org}--{domain}--{service}--{key}` (compound kebab-case with `--` segment delimiters)

---

## Template Variables Reference

Before using this cheatsheet, understand what each placeholder means:

| Variable | Definition | Example | Required? | Notes |
|----------|-----------|---------|-----------|-------|
| `{region}` | AWS region code | `ap-southeast-2`, `us-east-1`, `eu-west-1` | ✅ Only for globally unique services (S3, CloudFront, ACM, Route53) | Omit if using account-per-region segregation |
| `{env}` | Deployment environment | `prod`, `dev`, `staging`, `uat` | ✅ Only for globally unique services (S3) or DNS | Omit if using account-per-environment segregation (recommended) |
| `{org}` | Organization/top-level business unit | `acme`, `mycompany`, `client-name` | ✅ Always required | Most stable segment; rarely changes |
| `{domain}` | Business capability / bounded domain | `payments`, `identity`, `analytics`, `platform` | ✅ Always required | Owned independently; more stable than teams |
| `{service}` | Deployable service unit | `checkout-api`, `auth-service`, `webhook-worker` | ✅ Always required | The primary identity; can be renamed/refactored |
| `{key}` | Specific resource or config within service | `transactions`, `webhook-secret`, `primary`, `cache` | ✅ Always required | Purpose-specific identifier; changes frequently |
| `{partition}` | Data partition grouping (logs, events only) | `2024/01/15/14`, `2024-01-15` | ❌ Optional; data storage only | Only for time-series or partitioned data in S3/Glue |
| `{purpose}` | Functional purpose | `alb`, `db`, `lambda`, `encryption-enabled` | ✅ When needed for clarity | Qualifies the resource type |
| `{type}` | Resource subtype | `web`, `worker`, `private`, `public`, `primary`, `replica` | ✅ When distinguishing variants | Makes specific instances identifiable |
| `{az}` | Availability zone | `1a`, `1b`, `1c` | ✅ For multi-AZ resources | Ensures subnets are distributed |
| `{consumer}` | API consumer/client | `mobile-client`, `partner-integrations`, `internal` | ✅ For API keys and access | Identifies who consumes the resource |
| `{target}` | Target system for data source | `dynamodb`, `rds`, `lambda`, `s3` | ✅ For integration points | What the resource connects to |
| `{num}` | Sequential number | `01`, `02`, `03` | ❌ Optional; for instance naming | Zero-padded for sorting |
| `{yyyy}/{mm}/{dd}/{hh}` | Date/time partitions | `2024/01/15/14` | ✅ For time-series data only | Each level is independently queryable |
| `{file}` | Filename | `transactions.json`, `logs.parquet` | ✅ For object/file storage | The final artifact identifier |
| `{version}` or `{tag}` | Semantic version or release tag | `1.2.3`, `latest`, `v2.0.0-beta` | ✅ For images and artifacts | Identifies specific release |
| `{registry}` | Container registry host | `123456789.dkr.ecr.ap-southeast-2.amazonaws.com`, `docker.io` | ✅ For container images | Where the image is hosted |

---

## Summary Table - All Services

| Service | Format | Pattern | Delimiter | Example |
|---------|--------|---------|-----------|---------|
| **S3 Bucket** | Global + prefix | `ap-southeast-2--prod--{org}--{domain}--{service}--{key}` | `-` | `ap-southeast-2--prod--acme--payments--checkout-api--backups` |
| **S3 Object Keys** | Hierarchy | `{org}/{domain}/{service}/{key}` | `/` | `acme/payments/checkout-api/schema.sql` |
| **S3 Logs/Events** | With partition | `{org}/{domain}/{service}/{yyyy}/{mm}/{dd}/{hh}/{file}` | `/` | `acme/payments/checkout-api/2024/01/15/14/transactions-00001.json` |
| **CloudWatch Logs** | Hierarchy | `/{org}/{domain}/{service}/{key}` | `/` | `/acme/payments/checkout-api/application-logs` |
| **CloudWatch Metrics** | Flat kebab | `{org}--{domain}--{service}` | `--` / `-` | `acme--payments--checkout-api` |
| **ECS Cluster** | Flat kebab | `{org}--{domain}--{service}--{key}` | `--` / `-` | `acme--payments--checkout-api--cluster` |
| **ECS Service** | Flat kebab | `{org}--{domain}--{service}` | `--` / `-` | `acme--payments--checkout-api` |
| **ECS Task Definition** | Flat kebab | `{org}--{domain}--{service}` | `--` / `-` | `acme--payments--checkout-api` |
| **ECR Repository** | Registry path | `{org}/{domain}/{service}` | `/` | `acme/payments/checkout-api` |
| **ECR Image Tag** | Semantic | `{registry}/{org}/{domain}/{service}:{version}` | `/ :` | `123456789.dkr.ecr.ap-southeast-2.amazonaws.com/acme/payments/checkout-api:1.2.3` |
| **DynamoDB Table** | Flat kebab | `{org}--{domain}--{service}--{key}` | `--` / `-` | `acme--payments--checkout-api--transactions` |
| **DynamoDB GSI** | Flat kebab | `{key}--gsi` | `--` / `-` | `transactions-by-user--gsi` |
| **RDS Instance ID** | Flat kebab | `{org}--{domain}--{service}--{key}` | `--` / `-` | `acme--payments--checkout-api--primary` |
| **RDS DB Name** | Flat snake | `{org}_{domain}_{service}` | `_` | `acme_payments_checkout_api` |
| **RDS Parameter Group** | Flat kebab | `{org}--{domain}--{service}--{key}` | `--` / `-` | `acme--payments--checkout-api--params` |
| **RDS Subnet Group** | Flat kebab | `{org}--{domain}--{service}--subnet-group` | `--` / `-` | `acme--payments--checkout-api--subnet-group` |
| **EC2 Instance** | Flat kebab | `{org}--{domain}--{service}--{type}-{num}` | `--` / `-` | `acme--payments--checkout-api--web-01` |
| **EC2 Security Group** | Flat kebab | `{org}--{domain}--{service}--{purpose}` | `--` / `-` | `acme--payments--checkout-api--alb` |
| **EC2 Volume** | Flat kebab | `{org}--{domain}--{service}--volume-{purpose}` | `--` / `-` | `acme--payments--checkout-api--volume-data` |
| **EC2 Elastic IP** | Flat kebab | `{org}--{domain}--{service}--eip` | `--` / `-` | `acme--payments--checkout-api--eip` |
| **Lambda Function** | Flat kebab | `{org}--{domain}--{service}--{key}` | `--` / `-` | `acme--payments--checkout-api--webhook-handler` |
| **Lambda Layer** | Flat kebab | `{org}--{domain}--{service}--{purpose}` | `--` / `-` | `acme--shared-utilities--common-libs` |
| **Lambda Alias** | Simple | `prod`, `dev`, `staging` | N/A | `prod` |
| **IAM Role** | Path + name | Path: `/{org}/{domain}/{service}/` Name: `{service}--{purpose}-role` | `/` `--` `-` | ARN: `arn:aws:iam::123456789:role/acme/payments/checkout-api/checkout-api--lambda-role` |
| **IAM Policy** | Path + name | Path: `/{org}/{domain}/{service}/` Name: `{purpose}-policy` | `/` `-` | `acme--payments--checkout-api--s3-access-policy` |
| **IAM User** | Flat kebab | `{org}--{domain}--{service}--user` | `--` / `-` | `acme--payments--checkout-api--service-user` |
| **Route53 Hosted Zone** | Subdomain | `prod.acme.com` (env from account) | `.` | `prod.acme.com` |
| **Route53 DNS Record** | Reverse hierarchy | `{service}.prod.acme.com` | `.` | `checkout-api.prod.acme.com` |
| **Route53 Private Zone** | Internal DNS | `{service}.internal.prod.acme.com` | `.` | `checkout-api.internal.prod.acme.com` |
| **CloudFront Distribution** | Flat kebab | `{org}--{domain}--{service}--cdn` | `--` / `-` | `acme--payments--checkout-api--cdn` |
| **CloudFront Alias (CNAME)** | DNS pattern | `{service}.prod.acme.com` | `.` | `checkout-api.prod.acme.com` |
| **ACM Certificate Domain** | DNS pattern | `{service}.prod.acme.com` | `.` | `checkout-api.prod.acme.com` |
| **ACM Wildcard Cert** | DNS wildcard | `*.prod.acme.com` | `.` | `*.prod.acme.com` |
| **VPC** | Flat kebab | `{org}--{domain}--{service}--vpc` | `--` / `-` | `acme--payments--checkout-api--vpc` |
| **Subnet** | Flat kebab | `{org}--{domain}--{service}--subnet-{type}-{az}` | `--` / `-` | `acme--payments--checkout-api--subnet-private-1a` |
| **Route Table** | Flat kebab | `{org}--{domain}--{service}--rt-{type}` | `--` / `-` | `acme--payments--checkout-api--rt-private` |
| **Network ACL** | Flat kebab | `{org}--{domain}--{service}--nacl` | `--` / `-` | `acme--payments--checkout-api--nacl` |
| **ALB/NLB** | Flat kebab | `{org}--{domain}--{service}--alb` | `--` / `-` | `acme--payments--checkout-api--alb` |
| **Target Group** | Flat kebab | `{org}--{domain}--{service}--tg-{purpose}` | `--` / `-` | `acme--payments--checkout-api--tg-api` |
| **SNS Topic** | Flat kebab | `{org}--{domain}--{service}--{key}` | `--` / `-` | `acme--payments--checkout-api--transactions` |
| **SQS Queue** | Flat kebab | `{org}--{domain}--{service}--{key}` | `--` / `-` | `acme--payments--checkout-api--events` |
| **SQS FIFO Queue** | Flat kebab | `{org}--{domain}--{service}--{key}.fifo` | `--` / `-` | `acme--payments--checkout-api--events.fifo` |
| **SQS DLQ** | Flat kebab | `{queue-name}--dlq` | `--` / `-` | `acme--payments--checkout-api--events--dlq` |
| **Kinesis Stream** | Flat kebab | `{org}--{domain}--{service}--{key}` | `--` / `-` | `acme--payments--checkout-api--events` |
| **EventBridge Bus** | Flat kebab | `{org}--{domain}--{service}--events` | `--` / `-` | `acme--payments--checkout-api--events` |
| **EventBridge Rule** | Flat kebab | `{org}--{domain}--{service}--{key}-rule` | `--` / `-` | `acme--payments--checkout-api--process-webhook-rule` |
| **Step Functions** | Flat kebab | `{org}--{domain}--{service}--{key}` | `--` / `-` | `acme--payments--checkout-api--order-processing` |
| **API Gateway REST API** | Flat kebab | `{org}--{domain}--{service}--api` | `--` / `-` | `acme--payments--checkout-api--api` |
| **API Gateway HTTP API** | Flat kebab | `{org}--{domain}--{service}--http-api` | `--` / `-` | `acme--payments--checkout-api--http-api` |
| **API Gateway Stage** | Simple | `prod`, `dev`, `staging` | N/A | `prod` |
| **API Gateway Key** | Flat kebab | `{org}--{domain}--{service}--{consumer}` | `--` / `-` | `acme--payments--checkout-api--mobile-client` |
| **AppSync API** | Flat kebab | `{org}--{domain}--{service}--api` | `--` / `-` | `acme--payments--checkout-api--api` |
| **AppSync Data Source** | Flat kebab | `{org}--{domain}--{service}--{target}` | `--` / `-` | `acme--payments--checkout-api--dynamodb` |
| **ElastiCache Cluster** | Flat kebab | `{org}--{domain}--{service}--cache` | `--` / `-` | `acme--payments--checkout-api--cache` |
| **ElastiCache Replication Group** | Flat kebab | `{org}--{domain}--{service}--replication-group` | `--` / `-` | `acme--payments--checkout-api--replication-group` |
| **ElastiCache Parameter Group** | Flat kebab | `{org}--{domain}--{service}--params` | `--` / `-` | `acme--payments--checkout-api--params` |
| **OpenSearch Domain** | Flat kebab | `{org}--{domain}--{service}` | `--` / `-` | `acme--payments--checkout-api` |
| **OpenSearch Index** | Hierarchy | `{org}/{domain}/{service}/{key}/{date}` | `/` | `acme/payments/checkout-api/transactions/2024-01-15` |
| **RDS Proxy** | Flat kebab | `{org}--{domain}--{service}--proxy` | `--` / `-` | `acme--payments--checkout-api--proxy` |
| **AWS Backup Plan** | Flat kebab | `{org}--{domain}--{service}--backup-plan` | `--` / `-` | `acme--payments--checkout-api--backup-plan` |
| **AWS Backup Vault** | Flat kebab | `{org}--{domain}--{service}--vault` | `--` / `-` | `acme--payments--checkout-api--vault` |
| **Glue Database** | Flat snake | `{org}_{domain}_{service}` | `_` | `acme_payments_checkout_api` |
| **Glue Table** | Flat snake | `{key}` | N/A | `transactions` |
| **Glue Job** | Flat kebab | `{org}--{domain}--{service}--{key}-job` | `--` / `-` | `acme--analytics--etl--transform-job` |
| **Glue Crawler** | Flat kebab | `{org}--{domain}--{service}--{key}-crawler` | `--` / `-` | `acme--analytics--data-crawlers` |
| **Athena Workgroup** | Flat kebab | `{org}--{domain}--{service}--workgroup` | `--` / `-` | `acme--analytics--etl--workgroup` |
| **Athena Results Bucket** | S3 key path | `s3://bucket/{org}/{domain}/{service}/` | `/` | `s3://acme-analytics-athena-results/acme/analytics/etl/` |
| **QuickSight Dataset** | Flat kebab | `{org}--{domain}--{service}--dataset` | `--` / `-` | `acme--analytics--transactions--dataset` |
| **QuickSight Analysis** | Flat kebab | `{org}--{domain}--{service}--{key}-analysis` | `--` / `-` | `acme--analytics--revenue-dashboard--analysis` |
| **QuickSight Dashboard** | Flat kebab | `{org}--{domain}--{service}--{key}` | `--` / `-` | `acme--analytics--revenue-dashboard` |
| **Redshift Cluster** | Flat kebab | `{org}--{domain}--{service}--cluster` | `--` / `-` | `acme--analytics--warehouse--cluster` |
| **Redshift Database** | Flat snake | `{org}_{domain}_{service}` | `_` | `acme_analytics_warehouse` |
| **Redshift Subnet Group** | Flat kebab | `{org}--{domain}--{service}--subnet-group` | `--` / `-` | `acme--analytics--warehouse--subnet-group` |
| **MSK Cluster** | Flat kebab | `{org}--{domain}--{service}--cluster` | `--` / `-` | `acme--events--streaming--cluster` |
| **Kafka Topic** | Dotted path | `{org}.{domain}.{service}.{key}` | `.` | `acme.payments.checkout-api.transactions` |
| **AppConfig Application** | Flat kebab | `{org}--{domain}--{service}` | `--` / `-` | `acme--payments--checkout-api` |
| **AppConfig Environment** | Simple | `prod`, `dev`, `staging` | N/A | `prod` |
| **AppConfig Profile** | Flat kebab | `{org}--{domain}--{service}--{key}-profile` | `--` / `-` | `acme--payments--checkout-api--feature-flags-profile` |
| **Systems Manager Document** | Flat kebab | `{org}--{domain}--{service}--{key}` | `--` / `-` | `acme--payments--checkout-api--runbook` |
| **Systems Manager Maintenance Window** | Flat kebab | `{org}--{domain}--{service}--maintenance` | `--` / `-` | `acme--payments--checkout-api--patching-window` |
| **Service Catalog Portfolio** | Flat kebab | `{org}--{domain}--portfolio` | `--` / `-` | `acme--payments--portfolio` |
| **Service Catalog Product** | Flat kebab | `{org}--{domain}--{service}--product` | `--` / `-` | `acme--payments--checkout-api--product` |
| **X-Ray Sampling Rule** | Flat kebab | `{org}--{domain}--{service}--sampling-rule` | `--` / `-` | `acme--payments--checkout-api--sampling-rule` |
| **Config Rule** | Flat kebab | `{org}--{domain}--{service}--{key}-rule` | `--` / `-` | `acme--payments--checkout-api--encryption-enabled-rule` |
| **Config Aggregator** | Flat kebab | `{org}--{domain}--config-aggregator` | `--` / `-` | `acme--payments--config-aggregator` |
| **Security Hub Custom Insight** | Flat kebab | `{org}--{domain}--{service}--{key}-insight` | `--` / `-` | `acme--payments--checkout-api--critical-findings-insight` |
| **WAF Web ACL** | Flat kebab | `{org}--{domain}--{service}--waf` | `--` / `-` | `acme--payments--checkout-api--waf` |
| **WAF IP Set** | Flat kebab | `{org}--{domain}--{service}--{purpose}-ipset` | `--` / `-` | `acme--payments--checkout-api--blocked-ips` |
| **WAF Rule Group** | Flat kebab | `{org}--{domain}--{service}--rules` | `--` / `-` | `acme--payments--checkout-api--rate-limit-rules` |
| **CloudFormation Stack** | Flat kebab | `{org}--{domain}--{service}--{key}-stack` | `--` / `-` | `acme--payments--checkout-api--stack` |
| **SSM Parameter** | Hierarchy | `/{org}/{domain}/{service}/{key}` | `/` | `/acme/payments/checkout-api/stripe-webhook-secret` |
| **Secrets Manager Secret** | Hierarchy | `{org}/{domain}/{service}/{key}` | `/` | `acme/payments/checkout-api/db-password` |
| **Auto Scaling Group** | Flat kebab | `{org}--{domain}--{service}--asg` | `--` / `-` | `acme--payments--checkout-api--asg` |
| **Launch Template** | Flat kebab | `{org}--{domain}--{service}--launch-template` | `--` / `-` | `acme--payments--checkout-api--launch-template` |

---

## Delimiter Decision Matrix

| Use Case | Default Delimiter | Notes | When to Override | Services |
|----------|-------------------|-------|------------------|----------|
| **Segment separator** (org↔domain↔service↔key) | `--` (double hyphen) | Separates major naming segments; most readable | When service has native hierarchy | All flat resource names |
| **Word within segment** | `-` (hyphen) | Words/parts within a single segment | Never—always use `-` for words | All resource names |
| **Path hierarchy (native)** | `/` (slash) | Native hierarchical support—use instead of `--` | Use `/` instead of `--` | S3 keys, SSM Parameters, Secrets, IAM paths, ECR, CloudWatch Logs |
| **DNS hierarchy (native)** | `.` (dot) | Native DNS subdomain separation—use instead of `--` | Use `.` instead of `--` | Route53, DNS records, CloudFront aliases, Kafka topics |
| **Image tag/version (native)** | `:` (colon) | Native registry delimiter for versioning | Use `:` after image name | ECR, Docker registries |
| **Database internal names** | `_` (underscore) | DB-friendly; only for internal schema names, NOT identifiers | Use `_` instead of `-` for DB/schema names only | RDS database names, Glue databases |

---

## Global vs Regional Scope

| Resource Type | Scope | Includes region? | Includes env? | Example |
|---------------|-------|-------------------|-----------------|---------|
| S3 Buckets | Globally unique | ✅ Yes (ap-southeast-2) | ✅ Yes (prod) | `ap-southeast-2--prod--acme--payments--checkout-api--backups` |
| Route53 Hosted Zones | Global DNS | ❌ No | ✅ Via account (prod.acme.com) | `prod.acme.com` |
| CloudFront Distributions | Global CDN | ❌ No | ✅ Via DNS (prod) | `checkout-api.prod.acme.com` |
| ACM Certificates | Global DNS | ❌ No | ✅ Via DNS (prod) | `checkout-api.prod.acme.com` |
| IAM Roles | Global (within account) | ❌ No | ❌ No (via account) | `/acme/payments/checkout-api/checkout-api-role` |
| All Regional Services | Regional | ❌ No (via account) | ❌ No (via account) | `acme-payments-checkout-api` |

---

## Native Hierarchy Support

| Service | Native Delimiter | Use It? | Alternative |
|---------|------------------|--------|-------------|
| S3 Object Keys | `/` (slash) | ✅ YES | N/A |
| SSM Parameter Store | `/` (slash) | ✅ YES | N/A |
| Secrets Manager | `/` (slash) | ✅ YES | N/A |
| IAM Paths | `/` (slash) | ✅ YES | N/A |
| ECR Repositories | `/` (slash) | ✅ YES | N/A |
| OpenSearch Indices | `/` (slash) | ✅ YES | N/A |
| Route53 DNS | `.` (dot) | ✅ YES | N/A |
| CloudWatch Logs | `/` (slash) | ✅ YES | N/A |
| Kafka Topics | `.` (dot) | ✅ YES | N/A |
| DynamoDB Tables | `-` (kebab) | ❌ NO | Use `--` for segments |
| RDS Instances | `-` (kebab) | ❌ NO | Use `--` for segments |
| Lambda Functions | `-` (kebab) | ❌ NO | Use `--` for segments |
| ECS/EC2 Resources | `-` (kebab) | ❌ NO | Use `--` for segments |

---

## Common Pitfalls & Solutions

| Pitfall | Problem | Why It Matters | Solution |
|---------|---------|----------------|----------|
| Using `_` in S3 bucket names | S3 rejects underscores | S3 bucket naming constraint | Use `-` (hyphens) everywhere |
| Inconsistent names across environments | Cannot query resources across envs | Breaks filtering in CloudWatch, Config, Security Hub | Use identical logical names in all accounts |
| Including `{env}` when account-segregated | Redundant naming; violates consistency principle | Names become longer, harder to read; breaks queries | Omit `{env}` if managing via account boundaries |
| Randomly suffixed names | Names become unpredictable | Makes automation fragile; breaks IaC | Use account/region namespace for uniqueness |
| Changing `{org}` or `{domain}` | Breaks all downstream references and policies | All IAM policies, CloudWatch filters, and automation fail | Keep these segments stable; only change `{service}` |
| Not using native delimiters | Loses prefix querying capability | Cannot use S3 prefix filtering, SSM GetParametersByPath | Use `/` for hierarchical systems, `.` for DNS |
| DNS names don't mirror resources | Routing confusion; cross-team coordination failure | Applications cannot find correct endpoints | DNS = reversed hierarchy (service.domain.org.com) |
| Resource name > service character limit | Truncation breaks convention | Names get auto-truncated; cannot predict final name | Test limits early; use shorter domain/service names |
| Mixed kebab and snake case | Tools cannot parse consistently | Scripts fail; team confusion; automation breaks | Use kebab-case (`-`) for all naming except DB internals |
| Forgetting tagging for cost allocation | Cannot allocate costs accurately | Wrong cost attribution; misleading cost reports | Tag every resource with CostCenter, Owner, Service |

---

## Quick Reference by Layer

### Infrastructure Layer
```
VPC: acme--payments--checkout-api--vpc
Subnet: acme--payments--checkout-api--subnet-private-1a
Security Group: acme--payments--checkout-api--alb
```

### Compute Layer
```
ECS Cluster: acme--payments--checkout-api--cluster
ECS Service: acme--payments--checkout-api
EC2 Instance: acme--payments--checkout-api--web-01
Lambda: acme--payments--checkout-api--webhook-handler
```

### Data Layer
```
DynamoDB: acme--payments--checkout-api--transactions
RDS Instance: acme--payments--checkout-api--primary
RDS Database: acme_payments_checkout_api
ElastiCache: acme--payments--checkout-api--cache
S3 Bucket: ap-southeast-2--prod--acme--payments--checkout-api--data
```

### Messaging Layer
```
SNS Topic: acme--payments--checkout-api--transactions
SQS Queue: acme--payments--checkout-api--events
SQS DLQ: acme--payments--checkout-api--events--dlq
Kinesis Stream: acme--payments--checkout-api--events
```

### Integration Layer
```
API Gateway: acme--payments--checkout-api--api
Step Functions: acme--payments--checkout-api--order-processing
EventBridge Rule: acme--payments--checkout-api--process-webhook-rule
```

### Observability Layer
```
CloudWatch Logs: /acme/payments/checkout-api/application-logs
CloudWatch Metrics: acme--payments--checkout-api
X-Ray Rule: acme--payments--checkout-api--sampling-rule
Config Rule: acme--payments--checkout-api--encryption-enabled-rule
```

### Security Layer
```
IAM Role: /acme/payments/checkout-api/checkout-api--lambda-role
IAM Policy: acme--payments--checkout-api--s3-access-policy
WAF Web ACL: acme--payments--checkout-api--waf
ACM Certificate: checkout-api.prod.acme.com
```

### DNS Layer
```
Hosted Zone: prod.acme.com
DNS Record: checkout-api.prod.acme.com
Route53 Private Zone: checkout-api.internal.prod.acme.com
CloudFront Alias: checkout-api.prod.acme.com
```

### Storage/Config Layer
```
S3 Object Key: acme/payments/checkout-api/schema.sql
SSM Parameter: /acme/payments/checkout-api/stripe-webhook-secret
Secrets Manager: acme/payments/checkout-api/db-password
```

---

## Tagging Strategy

Apply these tags to **all** resources for cost allocation and resource management:

| Tag Key | Value | Example | Purpose |
|---------|-------|---------|---------|
| `org` | Organization | `acme` | Top-level ownership |
| `domain` | Business domain | `payments` | Capability boundary |
| `service` | Service name | `checkout-api` | Deployment unit |
| `environment` | Deployment stage | `prod` | Optional if account-segregated |
| `owner` | Team/person | `payments-team` | Responsibility tracking |
| `cost-center` | Cost allocation | `payments-team` | Billing attribution |
| `backup-required` | boolean | `true` | Backup policy enforcement |
| `terraform` | boolean | `true` | IaC management indicator |

---

## Implementation Checklist

- [ ] **Define segments:** org, domain, service values
- [ ] **Account strategy:** 1 per environment? (Recommended: yes)
- [ ] **Test naming:** Create 1-2 sample resources in non-prod
- [ ] **Document exceptions:** Route53 reverse hierarchy, DNS patterns
- [ ] **Create IAM policies:** Use path prefixes for least privilege
- [ ] **Enable AWS Config:** Enforce naming patterns automatically
- [ ] **Set up tagging:** Apply tags to non-nameable resources
- [ ] **Create runbooks:** How to find resources by naming convention
- [ ] **Train team:** Share cheatsheet and examples
- [ ] **Monitor drift:** Regular audits for non-compliant names

---

## Format Comparison Examples

| Use Case | Prefix Hierarchy | DNS Reverse | Flat Kebab | Result |
|----------|------------------|-------------|-----------|--------|
| Same service across systems | `/acme/payments/checkout-api` | `checkout-api.payments.acme.com` | `acme-payments-checkout-api` | ✅ All represent same logical resource |
| Different purposes | `/acme/payments/checkout-api/orders` | N/A | `acme-payments-checkout-api-orders` | ✅ Additional scope via suffix |
| Env segregated | `/acme/prod/payments/checkout-api` | `checkout-api.prod.acme.com` | ❌ Not used (handled via account) | ✅ Account provides namespace |
| With partition (logs) | `acme/payments/checkout-api/2024/01/15/logs` | N/A | N/A | ✅ Hierarchical querying in S3 |

---

## One-Liner Reference

**Need to name a resource?** Apply this logic in order:

1. Does it support native hierarchy? → Use it (`/` for paths, `.` for DNS)
2. Is it globally unique (S3, ACM, CloudFront)? → Add `ap-southeast-2--prod--` prefix (literal region and env values)
3. Is it DNS-based? → Use reverse hierarchy: `{service}.prod.acme.com` (env in domain via account)
4. Otherwise → Use format: `{org}-{domain}-{service}-{key}` with `-` delimiters, `--` between segments
5. Tag everything else that doesn't support naming
