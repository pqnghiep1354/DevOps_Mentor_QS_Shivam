# AAA — Component Build Table (Updated · 10-Step Guide)
## Adventure Asia App · AWS Deployment Components
**Updated:** 13/03/2026 · Based on: AAA_Practice_StepByStep_Guide.md (10 Steps)
**Architecture:** AWS Organizations + ECS Fargate + Docker + GitHub Actions OIDC

---

> **Naming Convention:** `aa-[env]-[component]`
> **Tag Standard (all resources):**
> `Project=AdventureAsia · Env=[dev|stg|prod] · Owner=Nghiep · Component=[component]`

---

## SECTION 1 — AWS ORGANIZATIONS & ACCOUNTS

| Component | Type | Value | Step | Status |
|---|---|---|---|---|
| Management Account | AWS Account | 867490540162 (pqnghiep1354) | 1 | Done |
| AA-DEV Account | AWS Account | 710590321660 | 1 | Done |
| AA-STAGING Account | AWS Account | 593110023608 | 1 | Done |
| AA-PROD Account | AWS Account | 007050358335 | 1 | Done |
| Organization Feature Set | Organizations | ALL (not Consolidated Billing) | 1 | Done |
| Root ID | Organizations | r-zgsn | 1 | Done |
| OU: Workloads | Organizational Unit | Parent of Non-Prod + Prod | 1 | Done |
| OU: Non-Production | Organizational Unit | Contains DEV + STAGING | 1 | Done |
| OU: Production | Organizational Unit | Contains PROD | 1 | Done |
| SCP: AA-RegionLock-SEA | Service Control Policy | p-qme9rmxr · Deny outside ap-southeast-1 | 1 | Done |

---

## SECTION 2 — IAM IDENTITY CENTER (SSO)

| Component | Type | Value | Step | Status |
|---|---|---|---|---|
| SSO Instance | IAM Identity Center | ssoins-8210d1d9b37ec22f | 1 | Done |
| Identity Store | IAM Identity Center | d-9667a69473 | 1 | Done |
| SSO Region | Region | ap-southeast-1 | 1 | Done |
| SSO Start URL | Portal URL | https://d-9667a69473.awsapps.com/start | 1 | Done |
| SSO Session | CLI Config | aa-sso | 1 | Done |
| User: nghiep | SSO User | 896a151c-60c1-70f6-3235-3af3e738ab91 | 1 | Done |
| User: shivam | SSO User | (pending email acceptance) | 1 | Pending |
| Permission Set: AA-Admin | Permission Set | ps-8210f3b57e3dd151 · 1h session · AdministratorAccess | 1 | Done |
| Permission Set: AA-DevOps | Permission Set | ps-821034db9a29e140 · 8h session · PowerUserAccess | 1 | Done |
| Permission Set: AA-ReadOnly | Permission Set | ps-82102dc2b7bf9e52 · 8h session · ReadOnlyAccess | 1 | Done |
| Assignment: nghiep → DEV | Account Assignment | AA-Admin + AA-DevOps | 1 | Done |
| Assignment: nghiep → STAGING | Account Assignment | AA-Admin + AA-DevOps | 1 | Done |
| Assignment: nghiep → PROD | Account Assignment | AA-Admin + AA-DevOps | 1 | Done |
| CLI Profile: aa-dev | AWS CLI | --profile aa-dev · Account 710590321660 | 1 | Done |
| CLI Profile: aa-stg | AWS CLI | --profile aa-stg · Account 593110023608 | 1 | Done |
| CLI Profile: aa-prod | AWS CLI | --profile aa-prod · Account 007050358335 | 1 | Done |

---

## SECTION 3 — IAM ROLES (per account: DEV / STG / PROD)

> Created in **each member account** individually.
> AA-CICD role created in the account that hosts ECR + ECS for that env.

| Component | Type | Trust Principal | Permissions | Step | Status |
|---|---|---|---|---|---|
| AA-Admin | IAM Role | SSO (via Identity Center) | AdministratorAccess | 2 | Pending |
| AA-DevOps | IAM Role | SSO (via Identity Center) | ECS + ECR + VPC + ALB + CW + Secrets + RDS (no billing/IAM) | 2 | Pending |
| AA-Developer-Read | IAM Role | SSO (via Identity Center) | ReadOnlyAccess + CW Logs | 2 | Pending |
| AA-CICD | IAM Role | GitHub OIDC | ECR push + ECS update + IAM PassRole | 2 | Pending |
| aa-role-ecs-task-execution | IAM Role | ecs-tasks.amazonaws.com | AmazonECSTaskExecutionRolePolicy + SecretsManager read | 2 | Pending |
| aa-role-ecs-task-dev | IAM Role | ecs-tasks.amazonaws.com | Secrets aa/dev/* + S3 aa-dev-media + CW Logs | 2 | Pending |
| aa-role-ecs-task-stg | IAM Role | ecs-tasks.amazonaws.com | Secrets aa/stg/* + S3 aa-stg-media + CW Logs | 2 | Pending |
| aa-role-ecs-task-prod | IAM Role | ecs-tasks.amazonaws.com | Secrets aa/prod/* + S3 aa-prod-media + CW Logs | 2 | Pending |
| GitHub OIDC Provider | IAM OIDC | token.actions.githubusercontent.com | sts:AssumeRoleWithWebIdentity | 2 | Pending |

---

## SECTION 4 — NETWORKING (VPC)

> One VPC per environment (DEV / STG / PROD) in each member account.
> Table below shows PROD. DEV and STG follow same structure, smaller instance sizes.

| Component | Name | Value | Step | Status |
|---|---|---|---|---|
| VPC | aa-vpc-prod | 10.0.0.0/16 · DNS enabled | 4 | Pending |
| Public Subnet 1a | aa-subnet-public-1a | 10.0.1.0/24 · ap-southeast-1a | 4 | Pending |
| Public Subnet 1b | aa-subnet-public-1b | 10.0.2.0/24 · ap-southeast-1b | 4 | Pending |
| Private Subnet 1a | aa-subnet-private-1a | 10.0.3.0/24 · ap-southeast-1a | 4 | Pending |
| Private Subnet 1b | aa-subnet-private-1b | 10.0.4.0/24 · ap-southeast-1b | 4 | Pending |
| Internet Gateway | aa-igw | Attached to aa-vpc-prod | 4 | Pending |
| Elastic IP | aa-eip-nat | For NAT Gateway | 4 | Pending |
| NAT Gateway | aa-nat-1a | In public-1a · ~$35/month | 4 | Pending |
| Route Table (public) | aa-rt-public | 0.0.0.0/0 → IGW | 4 | Pending |
| Route Table (private) | aa-rt-private | 0.0.0.0/0 → NAT | 4 | Pending |
| Security Group: ALB | aa-sg-alb | IN: 443+80 from 0.0.0.0/0 | 4 | Pending |
| Security Group: App | aa-sg-app | IN: 3000-3015 from aa-sg-alb only | 4 | Pending |
| Security Group: DB | aa-sg-db | IN: 5432 from aa-sg-app only | 4 | Pending |
| Security Group: Redis | aa-sg-redis | IN: 6379 from aa-sg-app only | 4 | Pending |

---

## SECTION 5 — CONTAINER REGISTRY (ECR)

> 9 services × 3 environments = 27 repositories total.
> Sprint 1-2: create api-gateway + auth first (6 repos).

| Repository Name | Environment | Tag Mutability | Scan on Push | Lifecycle | Step | Status |
|---|---|---|---|---|---|---|
| aa-api-gateway-dev | DEV | MUTABLE | Yes | 10 tagged / 7d untagged | 5 | Pending |
| aa-api-gateway-stg | STG | MUTABLE | Yes | 10 tagged / 7d untagged | 5 | Pending |
| aa-api-gateway-prod | PROD | **IMMUTABLE** | Yes | 10 tagged / 7d untagged | 5 | Pending |
| aa-auth-dev | DEV | MUTABLE | Yes | 10 tagged / 7d untagged | 5 | Pending |
| aa-auth-stg | STG | MUTABLE | Yes | 10 tagged / 7d untagged | 5 | Pending |
| aa-auth-prod | PROD | **IMMUTABLE** | Yes | 10 tagged / 7d untagged | 5 | Pending |
| aa-websocket-dev | DEV | MUTABLE | Yes | 10 tagged / 7d untagged | 7 | Future |
| aa-websocket-stg | STG | MUTABLE | Yes | 10 tagged / 7d untagged | 7 | Future |
| aa-websocket-prod | PROD | **IMMUTABLE** | Yes | 10 tagged / 7d untagged | 7 | Future |
| aa-trip-dev/stg/prod | all | per env | Yes | same | 7 | Future |
| aa-booking-dev/stg/prod | all | per env | Yes | same | 7 | Future |
| aa-user-dev/stg/prod | all | per env | Yes | same | 7 | Future |
| aa-sales-dev/stg/prod | all | per env | Yes | same | 7 | Future |
| aa-cms-dev/stg/prod | all | per env | Yes | same | 7 | Future |
| aa-map-dev/stg/prod | all | per env | Yes | same | 7 | Future |

---

## SECTION 6 — ECS CLUSTERS

| Component | Name | Container Insights | Account | Step | Status |
|---|---|---|---|---|---|
| ECS Cluster DEV | ecs-dev | Enabled | AA-DEV (710590321660) | 6 | Pending |
| ECS Cluster STG | ecs-stg | Enabled | AA-STAGING (593110023608) | 6 | Pending |
| ECS Cluster PROD | ecs-prod | Enabled | AA-PROD (007050358335) | 6 | Pending |

---

## SECTION 7 — ECS SERVICES (FARGATE)

> Sprint 1-2: api-gateway + auth only. Remaining 7 services: Sprint 3+.
> ALL tasks run in **private subnets**, NO public IP, NO SSH.
> Access via ALB only. Debug via ECS Exec.

| Service | Port | ECS Service Name | vCPU | Memory | Min Tasks | Step | Status |
|---|---|---|---|---|---|---|---|
| aa-api-gateway | 3000 | aa-api-gateway-[env]-svc | 0.5 | 1024 MB | 1 | 6 | Pending |
| aa-auth | 3001 | aa-auth-[env]-svc | 0.5 | 1024 MB | 1 | 6 | Pending |
| aa-websocket | 3002 | aa-websocket-[env]-svc | 0.5 | 1024 MB | 1 | 7 | Future |
| aa-trip | 3010 | aa-trip-[env]-svc | 0.5 | 1024 MB | 1 | 7 | Future |
| aa-booking | 3011 | aa-booking-[env]-svc | 0.5 | 1024 MB | 1 | 7 | Future |
| aa-user | 3012 | aa-user-[env]-svc | 0.25 | 512 MB | 1 | 7 | Future |
| aa-sales | 3013 | aa-sales-[env]-svc | 0.25 | 512 MB | 1 | 7 | Future |
| aa-cms | 3014 | aa-cms-[env]-svc | 0.25 | 512 MB | 1 | 7 | Future |
| aa-map | 3015 | aa-map-[env]-svc | 0.25 | 512 MB | 1 | 7 | Future |

**Task Definition fields (all services):**
```
networkMode          : awsvpc  (required for Fargate)
requiresCompatibility: FARGATE
taskRoleArn          : aa-role-ecs-task-[env]
executionRoleArn     : aa-role-ecs-task-execution
logDriver            : awslogs → /aa/[env]/ecs-[service]
healthCheck          : GET /health → HTTP 200
```

---

## SECTION 8 — LOAD BALANCER (ALB)

> One ALB per environment (STG + PROD). DEV uses direct ECS Exec / port-forward.

| Component | Name | Value | Step | Status |
|---|---|---|---|---|
| ALB (Staging) | aa-alb-stg | Internet-facing · Public subnets 1a+1b | 6 | Pending |
| ALB (Production) | aa-alb-prod | Internet-facing · Public subnets 1a+1b | 6 | Pending |
| ALB Idle Timeout | Both | **3600 seconds** (required for WebSocket) | 6 | Pending |
| Listener HTTPS | Both | Port 443 · SSL via ACM | 6 | Pending |
| Listener HTTP | Both | Port 80 → redirect to 443 | 6 | Pending |
| ACM Certificate | *.adventureasia.app | Wildcard · DNS validation via Route 53 | 6 | Pending |
| Target Group: API Gateway | aa-tg-[env]-api | Type: **IP** (not Instance) · Port 3000 · /health | 6 | Pending |
| Target Group: Auth | aa-tg-[env]-auth | Type: **IP** · Port 3001 · /health | 6 | Pending |
| Target Group: WebSocket | aa-tg-[env]-ws | Type: **IP** · Port 3002 · Stickiness: 1 day | 7 | Future |
| ALB Rule: /ws/* | Routing | → WebSocket Target Group | 7 | Future |
| ALB Rule: /api/v1/auth/* | Routing | → Auth Target Group | 6 | Pending |
| ALB Rule: /api/v1/trips/* | Routing | → Trip Target Group | 7 | Future |
| ALB Rule: default | Routing | → API Gateway Target Group | 6 | Pending |

---

## SECTION 9 — DATA LAYER

### 9A — RDS PostgreSQL

| Component | Name | DEV | STG | PROD | Step | Status |
|---|---|---|---|---|---|---|
| RDS Instance | aa-[env]-db | db.t3.micro | db.t3.micro | db.t3.small | 4 | Pending |
| Multi-AZ | — | No | No | **Yes** | 4 | Pending |
| Storage | — | 20 GB | 20 GB | 30 GB | 4 | Pending |
| Backup Retention | — | 1 day | 7 days | **30 days** | 4 | Pending |
| Deletion Protection | — | Off | Off | **On** | 4 | Pending |
| Point-in-Time Recovery | — | Off | Off | **Enabled** | 4 | Pending |
| Storage Encryption | — | Off | Off | **Enabled** | 4 | Pending |
| DB Subnet Group | aa-db-subnet-group | Private subnets 1a+1b | same | same | 4 | Pending |
| App DB User | aa_app_user | SELECT/INSERT/UPDATE/DELETE only (no DROP/ALTER) | — | — | 4 | Pending |
| Engine | PostgreSQL 15.4 | — | — | — | 4 | Pending |

### 9B — ElastiCache Redis

| Component | Name | DEV | STG | PROD | Step | Status |
|---|---|---|---|---|---|---|
| Redis Instance | aa-[env]-redis | cache.t3.micro | cache.t3.micro | cache.t3.micro | 4 | Pending |
| Redis Version | — | 7.x | 7.x | 7.x | 4 | Pending |
| Encryption at rest | — | No | No | **Yes** | 4 | Pending |
| Encryption in transit | — | No | No | **Yes** (rediss://) | 4 | Pending |
| Parameter: maxmemory-policy | aa-redis-params | allkeys-lru | same | same | 4 | Pending |
| Parameter: notify-keyspace-events | aa-redis-params | Ex | same | same | 4 | Pending |
| Redis Responsibilities | — | OTP · Session · Cache · Rate-limit · Job Queue | — | — | — | — |

### 9C — S3 Buckets

| Bucket | Environment | Public Access | Folder Structure | Step | Status |
|---|---|---|---|---|---|
| aa-dev-media | DEV | Blocked | images/trips/ · images/avatars/ · uploads/ · documents/ · exports/ | 4 | Pending |
| aa-stg-media | STG | Blocked | same | 4 | Pending |
| aa-prod-media | PROD | Blocked | same | 4 | Pending |
| Access: public images | PROD | Via CloudFront only | images/trips/* | 6 | Pending |
| Access: private files | PROD | Presigned URL (60 min) | uploads/* · documents/* | 6 | Pending |

---

## SECTION 10 — SECRETS MANAGER

> Pattern: `aa/[env]/[service]`
> Access: ECS Task Role scoped to `aa/[env]/*` only (not cross-env)

| Secret Name | Environment | Contains | Step | Status |
|---|---|---|---|---|
| aa/stg/db | STG | host, port, dbname, username, password | 7 | Pending |
| aa/prod/db | PROD | host, port, dbname, username, password | 7 | Pending |
| aa/stg/jwt | STG | secret_key, access_ttl, refresh_ttl | 7 | Pending |
| aa/prod/jwt | PROD | secret_key, access_ttl, refresh_ttl | 7 | Pending |
| aa/stg/redis | STG | url (redis://) | 7 | Pending |
| aa/prod/redis | PROD | url (rediss://) | 7 | Pending |
| aa/prod/s3 | PROD | bucket_name, region | 7 | Pending |
| aa/prod/email_smtp | PROD | host, port, username, password | 7 | Pending |
| aa/stg/email_smtp | STG | host, port, username, password | 7 | Pending |
| aa/prod/crm_api_key | PROD | api_key, webhook_url | 7 | Pending |
| aa/prod/maps_api | PROD | mapbox_key or google_maps_key | 7 | Pending |
| aa/prod/payment_api | PROD | api_key, webhook_secret | 7 | Pending |
| aa/stg/demo_accounts | STG | fixed_otp=123456 (investor demo only) | 7 | Pending |

---

## SECTION 11 — OBSERVABILITY

### 11A — CloudWatch Log Groups

| Log Group | Retention | Environment | Step | Status |
|---|---|---|---|---|
| /aa/dev/ecs-api-gateway | 14 days | DEV | 8 | Pending |
| /aa/stg/ecs-api-gateway | 14 days | STG | 8 | Pending |
| /aa/prod/ecs-api-gateway | **30 days** | PROD | 8 | Pending |
| /aa/stg/ecs-auth | 14 days | STG | 8 | Pending |
| /aa/prod/ecs-auth | 30 days | PROD | 8 | Pending |
| /aa/stg/ecs-websocket | 14 days | STG | 8 | Pending |
| /aa/prod/ecs-websocket | 30 days | PROD | 8 | Pending |
| (repeat for all 9 services × dev/stg/prod) | — | — | 8 | Pending |

### 11B — SNS Topics

| Component | Name | Subscribers | Step | Status |
|---|---|---|---|---|
| SNS Topic (PROD) | aa-prod-alerts | nghiep@adventureasia.app (email confirmed) | 8 | Pending |
| SNS Topic (STG) | aa-stg-alerts | nghiep@adventureasia.app | 8 | Pending |

### 11C — CloudWatch Alarms

| Alarm Name | Metric | Threshold | Action | Step | Status |
|---|---|---|---|---|---|
| aa-prod-ecs-cpu-high | ECS CPUUtilization | >70% (3/3 × 5min) | SNS prod | 8 | Pending |
| aa-prod-ecs-memory-high | ECS MemoryUtilization | >80% | SNS prod | 8 | Pending |
| aa-prod-ecs-unhealthy-tasks | ECS running < desired | ≥1 (2/2 × 1min) | SNS prod | 8 | Pending |
| aa-prod-rds-connections | DB Connections | >70 | SNS prod | 8 | Pending |
| aa-prod-rds-storage | FreeStorageSpace | <5 GB | SNS prod | 8 | Pending |
| aa-prod-alb-5xx | ALB 5XX Count | >10 (per 5min) | SNS prod | 8 | Pending |
| aa-prod-redis-memory | Redis FreeableMemory | <100 MB | SNS prod | 8 | Pending |
| aa-stg-ecs-cpu-high | ECS CPUUtilization | >70% | SNS stg | 8 | Pending |
| aa-stg-alb-5xx | ALB 5XX Count | >10 | SNS stg | 8 | Pending |

### 11D — Dashboard

| Component | Name | Widgets | Step | Status |
|---|---|---|---|---|
| CloudWatch Dashboard | AdventureAsia-Staging | ECS CPU · ALB Requests · ALB 5xx · Log Errors | 8 | Pending |
| CloudWatch Dashboard | AdventureAsia-Production | Same + RDS Connections · Redis Memory | 8 | Pending |

---

## SECTION 12 — CI/CD PIPELINE (GITHUB ACTIONS)

| Component | Value | Step | Status |
|---|---|---|---|
| GitHub Environment: staging | No protection · auto-deploy on push to develop | 9 | Pending |
| GitHub Environment: production | Required reviewers: Shivam + Nghiep · Prevent self-review | 9 | Pending |
| Auth Method | **GitHub OIDC** (no static keys) via AA-CICD role | 9 | Pending |
| Workflow file | .github/workflows/deploy.yml | 9 | Pending |
| Job 1: build-test | npm ci + lint + test | 9 | Pending |
| Job 2: build-push | docker build + ECR push (tag: commit SHA) | 9 | Pending |
| Job 3: deploy-staging | ECS update-service + wait stable + health check | 9 | Pending |
| Job 4: deploy-production | Manual gate → ECS update-service + health check | 10 | Pending |
| Rollback method | `aws ecs update-service --task-definition [FAMILY]:[PREV_REV]` (~2 min) | 10 | Pending |
| Image tagging | Staging: `sha-xxxxxxx` · Production: same SHA (retag) | 9 | Pending |

**GitHub Secrets required:**
```
AWS_REGION            = ap-southeast-1
AWS_ROLE_ARN          = arn:aws:iam::[ACCOUNT]:role/AA-CICD
ECR_REGISTRY          = [ACCOUNT].dkr.ecr.ap-southeast-1.amazonaws.com
STG_ECS_CLUSTER       = ecs-stg
STG_ECS_SERVICE       = aa-api-gateway-stg-svc
STG_TASK_DEFINITION   = aa-api-gateway-stg
PROD_ECS_CLUSTER      = ecs-prod
PROD_ECS_SERVICE      = aa-api-gateway-prod-svc
PROD_TASK_DEFINITION  = aa-api-gateway-prod
```

---

## SECTION 13 — BILLING & COST CONTROLS

| Component | Name | Value | Step | Status |
|---|---|---|---|---|
| AWS Budget | AAA-Monthly-Total | $50/month · alert at 80% ($40) | 1 | Done |
| Cost Explorer | Management Account | Enable Cost Allocation Tags | 1 | Pending |
| ECR Lifecycle Policy | All repos | Keep 10 tagged · Delete untagged after 7 days | 5 | Pending |
| ECS Fargate Spot | DEV + STG only | Up to 70% cost reduction | 7 | Future |
| NAT Gateway | 1 per env | ~$35/month each — create only when needed | 4 | Pending |

**Estimated cost (Sprint 1-2, 2 ECS services running):**
```
DEV environment     : ~$44/month
STG environment     : ~$76/month
PROD environment    : ~$186/month  (Multi-AZ RDS + NAT + ALB)
Total estimated     : ~$306/month
```

---

## OVERALL BUILD STATUS

| Section | Components | Done | Pending | Future |
|---|---|---|---|---|
| 1. Organizations & Accounts | 10 | 10 | 0 | 0 |
| 2. IAM Identity Center (SSO) | 16 | 15 | 1 | 0 |
| 3. IAM Roles | 9 | 0 | 9 | 0 |
| 4. Networking (VPC) | 14 | 0 | 14 | 0 |
| 5. ECR Repositories | 27 | 0 | 6 | 21 |
| 6. ECS Clusters + ALB | 14 | 0 | 8 | 6 |
| 7. ECS Services (Fargate) | 9 | 0 | 2 | 7 |
| 8. Data Layer (RDS+Redis+S3) | 15 | 0 | 15 | 0 |
| 9. Secrets Manager | 13 | 0 | 13 | 0 |
| 10. Observability | 20 | 0 | 20 | 0 |
| 11. CI/CD Pipeline | 10 | 0 | 10 | 0 |
| 12. Cost Controls | 5 | 1 | 4 | 0 |
| **TOTAL** | **162** | **26** | **102** | **34** |

---

## KEY CHANGES vs PREVIOUS 8-STEP GUIDE

| Area | Old (8-Step) | New (10-Step) | Impact |
|---|---|---|---|
| Compute | EC2 + PM2 + Nginx | **ECS Fargate + Docker** | No SSH, no server management |
| Account strategy | 1 account, naming isolation | **3 separate accounts (Option A)** | True blast-radius isolation |
| Auth | IAM users + Access Keys | **SSO + OIDC (no static keys)** | More secure, no key rotation needed |
| Deploy method | SSH → git pull → pm2 restart | **docker build → ECR push → ECS rolling update** | Zero-downtime, reproducible |
| Rollback | git revert + pm2 restart (~10 min) | **ECS task definition revision (~2 min)** | Faster, safer |
| Secrets | .env files on server | **AWS Secrets Manager (runtime inject)** | No secrets in code or filesystem |
| Scaling | Manual EC2 resize | **ECS auto-scaling by CPU/memory** | On-demand, pay per use |
| Environments | Naming convention only | **Separate AWS accounts per env** | Strongest possible isolation |

---

*Adventure Asia App — Component Build Table (Updated for 10-Step Guide)*
*Architecture: AWS Organizations + ECS Fargate + Docker + GitHub Actions OIDC*
*Region: ap-southeast-1 · Updated: 13/03/2026 · Prepared by Nghiep*
