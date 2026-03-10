# Architecture-to-DevOps Component Map — Adventure Asia App

**Prepared by:** Nghiep | **Mentor:** Shivam (QuanSkill) | **Date:** 10/03/2026

---

## System Overview

```
Flutter Mobile App          React Admin Dashboard
(iOS + Android)             (Web Browser)
       │                           │
       └──────────┬────────────────┘
                  │ HTTPS + JWT / WebSocket
                  ▼
            Nginx (Reverse Proxy + SSL)
                  │
                  ▼
          Laravel REST API (PHP)
          JWT Auth · RBAC Middleware
          ┌────────┬─────────────┐
          ▼        ▼             ▼
      PostgreSQL  Redis         S3
        (RDS)  (ElastiCache)  (Storage)
```

---

## Component Map

| Component                            | AWS Service             | DevOps Responsibility                                                                  |
| ------------------------------------ | ----------------------- | -------------------------------------------------------------------------------------- |
| **Mobile App** (Flutter iOS/Android) | — client-side           | Build artifacts, App Store / Play Store deployment pipeline                            |
| **Admin Dashboard** (React)          | S3 + CloudFront         | Build static assets, deploy to S3, CDN cache invalidation on release                   |
| **Backend API** (Laravel / PHP)      | EC2                     | Deploy via GitHub Actions, health check monitoring, restart on failure, scaling        |
| **Database** (PostgreSQL 15)         | RDS                     | Backup schedule, migration safety, connection monitoring, access control               |
| **Cache + Queue** (Redis)            | ElastiCache             | Sizing, hit rate monitoring, queue worker health, flush on demand                      |
| **File Storage**                     | S3                      | Bucket policies, Block Public Access, versioning, lifecycle rules                      |
| **SSL / HTTPS**                      | ACM + ALB               | Certificate renewal, HTTPS enforcement, routing rules                                  |
| **Firewall**                         | Security Groups         | Inbound/outbound rule audit — `aa-sg-app` (443/80/22) · `aa-sg-db` (5432 from SG only) |
| **Secrets**                          | Secrets Manager         | Rotation schedule, IAM access policies, audit trail — naming: `aa/[env]/[purpose]`     |
| **CI/CD Pipeline**                   | GitHub Actions          | Workflow maintenance, secret injection, manual approval gate for Production            |
| **Metrics**                          | CloudWatch Metrics      | Dashboard, alarm thresholds — CPU, DB connections, storage, error rate                 |
| **Logs**                             | CloudWatch Logs         | Log group management, retention policies, Log Insights queries for incidents           |
| **Alerts**                           | CloudWatch Alarms + SNS | Alarm thresholds, SNS topic `aa-[env]-alerts`, email/Slack notification path           |
| **Access Control**                   | IAM                     | Role/policy review, least-privilege audit, MFA enforcement                             |
| **Cost Tracking**                    | AWS Budgets             | Budget alerts, monthly review, cleanup after practice sessions                         |
| **DNS & Routing**                    | Route 53                | Domain routing per environment, health check failover                                  |

---

## Key Monitoring Targets (Production)

| Metric              | Service         | Threshold             | Action                                   |
| ------------------- | --------------- | --------------------- | ---------------------------------------- |
| CPUUtilization      | EC2             | > 70% for 15 min      | Investigate load, consider scaling       |
| DatabaseConnections | RDS             | > 70 (≈80% of max)    | Check connection pool, query performance |
| FreeStorageSpace    | RDS             | < 5 GB                | Expand storage or archive old data       |
| HTTP 5xx error rate | CloudWatch Logs | Spike > 200% baseline | Check logs, consider rollback            |
| EstimatedCharges    | Billing         | > 80% of budget       | Review and clean up idle resources       |

---


