# AAA — Bảng Tổng Hợp Thành Phần Cần Tạo
## Adventure Asia App · 4 Environments · ap-southeast-1

**Quy tắc đọc bảng:**
- ✅ Cần tạo · ⚡ Tùy chọn (tối ưu sau) · ❌ Không tạo · `[FILL]` điền sau khi tạo xong

---

## 1. NETWORKING

| Component | dev | staging | production | demo | Ghi chú |
|---|---|---|---|---|---|
| **VPC** | ❌ dùng Default | ❌ dùng Default | ✅ `aa-vpc-prod` 10.0.0.0/16 | ❌ dùng Default | Custom VPC chỉ cần cho prod |
| **Public Subnet 1a** | ❌ | ❌ | ✅ `aa-subnet-public-1a` 10.0.1.0/24 · ap-southeast-1a | ❌ | |
| **Public Subnet 1b** | ❌ | ❌ | ✅ `aa-subnet-public-1b` 10.0.2.0/24 · ap-southeast-1b | ❌ | ALB + Multi-AZ cần 2 AZ |
| **Private Subnet 1a** | ❌ | ❌ | ✅ `aa-subnet-private-1a` 10.0.3.0/24 · ap-southeast-1a | ❌ | RDS + Redis không có internet |
| **Private Subnet 1b** | ❌ | ❌ | ✅ `aa-subnet-private-1b` 10.0.4.0/24 · ap-southeast-1b | ❌ | RDS Multi-AZ standby |
| **Internet Gateway** | ❌ | ❌ | ✅ `aa-igw` · attach aa-vpc-prod | ❌ | |
| **NAT Gateway** | ❌ | ❌ | ✅ `aa-nat-1a` · in public-1a · Elastic IP | ❌ | ~$35/month · private subnet outbound |
| **Route Table Public** | ❌ | ❌ | ✅ `aa-rt-public` · 0.0.0.0/0 → aa-igw | ❌ | |
| **Route Table Private** | ❌ | ❌ | ✅ `aa-rt-private` · 0.0.0.0/0 → aa-nat-1a | ❌ | |
| **SG: aa-sg-alb** | ❌ | ✅ share | ✅ `aa-sg-alb` · IN: 443+80/0.0.0.0/0 | ❌ | Dùng chung staging + prod |
| **SG: aa-sg-app** | ✅ `aa-sg-app` · IN: 443+80+22/MY-IP | ✅ share | ✅ share · IN: 80 from aa-sg-alb · 22/MY-IP | ❌ | Dev: thêm port 443/80 từ internet |
| **SG: aa-sg-db** | ✅ share | ✅ share | ✅ `aa-sg-db` · IN: 5432 from aa-sg-app | ❌ | KHÔNG BAO GIỜ 0.0.0.0/0 |
| **SG: aa-sg-redis** | ✅ share | ✅ share | ✅ `aa-sg-redis` · IN: 6379 from aa-sg-app | ❌ | |
| **SSL Certificate (ACM)** | ❌ | ✅ share | ✅ `*.adventureasia.app` + `adventureasia.app` | ❌ | Wildcard dùng chung tất cả envs |
| **Application Load Balancer** | ❌ | ✅ `aa-alb-stg` | ✅ `aa-alb-prod` · HTTPS 443 → aa-tg-prod | ❌ | HTTP 80 redirect → HTTPS |
| **Target Group** | ❌ | ✅ `aa-tg-stg` · /health · port 80 | ✅ `aa-tg-prod` · /health · port 80 | ❌ | Healthy threshold: 2 · Interval: 30s |
| **Route 53 Hosted Zone** | ❌ | ✅ share | ✅ `adventureasia.app` | ❌ | 1 hosted zone cho tất cả |
| **DNS Record — API** | ❌ | ✅ `api.stg` → aa-alb-stg | ✅ `api` → aa-alb-prod | ❌ | A record, Alias = Yes |
| **DNS Record — Admin** | ❌ | ✅ `admin.stg` → CloudFront stg | ✅ `admin` → CloudFront prod | ❌ | |
| **DNS Record — Web** | ❌ | ❌ | ✅ `www` + `@` → CloudFront prod | ❌ | |
| **DNS Record — Demo** | ❌ | ❌ | ❌ | ✅ `demo` → EC2 demo IP | |

---

## 2. COMPUTE — EC2

| Component | dev | staging | production | demo | Ghi chú |
|---|---|---|---|---|---|
| **EC2 Instance** | ✅ `aa-dev-api` | ✅ `aa-stg-api` | ✅ `aa-prod-api` | ✅ `aa-demo-api` | |
| **AMI** | Amazon Linux 2023 | Amazon Linux 2023 | Amazon Linux 2023 | Amazon Linux 2023 | |
| **Instance Type** | t3.micro (1 vCPU, 1GB) | t3.small (2 vCPU, 2GB) | **t3.medium (2 vCPU, 4GB)** | t3.micro (1 vCPU, 1GB) | Prod upgrade t3.large nếu cần |
| **Root Volume** | 20 GB gp3 | 30 GB gp3 | **50 GB gp3** | 20 GB gp3 | |
| **Subnet** | Public (Default VPC) | Public 1a | Public 1a (Custom VPC) | Public (Default VPC) | |
| **Auto-assign Public IP** | ✅ Yes | ✅ Yes (dùng IP trực tiếp) | ❌ No (dùng qua ALB) | ✅ Yes | |
| **IAM Role** | `aa-role-ec2-dev` | `aa-role-ec2-stg` | **`aa-role-ec2-prod`** | `aa-role-ec2-demo` | Mỗi env có role riêng |
| **IAM Role Policies** | S3(dev) + CW + SSM + SM(dev) | S3(stg) + CW + SSM + SM(stg) | S3(prod) + CW + SSM + SM(prod) | S3(demo) + CW + SSM | SM = Secrets Manager |
| **Key Pair** | `aa-ec2-keypair` | `aa-ec2-keypair` | `aa-ec2-keypair` | `aa-ec2-keypair` | 1 key pair cho tất cả |
| **Security Group** | aa-sg-app | aa-sg-app | aa-sg-app | aa-sg-app | |
| **Elastic IP** | ❌ | ❌ | ❌ (dùng ALB) | ⚡ Tuỳ | |
| **SSM Session Manager** | ✅ | ✅ | ✅ (SSH không cần thiết) | ✅ | Cần IAM role có AmazonSSMManagedInstanceCore |
| **SSH port 22** | ✅ MY-IP only | ✅ MY-IP only | ⚡ Nên tắt · dùng SSM | ✅ MY-IP only | |
| **Nginx** | ✅ reverse proxy → port 3000 | ✅ | ✅ | ✅ | |
| **PM2 (Node.js)** | ✅ | ✅ | ✅ | ✅ | Hoặc PHP-FPM nếu Laravel |
| **CloudWatch Agent** | ⚡ Optional | ✅ | ✅ | ❌ | Log groups: /aa/[env]/app, /aa/[env]/nginx |
| **User Data (bootstrap)** | ✅ install Node+PM2+Nginx+git | ✅ | ✅ | ✅ | Chạy 1 lần lúc launch |

---

## 3. COMPUTE — STATIC FRONTEND (S3 + CloudFront)

| Component | dev | staging | production | demo | Ghi chú |
|---|---|---|---|---|---|
| **S3 Bucket — Admin Dashboard** | `aa-dev-admin-static` | `aa-stg-admin-static` | `aa-prod-admin-static` | ❌ | React build output |
| **S3 Bucket — Customer Website** | ❌ | `aa-stg-web-static` | `aa-prod-web-static` | ❌ | |
| **Static Website Hosting** | ✅ | ✅ | ❌ dùng CloudFront | ❌ | Dev/stg: có thể dùng S3 website trực tiếp |
| **Block Public Access** | ✅ ON | ✅ ON | ✅ ON | — | CloudFront OAC thay thế public access |
| **CloudFront Distribution — Admin** | ❌ | ⚡ Optional | ✅ `aa-cf-admin` | ❌ | Origin: aa-prod-admin-static |
| **CloudFront Distribution — Web** | ❌ | ⚡ Optional | ✅ `aa-cf-web` | ❌ | Origin: aa-prod-web-static |
| **CloudFront Distribution — Media** | ❌ | ❌ | ✅ `aa-cf-media` | ❌ | Origin: aa-prod-media/images/ |
| **Origin Access Control (OAC)** | ❌ | ❌ | ✅ (1 per CloudFront dist) | ❌ | Thay thế OAI |
| **Cache Policy** | — | — | CachingOptimized | — | |
| **Alternate Domain (CNAME)** | — | — | admin.adventureasia.app | — | |

---

## 4. DATABASE — RDS PostgreSQL

| Component | dev | staging | production | demo | Ghi chú |
|---|---|---|---|---|---|
| **RDS Instance** | ✅ `aa-dev-db` | ✅ `aa-stg-db` | ✅ `aa-prod-db` | ✅ `aa-demo-db` | |
| **Engine** | PostgreSQL 15.x | PostgreSQL 15.x | PostgreSQL 15.x | PostgreSQL 15.x | Match dev environment |
| **Instance Class** | db.t3.micro (1 vCPU, 1GB) | db.t3.micro (1 vCPU, 1GB) | **db.t3.small (2 vCPU, 2GB)** | db.t3.micro | Upgrade nếu connections đầy |
| **Storage** | 20 GB gp3 | 20 GB gp3 | **50 GB gp3 · autoscale → 200GB** | 20 GB gp3 | |
| **Multi-AZ** | ❌ No | ❌ No | ✅ **Yes — mandatory** | ❌ No | Prod: standby tự động failover |
| **DB Subnet Group** | Default subnet group | Default subnet group | ✅ `aa-db-subnet-group` (private subnets) | Default | |
| **Public Accessibility** | ❌ **No** | ❌ **No** | ❌ **No** | ❌ **No** | KHÔNG BAO GIỜ Yes |
| **Security Group** | aa-sg-db | aa-sg-db | aa-sg-db | aa-sg-db | |
| **Initial Database Name** | aa_development | aa_staging | aa_production | aa_demo | |
| **Master Username** | aa_admin | aa_admin | aa_admin | aa_admin | Chỉ dùng cho migrations |
| **App Username** | aa_app_user | aa_app_user | aa_app_user | aa_app_user | SELECT/INSERT/UPDATE/DELETE only |
| **Credentials Storage** | Secrets Manager `aa/dev/db` | `aa/stg/db` | **`aa/prod/db`** | `aa/demo/db` | |
| **Automated Backups** | ✅ 7 days | ✅ 7 days | ✅ **30 days** | ✅ 3 days | |
| **Backup Window** | 18:00-20:00 UTC | 18:00-20:00 UTC | 18:00-20:00 UTC | any | = 01:00-03:00 giờ VN |
| **Deletion Protection** | ❌ No | ❌ No | ✅ **Yes** | ❌ No | |
| **Performance Insights** | ❌ | ⚡ Optional | ✅ 7-day retention | ❌ | |
| **Enhanced Monitoring** | ❌ | ❌ | ✅ 60-second interval | ❌ | |
| **max_connections (estimate)** | db.t3.micro ~87 | db.t3.micro ~87 | db.t3.small ~172 | db.t3.micro ~87 | PgBouncer nếu cần >172 |

---

## 5. CACHE — ElastiCache Redis

| Component | dev | staging | production | demo | Ghi chú |
|---|---|---|---|---|---|
| **Redis Cluster** | ✅ `aa-dev-redis` | ✅ `aa-stg-redis` | ✅ `aa-prod-redis` | ❌ (dùng dev redis hoặc không cần) | |
| **Engine** | Redis 7.x | Redis 7.x | Redis 7.x | — | |
| **Node Type** | cache.t3.micro (0.5GB) | cache.t3.micro (0.5GB) | **cache.t3.micro (0.5GB)** | — | Upgrade nếu OOM errors |
| **Cluster Mode** | Disabled (single node) | Disabled | Disabled | — | Enable nếu cần HA |
| **Port** | 6379 | 6379 | 6379 | — | |
| **Subnet Group** | Default | Default | ✅ `aa-redis-subnet-group` (private) | — | |
| **Security Group** | aa-sg-redis | aa-sg-redis | aa-sg-redis | — | |
| **Backup** | ❌ | ❌ | ✅ Daily snapshot · 7 days | — | |
| **Endpoint** | `aa-dev-redis.[FILL].cache.amazonaws.com` | `aa-stg-redis.[FILL]` | `aa-prod-redis.[FILL]` | — | |
| **Redis dùng cho** | OTP + session cache | OTP + session + queue | OTP + session + cache + queue | — | OTP = single point of failure nếu Redis down |

---

## 6. STORAGE — S3 (Media & Files)

| Component | dev | staging | production | demo | Ghi chú |
|---|---|---|---|---|---|
| **Bucket Name** | `aa-dev-media` | `aa-stg-media` | `aa-prod-media` | `aa-demo-media` | Globally unique |
| **Block Public Access** | ✅ All 4 ON | ✅ All 4 ON | ✅ All 4 ON | ✅ All 4 ON | |
| **Versioning** | ⚡ Optional | ⚡ Optional | ✅ **Enabled** | ❌ | Prod: restore nếu overwrite nhầm |
| **Object Ownership** | ACLs disabled | ACLs disabled | ACLs disabled | ACLs disabled | AWS recommended |
| **Encryption** | SSE-S3 (default) | SSE-S3 | SSE-S3 | SSE-S3 | |
| **Folder: uploads/** | ✅ | ✅ | ✅ | ✅ | Booking docs, passport scans — presigned URL only |
| **Folder: images/** | ✅ | ✅ | ✅ | ✅ | Tour photos — via CloudFront |
| **Folder: documents/** | ✅ | ✅ | ✅ | ✅ | PDF brochures, itineraries — presigned URL |
| **Folder: exports/** | ✅ | ✅ | ✅ | ✅ | Receipts, reports — auto-delete |
| **Lifecycle: exports/** | ❌ | ❌ | ✅ Expire after 90 days | ❌ | |
| **Lifecycle: noncurrent versions** | ❌ | ❌ | ✅ Delete after 30 days | ❌ | Giữ versioning nhưng tiết kiệm chi phí |
| **Bucket Policy** | ❌ | ❌ | ✅ Allow CloudFront OAC → images/* | ❌ | |
| **Access Pattern** | EC2 via IAM role | EC2 via IAM role | EC2 via IAM role + CloudFront | EC2 via IAM role | Không bao giờ public URL trực tiếp |
| **Presigned URL Expiry** | 60 phút | 60 phút | **60 phút** | 60 phút | Tùy loại file có thể điều chỉnh |

---

## 7. SECRETS MANAGER

| Secret Name | dev | staging | production | demo | Nội dung |
|---|---|---|---|---|---|
| `aa/dev/db` | ✅ | ❌ | ❌ | ❌ | host, port, dbname, username, password |
| `aa/stg/db` | ❌ | ✅ | ❌ | ❌ | host, port, dbname, username, password |
| `aa/prod/db` | ❌ | ❌ | ✅ | ❌ | host, port, dbname, username, password |
| `aa/demo/db` | ❌ | ❌ | ❌ | ✅ | |
| `aa/dev/jwt` | ✅ | ❌ | ❌ | ❌ | secret_key (64-char) |
| `aa/stg/jwt` | ❌ | ✅ | ❌ | ❌ | secret_key (64-char, khác prod) |
| `aa/prod/jwt` | ❌ | ❌ | ✅ | ❌ | secret_key (64-char) |
| `aa/prod/payment_api` | ❌ | ❌ | ✅ | ❌ | api_key, webhook_secret |
| `aa/stg/payment_api` | ❌ | ✅ | ❌ | ❌ | Test keys từ payment provider |
| `aa/prod/maps_api` | ❌ | ❌ | ✅ | ❌ | api_key |
| `aa/prod/email_smtp` | ❌ | ❌ | ✅ | ❌ | host, port, username, password |
| `aa/stg/email_smtp` | ❌ | ✅ | ❌ | ❌ | |
| **Automatic Rotation** | ❌ | ❌ | ✅ (sau launch) | ❌ | Lambda rotation function |
| **Encryption** | aws/secretsmanager | aws/secretsmanager | aws/secretsmanager | aws/secretsmanager | Default key |

---

## 8. IAM — ROLES & POLICIES

| Component | dev | staging | production | demo | Ghi chú |
|---|---|---|---|---|---|
| **EC2 Role** | `aa-role-ec2-dev` | `aa-role-ec2-stg` | `aa-role-ec2-prod` | `aa-role-ec2-demo` | 1 role per env |
| **Policy: S3 Access** | ✅ aa-dev-media only | ✅ aa-stg-media only | ✅ aa-prod-media only | ✅ aa-demo-media only | Inline policy giới hạn per bucket |
| **Policy: CloudWatch Agent** | ✅ CloudWatchAgentServerPolicy | ✅ | ✅ | ✅ | AWS managed policy |
| **Policy: SSM Session Manager** | ✅ AmazonSSMManagedInstanceCore | ✅ | ✅ | ✅ | Thay thế SSH |
| **Policy: Secrets Manager** | ✅ aa/dev/* only | ✅ aa/stg/* only | ✅ aa/prod/* only | ✅ aa/demo/* only | Inline policy |
| **GitHub Actions Role/User** | ❌ | ✅ deploy access stg | ✅ deploy access prod | ❌ | Confirm OIDC vs static keys với Shivam |
| **IAM Group: aa-devops** | ✅ shared | ✅ shared | ✅ shared | ✅ shared | Admin_nghiep là member |
| **Group Policies** | EC2+RDS+S3+CW ReadOnly | same | same | same | ReadOnly cho monitoring |
| **Root MFA** | N/A | N/A | ✅ **Bắt buộc** | N/A | Verify ngay |
| **IAM User MFA** | ✅ admin_nghiep-mfa | ✅ | ✅ | ✅ | |

---

## 9. MONITORING — CloudWatch

| Component | dev | staging | production | demo | Ghi chú |
|---|---|---|---|---|---|
| **SNS Topic** | ❌ | ✅ `aa-stg-alerts` | ✅ `aa-prod-alerts` | ❌ | Email subscription confirmed |
| **Alarm: EC2 CPU > 70%** | ❌ | ✅ | ✅ `aa-prod-ec2-cpu-high` | ❌ | 3/3 × 5min = 15 phút liên tục |
| **Alarm: EC2 Status Check** | ❌ | ✅ | ✅ `aa-prod-ec2-status` | ❌ | 2/2 × 1min |
| **Alarm: RDS Connections > 70** | ❌ | ✅ | ✅ `aa-prod-rds-connections` | ❌ | db.t3.small max ~172 |
| **Alarm: RDS Storage < 5GB** | ❌ | ✅ | ✅ `aa-prod-rds-storage` | ❌ | In bytes: 5368709120 |
| **Alarm: ALB 5xx Count > 10** | ❌ | ✅ | ✅ `aa-prod-alb-5xx` | ❌ | 2/2 × 5min |
| **Alarm: Redis Memory < 100MB** | ❌ | ⚡ | ✅ `aa-prod-redis-memory` | ❌ | In bytes: 104857600 |
| **Alarm: Billing > $240** | ❌ | ❌ | ✅ `aa-billing-prod` | ❌ | = 80% của $300 budget |
| **Log Group: Nginx Access** | ❌ | ✅ `/aa/stg/nginx-access` | ✅ `/aa/prod/nginx-access` | ❌ | Retention: 30 days |
| **Log Group: Nginx Error** | ❌ | ✅ `/aa/stg/nginx-error` | ✅ `/aa/prod/nginx-error` | ❌ | Retention: 30 days |
| **Log Group: App** | ❌ | ✅ `/aa/stg/app` | ✅ `/aa/prod/app` | ❌ | Retention: 30 days |
| **CloudWatch Dashboard** | ❌ | ⚡ Optional | ✅ `AdventureAsia-Production` | ❌ | 7 widgets |
| **CloudWatch Agent on EC2** | ❌ | ✅ | ✅ | ❌ | Memory + disk metrics (không có trong default) |

---

## 10. CI/CD — GitHub Actions

| Component | dev | staging | production | demo | Ghi chú |
|---|---|---|---|---|---|
| **GitHub Environment** | ❌ | ✅ `staging` · no protection | ✅ `production` · required reviewers | ❌ | |
| **Required Reviewer (prod)** | — | — | ✅ Shivam + Nghiep | — | |
| **Workflow File** | ✅ `.github/workflows/deploy.yml` | ✅ Job: deploy-staging | ✅ Job: deploy-production | ❌ | |
| **Trigger: Staging** | — | `push to develop` (auto) | — | — | After build+test pass |
| **Trigger: Production** | — | — | Manual approval gate | — | After staging deploy |
| **Deploy Method** | git pull trực tiếp | SSM Send-Command | SSM Send-Command | Manual | Không cần SSH |
| **Health Check sau deploy** | `curl /health` | ✅ `curl stg.../health` | ✅ `curl api.../health` | ❌ | Fail = rollback |
| **Rollback** | git checkout | Redeploy prev commit | Redeploy prev commit | Manual | DB migration cần riêng |
| **GitHub Secrets** | — | STG_EC2_INSTANCE_ID | PROD_EC2_INSTANCE_ID | — | + AWS credentials |
| **Branch Strategy** | `feature/*` | auto từ `develop` | manual từ `develop` | manual | |

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

## 12. COST CONTROL

| Component | dev | staging | production | demo | Ghi chú |
|---|---|---|---|---|---|
| **AWS Budget** | ⚡ `aa-dev-monthly` $20 | ✅ `aa-stg-monthly` $50 | ✅ `aa-prod-monthly` $300 | ❌ | |
| **Alert thresholds** | 80% | 70% + 100% | 70% + 100% | — | SNS email |
| **Resource Tagging** | ✅ | ✅ | ✅ | ✅ | Project + Env + Owner + Component |
| **Cost Allocation Tags** | ✅ activate | ✅ | ✅ | ✅ | Xem breakdown theo env trong Cost Explorer |
| **Stop khi không dùng** | ✅ Stop EC2 + RDS | ✅ Stop ngoài giờ | ❌ 24/7 | ✅ Stop khi không demo | |
| **Reserved Instances** | ❌ | ❌ | ⚡ Sau 3 tháng stable | ❌ | Giảm ~35% EC2 + RDS |

---

## 13. TỔNG HỢP CHI PHÍ ƯỚC TÍNH (On-Demand · tháng)

| Service | dev | staging | production | demo | Total/tháng |
|---|---|---|---|---|---|
| EC2 | t3.micro ~$9 | t3.small ~$17 (stop nights/weekends → ~$9) | t3.medium ~$34 | t3.micro ~$9 (chạy khi cần) | ~$55–70 |
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
| **TOTAL** | **~$44** | **~$76** | **~$186** | **~$30** | **~$336/tháng** |

> 💡 Tối ưu ngay được: Stop dev+stg EC2+RDS ngoài giờ làm → tiết kiệm ~$40/tháng. Sau 3 tháng stable: Reserved Instances cho prod EC2+RDS → tiết kiệm thêm ~$40/tháng.

---

## 14. CHECKLIST THỨ TỰ TẠO (Build Order)

```
PHASE 1 — FOUNDATION
  □ 1.  IAM: Root MFA confirm
  □ 2.  IAM: Roles (aa-role-ec2-dev/stg/prod) + inline policies
  □ 3.  VPC: aa-vpc-prod + 4 subnets + IGW + NAT + route tables
  □ 4.  Security Groups: aa-sg-alb, aa-sg-app, aa-sg-db, aa-sg-redis
  □ 5.  ACM: Request *.adventureasia.app certificate → validate via Route 53

PHASE 2 — DATA LAYER
  □ 6.  Secrets Manager: tạo tất cả secrets (host điền sau)
  □ 7.  S3: aa-dev/stg/prod/demo-media + folder structure + lifecycle
  □ 8.  RDS Subnet Group: aa-db-subnet-group (private subnets)
  □ 9.  RDS: aa-stg-db trước → aa-prod-db sau (Multi-AZ, deletion protection)
  □ 10. RDS: Tạo aa_app_user, cập nhật endpoint vào Secrets Manager
  □ 11. ElastiCache Subnet Group: aa-redis-subnet-group
  □ 12. ElastiCache: aa-stg-redis → aa-prod-redis

PHASE 3 — COMPUTE
  □ 13. Key Pair: aa-ec2-keypair (tải về, lưu an toàn)
  □ 14. EC2: aa-stg-api (test trước)
  □ 15. EC2: Nginx config + PM2 + CloudWatch agent trên staging
  □ 16. EC2: Deploy app lên staging, test /health endpoint
  □ 17. EC2: aa-prod-api (sau khi staging ổn định)
  □ 18. Target Groups: aa-tg-stg + aa-tg-prod (register instances)
  □ 19. ALB: aa-alb-stg + aa-alb-prod (HTTPS + redirect HTTP)
  □ 20. Route 53: DNS records cho tất cả subdomains
  □ 21. CloudFront: distributions cho static assets

PHASE 4 — OBSERVABILITY
  □ 22. SNS: aa-stg-alerts + aa-prod-alerts (confirm email)
  □ 23. CloudWatch Alarms: 6 alarms cho production
  □ 24. CloudWatch Dashboard: AdventureAsia-Production

PHASE 5 — CI/CD
  □ 25. GitHub Environments: staging (auto) + production (manual gate)
  □ 26. GitHub Secrets: tất cả 7 secrets
  □ 27. Workflow file: .github/workflows/deploy.yml
  □ 28. Test full pipeline: push → build → staging → health check

PHASE 6 — COST CONTROLS
  □ 29. AWS Budgets: 3 budgets với alert thresholds
  □ 30. Cost Allocation Tags: activate trong Billing
```

---

*Adventure Asia App — Component Build Table | 4 Environments | Nghiep | DevOps Mentorship*
*Based on: PRD Technical Spec · Mentor Architecture Plan (Shivam) · ap-southeast-1*
