# AAA — Component Build Table v5
## Adventure Asia App · AWS Deployment Components

**Updated:** 18/03/2026 · Final Practice Report
**Architecture:** EC2 Phase 1 → ECS Fargate Phase 2 (future)
**Guide:** AA AWS DevOps Setup Guide (Shivam · 15/03/2026)
**Region:** ap-southeast-1

---

## OVERALL PROGRESS SUMMARY

| Step | Content | DEV | STG | PROD |
|---|---|---|---|---|
| 1 | AWS Organizations + SSO | ✅ | ✅ | ✅ |
| 2 | MFA Enforcement | ✅ | ✅ | ✅ |
| 3 | IAM Roles + OIDC | ✅ | ✅ | ✅ |
| 4 | VPC + Security Groups | ✅ | ✅ | ✅ |
| 5 | ALB + EC2 Launch | ✅ | ✅ | ✅ |
| 6 | Laravel Deployment | ✅ | ⚠️ SSM pending | ⏳ |
| 7 | RDS + Redis + S3 + Secrets | ✅ | ⏳ | ⏳ |
| 8 | CloudWatch Monitoring | ✅ | ⏳ | ⏳ |
| 9 | CI/CD Pipeline | ✅ | ⏳ | ⏳ |
| 10 | Staging Gate | ⚠️ | ⏳ | ⏳ |

**DEV: 100% complete · STG: 40% · PROD: 20% infrastructure only**

---

## SECTION 1 — AWS ORGANIZATIONS & ACCOUNTS ✅

| Component | Value | Status |
|---|---|---|
| Management Account | 867490540162 (pqnghiep1354) | ✅ |
| AA-DEV Account | 710590321660 | ✅ |
| AA-STAGING Account | 593110023608 | ✅ |
| AA-PROD Account | 007050358335 | ✅ |
| Organization Feature Set | ALL | ✅ |
| OU: Non-Production | DEV + STAGING | ✅ |
| OU: Production | PROD | ✅ |
| SCP: AA-RegionLock-SEA | p-qme9rmxr · deny outside ap-southeast-1 | ✅ |
| Budget Alert | $50/month · 80% alert | ✅ |

---

## SECTION 2 — IAM IDENTITY CENTER ✅

| Component | Value | Status |
|---|---|---|
| SSO Instance | ssoins-8210d1d9b37ec22f | ✅ |
| SSO Start URL | https://d-9667a69473.awsapps.com/start | ✅ |
| User: nghiep | 896a151c-60c1-70f6-3235-3af3e738ab91 | ✅ |
| User: shivam | 590a55ac-60e1-7086-aee8-e83e0c728554 | ✅ |
| Permission Set: AA-Admin | 1h session · AdministratorAccess | ✅ |
| Permission Set: AA-DevOps | 8h session · PowerUserAccess | ✅ |
| Permission Set: AA-ReadOnly | 8h session · ReadOnly+CWLogs (=AA-Developer-Read) | ⚠️ rename pending |
| MFA Enforcement | Every sign-in | ✅ |
| SignInLocalDevelopmentAccess | Added to AA-Admin + AA-DevOps | ✅ |
| CLI profiles | aa-dev/stg/prod + aa-dev/stg/prod-admin | ✅ |

---

## SECTION 3 — IAM ROLES ✅

| Role | Account | Status |
|---|---|---|
| aa-role-ec2-app-dev + profile | DEV 710590321660 | ✅ |
| aa-role-ec2-app-stg + profile | STG 593110023608 | ✅ |
| aa-role-ec2-app-prod + profile | PROD 007050358335 | ✅ |
| aa-role-nat-dev + profile | DEV | ✅ |
| aa-role-nat-stg + profile | STG | ✅ |
| OIDC Provider | DEV + STG + PROD | ✅ |
| AA-CICD role | DEV + STG + PROD | ✅ |

---

## SECTION 4 — NETWORKING ✅

### VPCs

| VPC | ID | CIDR | Status |
|---|---|---|---|
| aa-dev-vpc | vpc-08930f8f35962726e | 10.0.0.0/16 | ✅ |
| aa-stg-vpc | vpc-0c6397e17adf02007 | 10.1.0.0/16 | ✅ |
| aa-prod-vpc | vpc-0c527df6ff7cd2ca9 | 10.2.0.0/16 | ✅ |

### Security Groups

| SG | DEV | STG | PROD | Status |
|---|---|---|---|---|
| aa-sg-alb | sg-0bfa9ef4a3c051cb6 | sg-090d74a1baeb1b896 | sg-0d87ddc810a95698a | ✅ |
| aa-sg-app | sg-00e2a7d1be82e5104 | sg-00281323fc9c57aaf | sg-074187fcd9af2f7c4 | ✅ |
| aa-sg-db | sg-0dae5a6e75ca2e446 | sg-054f8c56f91ecf139 | sg-0a43cd8640ae409ae | ✅ |
| aa-sg-redis | sg-0872f95442011f106 | sg-0fe1c805d5865ec80 | sg-07897bd80c0cb06aa | ✅ |
| aa-sg-nat-dev | sg-00e3dcf32ac299e62 | — | — | ✅ |
| aa-sg-nat-stg | sg-09d226fd0e8e612fd | — | — | ✅ |

### NAT Instances

| NAT | ID | EIP | Status |
|---|---|---|---|
| aa-nat-dev | i-03012d723f7e84e4b | 47.131.134.75 | ✅ IP forward enabled |
| aa-nat-stg | i-0f3873f9819d309cf | 13.215.119.211 | ✅ IP forward enabled |

---

## SECTION 5 — ALB + EC2 ✅

| Component | DEV | STG | PROD | Status |
|---|---|---|---|---|
| ALB | aa-alb-dev · active | aa-alb-stg · active | aa-alb-prod · active | ✅ |
| ALB DNS DEV | aa-alb-dev-1308152565.ap-southeast-1.elb.amazonaws.com | — | — | ✅ |
| Target Group | aa-tg-dev-api /api/health | aa-tg-stg-api | aa-tg-prod-api | ✅ |
| EC2 Instance | i-09556545ff6dcf169 t3.micro | i-08ead6ff0e69560dd t3.small | i-0d010f6aa4c5f467b t3.medium | ✅ |
| Public IP | None | None | None | ✅ |
| Target Health | **healthy** ✅ | unhealthy (app pending) | unhealthy | ⚠️ |

---

## SECTION 6 — APPLICATION DEPLOYMENT

### DEV ✅ Complete

| Component | Value | Status |
|---|---|---|
| PHP 8.2.30 | Installed via SSM | ✅ |
| Nginx | Running | ✅ |
| Composer 2.9.5 | Installed | ✅ |
| Laravel 12 | Deployed | ✅ |
| /api/health | {"status":"ok","env":"local"} | ✅ |
| Secrets Manager connection | aa/dev/db + aa/dev/redis | ✅ |
| DB Migrations | users + cache + jobs tables | ✅ |
| CloudWatch Agent | active | ✅ |
| Git repo | develop branch tracking | ✅ |

### STG ⚠️ Infrastructure ready, app pending

| Component | Status | Note |
|---|---|---|
| NAT Instance | ✅ Running | IP forward enabled |
| SSM Connectivity | ❌ Pending | Ubuntu 22.04 SSM agent issue |
| Runtime (PHP/Nginx) | ⏳ | Blocked by SSM |
| Laravel deploy | ⏳ | Blocked by SSM |

---

## SECTION 7 — DATA LAYER

### DEV ✅

| Component | Value | Status |
|---|---|---|
| RDS aa-dev-db | aa-dev-db.cjye026a0zfr.ap-southeast-1.rds.amazonaws.com | ✅ |
| RDS Engine | PostgreSQL 15.14 | ✅ |
| RDS Backup | 1 day | ✅ |
| RDS Public Access | No | ✅ |
| Redis aa-dev-redis | aa-dev-redis.mtg7aw.ng.0001.apse1.cache.amazonaws.com | ✅ |
| S3 aa-dev-media | aa-dev-media-710590321660 · Block public | ✅ |
| S3 aa-dev-deploy | aa-dev-deploy-710590321660 | ✅ |
| Secret aa/dev/db | host+port+dbname+user+password | ✅ |
| Secret aa/dev/redis | host+port+url | ✅ |
| Secret aa/dev/jwt | secret_key+ttls | ✅ |

### STG ⏳ (Infrastructure ready, pending app deployment)
### PROD ⏳ (Not started)

---

## SECTION 8 — CLOUDWATCH ✅ (DEV)

| Component | Value | Status |
|---|---|---|
| Log Group /aa/dev/laravel | 7 days retention | ✅ |
| Log Group /aa/dev/nginx | 7 days retention | ✅ |
| CloudWatch Agent | active on EC2 DEV | ✅ |
| SNS aa-dev-alerts | pqnghiep1354@gmail.com · confirmed | ✅ |
| Alarm aa-dev-alb-5xx | >10 per 5min → OK | ✅ |
| Alarm aa-dev-alb-unhealthy | ≥1 → OK | ✅ |
| Alarm aa-dev-ec2-cpu | >80% → OK | ✅ |
| Alarm aa-dev-ec2-memory | >85% → OK | ✅ |
| Alarm aa-dev-rds-cpu | >75% → OK | ✅ |
| Alarm aa-dev-rds-storage | <5GB → OK | ✅ |
| Alarm aa-dev-redis-evictions | >0 **(P0)** → OK | ✅ |
| Dashboard AdventureAsia-DEV | 4 widgets | ✅ |

---

## SECTION 9 — CI/CD PIPELINE ✅

| Component | Value | Status |
|---|---|---|
| GitHub repo | pqnghiep1354/adventure-asia | ✅ |
| Branch develop | Auto-deploy DEV | ✅ |
| Branch main | PROD protected | ✅ |
| OIDC Authentication | No static AWS keys | ✅ |
| GitHub Env: staging | No protection | ✅ |
| GitHub Env: production | Required reviewers | ✅ |
| Run Tests job | PHPUnit pass | ✅ |
| Deploy to DEV job | SSM + S3 script pattern | ✅ |
| Health check | HTTP 200 | ✅ |
| Last successful run | #7 fix: add php opening tag | ✅ |

---

## KEY VARIABLES REFERENCE

```bash
# Accounts
AA_DEV=710590321660  AA_STG=593110023608  AA_PROD=007050358335

# VPCs
VPC_DEV=vpc-08930f8f35962726e
VPC_STG=vpc-0c6397e17adf02007
VPC_PROD=vpc-0c527df6ff7cd2ca9

# ALB DNS
ALB_DNS_DEV=aa-alb-dev-1308152565.ap-southeast-1.elb.amazonaws.com

# EC2
INSTANCE_ID_DEV=i-09556545ff6dcf169
INSTANCE_ID_STG=i-08ead6ff0e69560dd
INSTANCE_ID_PROD=i-0d010f6aa4c5f467b

# RDS DEV
RDS_DEV=aa-dev-db.cjye026a0zfr.ap-southeast-1.rds.amazonaws.com

# Redis DEV
REDIS_DEV=aa-dev-redis.mtg7aw.ng.0001.apse1.cache.amazonaws.com

# S3
S3_DEPLOY_DEV=aa-dev-deploy-710590321660

# GitHub
REPO=pqnghiep1354/adventure-asia
```

---

*Component Build Table v5 · 18/03/2026 · Prepared by Nghiep*
