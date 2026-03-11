# AAA — Component Build Reference Table
## Adventure Asia App · 4 Environments · ap-southeast-1

**Legend:**
- ✅ Required — create this · ⚡ Optional (optimise later) · ❌ Not needed · `[FILL]` complete after resource is created

---

## 1. NETWORKING

| Component | dev | staging | production | demo | Notes |
|---|---|---|---|---|---|
| **VPC** | ❌ use Default | ❌ use Default | ✅ `aa-vpc-prod` 10.0.0.0/16 | ❌ use Default | Custom VPC required for prod only |
| **Public Subnet 1a** | ❌ | ❌ | ✅ `aa-subnet-public-1a` 10.0.1.0/24 · ap-southeast-1a | ❌ | |
| **Public Subnet 1b** | ❌ | ❌ | ✅ `aa-subnet-public-1b` 10.0.2.0/24 · ap-southeast-1b | ❌ | ALB + Multi-AZ require 2 AZs |
| **Private Subnet 1a** | ❌ | ❌ | ✅ `aa-subnet-private-1a` 10.0.3.0/24 · ap-southeast-1a | ❌ | RDS + Redis — no internet access |
| **Private Subnet 1b** | ❌ | ❌ | ✅ `aa-subnet-private-1b` 10.0.4.0/24 · ap-southeast-1b | ❌ | RDS Multi-AZ standby |
| **Internet Gateway** | ❌ | ❌ | ✅ `aa-igw` · attach to aa-vpc-prod | ❌ | |
| **NAT Gateway** | ❌ | ❌ | ✅ `aa-nat-1a` · in public-1a · Elastic IP | ❌ | ~$35/month · outbound from private subnets |
| **Route Table — Public** | ❌ | ❌ | ✅ `aa-rt-public` · 0.0.0.0/0 → aa-igw | ❌ | |
| **Route Table — Private** | ❌ | ❌ | ✅ `aa-rt-private` · 0.0.0.0/0 → aa-nat-1a | ❌ | |
| **SG: aa-sg-alb** | ❌ | ✅ shared | ✅ `aa-sg-alb` · IN: 443+80 / 0.0.0.0/0 | ❌ | Shared between staging + prod |
| **SG: aa-sg-app** | ✅ `aa-sg-app` · IN: 443+80+22 / MY-IP | ✅ shared | ✅ shared · IN: 80 from aa-sg-alb · 22 / MY-IP | ❌ | Dev: add port 443/80 from internet |
| **SG: aa-sg-db** | ✅ shared | ✅ shared | ✅ `aa-sg-db` · IN: 5432 from aa-sg-app | ❌ | NEVER use 0.0.0.0/0 as source |
| **SG: aa-sg-redis** | ✅ shared | ✅ shared | ✅ `aa-sg-redis` · IN: 6379 from aa-sg-app | ❌ | |
| **SSL Certificate (ACM)** | ❌ | ✅ shared | ✅ `*.adventureasia.app` + `adventureasia.app` | ❌ | Wildcard shared across all environments |
| **Application Load Balancer** | ❌ | ✅ `aa-alb-stg` | ✅ `aa-alb-prod` · HTTPS 443 → aa-tg-prod | ❌ | HTTP 80 redirects to HTTPS |
| **Target Group** | ❌ | ✅ `aa-tg-stg` · /health · port 80 | ✅ `aa-tg-prod` · /health · port 80 | ❌ | Healthy threshold: 2 · Interval: 30s |
| **Route 53 Hosted Zone** | ❌ | ✅ shared | ✅ `adventureasia.app` | ❌ | 1 hosted zone for all environments |
| **DNS Record — API** | ❌ | ✅ `api.stg` → aa-alb-stg | ✅ `api` → aa-alb-prod | ❌ | A record, Alias = Yes |
| **DNS Record — Admin** | ❌ | ✅ `admin.stg` → CloudFront stg | ✅ `admin` → CloudFront prod | ❌ | |
| **DNS Record — Web** | ❌ | ❌ | ✅ `www` + `@` → CloudFront prod | ❌ | |
| **DNS Record — Demo** | ❌ | ❌ | ❌ | ✅ `demo` → EC2 demo public IP | |

---

## 2. COMPUTE — EC2

| Component | dev | staging | production | demo | Notes |
|---|---|---|---|---|---|
| **EC2 Instance** | ✅ `aa-dev-api` | ✅ `aa-stg-api` | ✅ `aa-prod-api` | ✅ `aa-demo-api` | |
| **AMI** | Amazon Linux 2023 | Amazon Linux 2023 | Amazon Linux 2023 | Amazon Linux 2023 | |
| **Instance Type** | t3.micro (1 vCPU, 1 GB) | t3.small (2 vCPU, 2 GB) | **t3.medium (2 vCPU, 4 GB)** | t3.micro (1 vCPU, 1 GB) | Upgrade to t3.large if needed |
| **Root Volume** | 20 GB gp3 | 30 GB gp3 | **50 GB gp3** | 20 GB gp3 | |
| **Subnet** | Public (Default VPC) | Public 1a | Public 1a (Custom VPC) | Public (Default VPC) | |
| **Auto-assign Public IP** | ✅ Yes | ✅ Yes (direct IP access) | ❌ No (accessed via ALB) | ✅ Yes | |
| **IAM Role** | `aa-role-ec2-dev` | `aa-role-ec2-stg` | **`aa-role-ec2-prod`** | `aa-role-ec2-demo` | One dedicated role per environment |
| **IAM Role Policies** | S3(dev) + CW + SSM + SM(dev) | S3(stg) + CW + SSM + SM(stg) | S3(prod) + CW + SSM + SM(prod) | S3(demo) + CW + SSM | SM = Secrets Manager |
| **Key Pair** | `aa-ec2-keypair` | `aa-ec2-keypair` | `aa-ec2-keypair` | `aa-ec2-keypair` | 1 key pair shared across all environments |
| **Security Group** | aa-sg-app | aa-sg-app | aa-sg-app | aa-sg-app | |
| **Elastic IP** | ❌ | ❌ | ❌ (use ALB) | ⚡ Optional | |
| **SSM Session Manager** | ✅ | ✅ | ✅ (SSH not required) | ✅ | Requires AmazonSSMManagedInstanceCore in IAM role |
| **SSH port 22** | ✅ MY-IP only | ✅ MY-IP only | ⚡ Recommended off · use SSM | ✅ MY-IP only | |
| **Nginx** | ✅ reverse proxy → port 3000 | ✅ | ✅ | ✅ | |
| **PM2 (Node.js)** | ✅ | ✅ | ✅ | ✅ | Or PHP-FPM if using Laravel |
| **CloudWatch Agent** | ⚡ Optional | ✅ | ✅ | ❌ | Log groups: /aa/[env]/app, /aa/[env]/nginx |
| **User Data (bootstrap)** | ✅ install Node+PM2+Nginx+git | ✅ | ✅ | ✅ | Runs once at instance launch |

---

## 3. COMPUTE — STATIC FRONTEND (S3 + CloudFront)

| Component | dev | staging | production | demo | Notes |
|---|---|---|---|---|---|
| **S3 Bucket — Admin Dashboard** | `aa-dev-admin-static` | `aa-stg-admin-static` | `aa-prod-admin-static` | ❌ | React build output |
| **S3 Bucket — Customer Website** | ❌ | `aa-stg-web-static` | `aa-prod-web-static` | ❌ | |
| **Static Website Hosting** | ✅ | ✅ | ❌ use CloudFront | ❌ | Dev/stg: S3 website endpoint is sufficient |
| **Block Public Access** | ✅ ON | ✅ ON | ✅ ON | — | CloudFront OAC replaces direct public access |
| **CloudFront Distribution — Admin** | ❌ | ⚡ Optional | ✅ `aa-cf-admin` | ❌ | Origin: aa-prod-admin-static |
| **CloudFront Distribution — Web** | ❌ | ⚡ Optional | ✅ `aa-cf-web` | ❌ | Origin: aa-prod-web-static |
| **CloudFront Distribution — Media** | ❌ | ❌ | ✅ `aa-cf-media` | ❌ | Origin: aa-prod-media/images/ |
| **Origin Access Control (OAC)** | ❌ | ❌ | ✅ (1 per CloudFront distribution) | ❌ | Replaces legacy OAI |
| **Cache Policy** | — | — | CachingOptimized | — | |
| **Alternate Domain (CNAME)** | — | — | admin.adventureasia.app | — | |

---

## 4. DATABASE — RDS PostgreSQL

| Component | dev | staging | production | demo | Notes |
|---|---|---|---|---|---|
| **RDS Instance** | ✅ `aa-dev-db` | ✅ `aa-stg-db` | ✅ `aa-prod-db` | ✅ `aa-demo-db` | |
| **Engine** | PostgreSQL 15.x | PostgreSQL 15.x | PostgreSQL 15.x | PostgreSQL 15.x | Must match dev environment version |
| **Instance Class** | db.t3.micro (1 vCPU, 1 GB) | db.t3.micro (1 vCPU, 1 GB) | **db.t3.small (2 vCPU, 2 GB)** | db.t3.micro | Upgrade if connections are consistently high |
| **Storage** | 20 GB gp3 | 20 GB gp3 | **50 GB gp3 · autoscale → 200 GB** | 20 GB gp3 | |
| **Multi-AZ** | ❌ No | ❌ No | ✅ **Yes — mandatory** | ❌ No | Prod: automatic standby failover |
| **DB Subnet Group** | Default subnet group | Default subnet group | ✅ `aa-db-subnet-group` (private subnets) | Default | |
| **Public Accessibility** | ❌ **No** | ❌ **No** | ❌ **No** | ❌ **No** | NEVER set to Yes — any environment |
| **Security Group** | aa-sg-db | aa-sg-db | aa-sg-db | aa-sg-db | |
| **Initial Database Name** | aa_development | aa_staging | aa_production | aa_demo | |
| **Master Username** | aa_admin | aa_admin | aa_admin | aa_admin | For migrations only — never used by the app |
| **App Username** | aa_app_user | aa_app_user | aa_app_user | aa_app_user | SELECT / INSERT / UPDATE / DELETE only |
| **Credentials Storage** | Secrets Manager `aa/dev/db` | `aa/stg/db` | **`aa/prod/db`** | `aa/demo/db` | |
| **Automated Backups** | ✅ 7 days | ✅ 7 days | ✅ **30 days** | ✅ 3 days | |
| **Backup Window** | 18:00–20:00 UTC | 18:00–20:00 UTC | 18:00–20:00 UTC | any | = 01:00–03:00 Vietnam time |
| **Deletion Protection** | ❌ No | ❌ No | ✅ **Yes** | ❌ No | |
| **Performance Insights** | ❌ | ⚡ Optional | ✅ 7-day retention | ❌ | |
| **Enhanced Monitoring** | ❌ | ❌ | ✅ 60-second interval | ❌ | |
| **max_connections (estimate)** | db.t3.micro ~87 | db.t3.micro ~87 | db.t3.small ~172 | db.t3.micro ~87 | Use PgBouncer if >172 connections needed |

---

## 5. CACHE — ElastiCache Redis

| Component | dev | staging | production | demo | Notes |
|---|---|---|---|---|---|
| **Redis Cluster** | ✅ `aa-dev-redis` | ✅ `aa-stg-redis` | ✅ `aa-prod-redis` | ❌ (share dev Redis or skip) | |
| **Engine** | Redis 7.x | Redis 7.x | Redis 7.x | — | |
| **Node Type** | cache.t3.micro (0.5 GB) | cache.t3.micro (0.5 GB) | **cache.t3.micro (0.5 GB)** | — | Upgrade to cache.t3.small if OOM errors occur |
| **Cluster Mode** | Disabled (single node) | Disabled | Disabled | — | Enable for HA if required |
| **Port** | 6379 | 6379 | 6379 | — | |
| **Subnet Group** | Default | Default | ✅ `aa-redis-subnet-group` (private) | — | |
| **Security Group** | aa-sg-redis | aa-sg-redis | aa-sg-redis | — | |
| **Backup** | ❌ | ❌ | ✅ Daily snapshot · 7 days | — | |
| **Endpoint** | `aa-dev-redis.[FILL].cache.amazonaws.com` | `aa-stg-redis.[FILL]` | `aa-prod-redis.[FILL]` | — | |
| **Used for** | OTP + session cache | OTP + session + queue | OTP + session + cache + queue | — | Redis down = OTP login fully broken (single point of failure) |

---

## 6. STORAGE — S3 (Media & Files)

| Component | dev | staging | production | demo | Notes |
|---|---|---|---|---|---|
| **Bucket Name** | `aa-dev-media` | `aa-stg-media` | `aa-prod-media` | `aa-demo-media` | Must be globally unique |
| **Block Public Access** | ✅ All 4 ON | ✅ All 4 ON | ✅ All 4 ON | ✅ All 4 ON | |
| **Versioning** | ⚡ Optional | ⚡ Optional | ✅ **Enabled** | ❌ | Prod: allows restore if file is accidentally overwritten |
| **Object Ownership** | ACLs disabled | ACLs disabled | ACLs disabled | ACLs disabled | AWS recommended setting |
| **Encryption** | SSE-S3 (default) | SSE-S3 | SSE-S3 | SSE-S3 | |
| **Folder: uploads/** | ✅ | ✅ | ✅ | ✅ | Booking documents, passport scans — presigned URL access only |
| **Folder: images/** | ✅ | ✅ | ✅ | ✅ | Tour photos — delivered via CloudFront |
| **Folder: documents/** | ✅ | ✅ | ✅ | ✅ | PDF brochures, itineraries — presigned URL access |
| **Folder: exports/** | ✅ | ✅ | ✅ | ✅ | Receipts, system reports — auto-deleted by lifecycle rule |
| **Lifecycle: exports/** | ❌ | ❌ | ✅ Expire after 90 days | ❌ | |
| **Lifecycle: noncurrent versions** | ❌ | ❌ | ✅ Delete after 30 days | ❌ | Keeps versioning active while controlling storage cost |
| **Bucket Policy** | ❌ | ❌ | ✅ Allow CloudFront OAC → images/* | ❌ | |
| **Access Pattern** | EC2 via IAM role | EC2 via IAM role | EC2 via IAM role + CloudFront | EC2 via IAM role | Direct public S3 URLs must never be used |
| **Presigned URL Expiry** | 60 minutes | 60 minutes | **60 minutes** | 60 minutes | Adjust per file type as needed |

---

## 7. SECRETS MANAGER

| Secret Name | dev | staging | production | demo | Contents |
|---|---|---|---|---|---|
| `aa/dev/db` | ✅ | ❌ | ❌ | ❌ | host, port, dbname, username, password |
| `aa/stg/db` | ❌ | ✅ | ❌ | ❌ | host, port, dbname, username, password |
| `aa/prod/db` | ❌ | ❌ | ✅ | ❌ | host, port, dbname, username, password |
| `aa/demo/db` | ❌ | ❌ | ❌ | ✅ | host, port, dbname, username, password |
| `aa/dev/jwt` | ✅ | ❌ | ❌ | ❌ | secret_key (64-char random string) |
| `aa/stg/jwt` | ❌ | ✅ | ❌ | ❌ | secret_key (64-char, different from prod) |
| `aa/prod/jwt` | ❌ | ❌ | ✅ | ❌ | secret_key (64-char random string) |
| `aa/prod/payment_api` | ❌ | ❌ | ✅ | ❌ | api_key, webhook_secret |
| `aa/stg/payment_api` | ❌ | ✅ | ❌ | ❌ | Test keys from payment provider |
| `aa/prod/maps_api` | ❌ | ❌ | ✅ | ❌ | api_key |
| `aa/prod/email_smtp` | ❌ | ❌ | ✅ | ❌ | host, port, username, password |
| `aa/stg/email_smtp` | ❌ | ✅ | ❌ | ❌ | |
| **Automatic Rotation** | ❌ | ❌ | ✅ (after launch) | ❌ | Lambda rotation function |
| **Encryption** | aws/secretsmanager | aws/secretsmanager | aws/secretsmanager | aws/secretsmanager | Default managed key |

---

## 8. IAM — ROLES & POLICIES

| Component | dev | staging | production | demo | Notes |
|---|---|---|---|---|---|
| **EC2 Role** | `aa-role-ec2-dev` | `aa-role-ec2-stg` | `aa-role-ec2-prod` | `aa-role-ec2-demo` | 1 dedicated role per environment |
| **Policy: S3 Access** | ✅ aa-dev-media only | ✅ aa-stg-media only | ✅ aa-prod-media only | ✅ aa-demo-media only | Inline policy — scoped to single bucket |
| **Policy: CloudWatch Agent** | ✅ CloudWatchAgentServerPolicy | ✅ | ✅ | ✅ | AWS managed policy |
| **Policy: SSM Session Manager** | ✅ AmazonSSMManagedInstanceCore | ✅ | ✅ | ✅ | Replaces SSH access |
| **Policy: Secrets Manager** | ✅ aa/dev/* only | ✅ aa/stg/* only | ✅ aa/prod/* only | ✅ aa/demo/* only | Inline policy — scoped per environment |
| **GitHub Actions Role / User** | ❌ | ✅ deploy access stg | ✅ deploy access prod | ❌ | Confirm OIDC vs static keys with Shivam |
| **IAM Group: aa-devops** | ✅ shared | ✅ shared | ✅ shared | ✅ shared | admin_nghiep is a member |
| **Group Policies** | EC2+RDS+S3+CW ReadOnly | same | same | same | ReadOnly for monitoring and audit |
| **Root MFA** | N/A | N/A | ✅ **Mandatory** | N/A | Verify immediately — highest priority |
| **IAM User MFA** | ✅ admin_nghiep-mfa | ✅ | ✅ | ✅ | |

---

## 9. MONITORING — CloudWatch

| Component | dev | staging | production | demo | Notes |
|---|---|---|---|---|---|
| **SNS Topic** | ❌ | ✅ `aa-stg-alerts` | ✅ `aa-prod-alerts` | ❌ | Email subscription must be confirmed |
| **Alarm: EC2 CPU > 70%** | ❌ | ✅ | ✅ `aa-prod-ec2-cpu-high` | ❌ | 3/3 × 5 min = 15 consecutive minutes |
| **Alarm: EC2 Status Check** | ❌ | ✅ | ✅ `aa-prod-ec2-status` | ❌ | 2/2 × 1 min |
| **Alarm: RDS Connections > 70** | ❌ | ✅ | ✅ `aa-prod-rds-connections` | ❌ | db.t3.small max ~172 connections |
| **Alarm: RDS Storage < 5 GB** | ❌ | ✅ | ✅ `aa-prod-rds-storage` | ❌ | In bytes: 5368709120 |
| **Alarm: ALB 5xx Count > 10** | ❌ | ✅ | ✅ `aa-prod-alb-5xx` | ❌ | 2/2 × 5 min |
| **Alarm: Redis Memory < 100 MB** | ❌ | ⚡ | ✅ `aa-prod-redis-memory` | ❌ | In bytes: 104857600 |
| **Alarm: Billing > $240** | ❌ | ❌ | ✅ `aa-billing-prod` | ❌ | = 80% of $300 monthly budget |
| **Log Group: Nginx Access** | ❌ | ✅ `/aa/stg/nginx-access` | ✅ `/aa/prod/nginx-access` | ❌ | Retention: 30 days |
| **Log Group: Nginx Error** | ❌ | ✅ `/aa/stg/nginx-error` | ✅ `/aa/prod/nginx-error` | ❌ | Retention: 30 days |
| **Log Group: App** | ❌ | ✅ `/aa/stg/app` | ✅ `/aa/prod/app` | ❌ | Retention: 30 days |
| **CloudWatch Dashboard** | ❌ | ⚡ Optional | ✅ `AdventureAsia-Production` | ❌ | 7 widgets |
| **CloudWatch Agent on EC2** | ❌ | ✅ | ✅ | ❌ | Captures memory + disk — not in default EC2 metrics |

---

## 10. CI/CD — GitHub Actions

| Component | dev | staging | production | demo | Notes |
|---|---|---|---|---|---|
| **GitHub Environment** | ❌ | ✅ `staging` · no protection rules | ✅ `production` · required reviewers | ❌ | |
| **Required Reviewer (prod)** | — | — | ✅ Shivam + Nghiep | — | |
| **Workflow File** | ✅ `.github/workflows/deploy.yml` | ✅ Job: deploy-staging | ✅ Job: deploy-production | ❌ | |
| **Trigger: Staging** | — | `push to develop` (automatic) | — | — | Only after build + test pass |
| **Trigger: Production** | — | — | Manual approval gate | — | Only after staging deploy |
| **Deploy Method** | git pull directly | SSM Send-Command | SSM Send-Command | Manual | No SSH required |
| **Health Check after deploy** | `curl /health` | ✅ `curl stg.../health` | ✅ `curl api.../health` | ❌ | Failure triggers rollback |
| **Rollback** | git checkout | Redeploy previous commit | Redeploy previous commit | Manual | DB migrations require separate rollback plan |
| **GitHub Secrets** | — | STG_EC2_INSTANCE_ID | PROD_EC2_INSTANCE_ID | — | + AWS credentials |
| **Branch Strategy** | `feature/*` | auto from `develop` | manual from `develop` | manual | |

---

## 11. DNS — Route 53

| Record | Type | Value | Environment |
|---|---|---|---|
| `adventureasia.app` | A Alias | → aa-alb-prod | production |
| `www.adventureasia.app` | A Alias | → aa-alb-prod | production |
| `api.adventureasia.app` | A Alias | → aa-alb-prod | production |
| `admin.adventureasia.app` | A Alias | → aa-cf-admin | production |
| `assets.adventureasia.app` | A Alias | → aa-cf-media | production |
| `api.stg.adventureasia.app` | A Alias | → aa-alb-stg | staging |
| `admin.stg.adventureasia.app` | A Alias | → aa-cf-admin-stg | staging |
| `demo.adventureasia.app` | A | → aa-demo-api public IP | demo |
| `_acme-challenge.*` | CNAME | → ACM validation values | all |

---

## 12. COST CONTROLS

| Component | dev | staging | production | demo | Notes |
|---|---|---|---|---|---|
| **AWS Budget** | ⚡ `aa-dev-monthly` $20 | ✅ `aa-stg-monthly` $50 | ✅ `aa-prod-monthly` $300 | ❌ | |
| **Alert Thresholds** | 80% | 70% + 100% | 70% + 100% | — | SNS email notification |
| **Resource Tagging** | ✅ | ✅ | ✅ | ✅ | Tags: Project + Env + Owner + Component |
| **Cost Allocation Tags** | ✅ activate | ✅ | ✅ | ✅ | View per-environment breakdown in Cost Explorer |
| **Stop when not in use** | ✅ Stop EC2 + RDS | ✅ Stop outside business hours | ❌ 24/7 | ✅ Stop when not demoing | |
| **Reserved Instances** | ❌ | ❌ | ⚡ After 3 months of stable usage | ❌ | Reduces EC2 + RDS cost by ~35% |

---

## 13. ESTIMATED MONTHLY COST (On-Demand)

| Service | dev | staging | production | demo | Total / month |
|---|---|---|---|---|---|
| EC2 | t3.micro ~$9 | t3.small ~$17 (stop nights/weekends → ~$9) | t3.medium ~$34 | t3.micro ~$9 (on demand only) | ~$55–70 |
| RDS | db.t3.micro ~$18 | db.t3.micro ~$18 | db.t3.small Multi-AZ ~$55 | db.t3.micro ~$18 | ~$109 |
| ElastiCache | cache.t3.micro ~$14 | cache.t3.micro ~$14 | cache.t3.micro ~$14 | ❌ | ~$42 |
| S3 | ~$1 | ~$1 | ~$2 | ~$1 | ~$5 |
| ALB | ❌ | ~$18 | ~$18 | ❌ | ~$36 |
| NAT Gateway | ❌ | ❌ | ~$35 | ❌ | ~$35 |
| CloudFront | ❌ | ❌ | ~$5 | ❌ | ~$5 |
| Route 53 | — | — | ~$1 | — | ~$1 |
| ACM | FREE | FREE | FREE | FREE | $0 |
| Secrets Manager | ~$1 | ~$1 | ~$2 | ~$1 | ~$5 |
| CloudWatch | ❌ | ~$5 | ~$10 | ❌ | ~$15 |
| Data Transfer | ~$1 | ~$2 | ~$10 | ~$1 | ~$14 |
| **TOTAL** | **~$44** | **~$76** | **~$186** | **~$30** | **~$336 / month** |

> 💡 **Quick wins:** Stop dev + stg EC2 + RDS outside business hours → saves ~$40/month immediately. After 3 months of stable usage: apply Reserved Instances for prod EC2 + RDS → saves an additional ~$40/month.

---

## 14. BUILD ORDER CHECKLIST

```
PHASE 1 — FOUNDATION
  □ 1.  IAM: Confirm Root MFA
  □ 2.  IAM: Create Roles (aa-role-ec2-dev/stg/prod) + inline policies
  □ 3.  VPC: aa-vpc-prod + 4 subnets + IGW + NAT + route tables
  □ 4.  Security Groups: aa-sg-alb, aa-sg-app, aa-sg-db, aa-sg-redis
  □ 5.  ACM: Request *.adventureasia.app certificate → validate via Route 53

PHASE 2 — DATA LAYER
  □ 6.  Secrets Manager: create all secrets (fill in host values after RDS is created)
  □ 7.  S3: aa-dev/stg/prod/demo-media + folder structure + lifecycle rules
  □ 8.  RDS Subnet Group: aa-db-subnet-group (private subnets)
  □ 9.  RDS: aa-stg-db first → aa-prod-db second (Multi-AZ, deletion protection)
  □ 10. RDS: Create aa_app_user + update endpoint in Secrets Manager
  □ 11. ElastiCache Subnet Group: aa-redis-subnet-group
  □ 12. ElastiCache: aa-stg-redis → aa-prod-redis

PHASE 3 — COMPUTE
  □ 13. Key Pair: aa-ec2-keypair (download and store securely)
  □ 14. EC2: aa-stg-api (test on staging first)
  □ 15. EC2: Configure Nginx + PM2 + CloudWatch Agent on staging
  □ 16. EC2: Deploy application to staging, verify /health endpoint responds
  □ 17. EC2: aa-prod-api (only after staging is stable)
  □ 18. Target Groups: aa-tg-stg + aa-tg-prod (register instances)
  □ 19. ALB: aa-alb-stg + aa-alb-prod (HTTPS + HTTP redirect)
  □ 20. Route 53: DNS records for all subdomains
  □ 21. CloudFront: distributions for static assets

PHASE 4 — OBSERVABILITY
  □ 22. SNS: aa-stg-alerts + aa-prod-alerts (confirm email subscription)
  □ 23. CloudWatch Alarms: 6 alarms for production
  □ 24. CloudWatch Dashboard: AdventureAsia-Production

PHASE 5 — CI/CD
  □ 25. GitHub Environments: staging (auto) + production (manual gate)
  □ 26. GitHub Secrets: all 7 required secrets
  □ 27. Workflow file: .github/workflows/deploy.yml
  □ 28. Test full pipeline: push → build → staging → health check

PHASE 6 — COST CONTROLS
  □ 29. AWS Budgets: 3 budgets with alert thresholds
  □ 30. Cost Allocation Tags: activate in Billing console
```

---

*Adventure Asia App — Component Build Reference Table | 4 Environments | Nghiep | DevOps Mentorship*
*Based on: PRD Technical Spec · Mentor Architecture Plan (Shivam) · ap-southeast-1*
