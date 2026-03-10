# Environment Plan v1 — Adventure Asia App

**Prepared by:** Nghiep | **Mentor:** Shivam (QuanSkill) | **Date:** 10/03/2026

---

## Environments

| Environment    | Purpose                                   | Data                                           |
| -------------- | ----------------------------------------- | ---------------------------------------------- |
| **dev**        | Active development and unit testing       | Seed / synthetic data only                     |
| **staging**    | Integration testing, QA sign-off, UAT     | Anonymised copy of production data             |
| **production** | Live system — real customers and bookings | Real customer and transaction data             |
| **demo**       | Sales presentations                       | Curated sample data — never real customer data |

---

## Naming Convention

All AWS resources follow: **`aa-[env]-[component]`**

| Component | dev            | staging        | production      |
| --------- | -------------- | -------------- | --------------- |
| EC2       | `aa-dev-api`   | `aa-stg-api`   | `aa-prod-api`   |
| RDS       | `aa-dev-db`    | `aa-stg-db`    | `aa-prod-db`    |
| S3        | `aa-dev-media` | `aa-stg-media` | `aa-prod-media` |
| Redis     | `aa-dev-redis` | `aa-stg-redis` | `aa-prod-redis` |
| Secrets   | `aa/dev/*`     | `aa/stg/*`     | `aa/prod/*`     |

**Tags on all resources:** `Project=AdventureAsia` · `Env=[env]` · `Owner=Nghiep` · `Component=[component]`

---

## Access Rules

| Rule                      | dev                             | staging                                | production                                             |
| ------------------------- | ------------------------------- | -------------------------------------- | ------------------------------------------------------ |
| Who can deploy            | Any developer, anytime          | CI/CD pipeline only (after tests pass) | Written approval from Shivam required                  |
| Server access             | Key pair, IP-restricted port 22 | Key pair, IP-restricted port 22        | AWS SSM Session Manager only — port 22 closed          |
| Database access           | Direct connection allowed       | Direct connection allowed              | EC2 IAM Role only — no direct human access             |
| Secrets location          | `aa/dev/*` in Secrets Manager   | `aa/stg/*` in Secrets Manager          | `aa/prod/*` in Secrets Manager — never in code or chat |
| Real customer data        | ❌ Never                         | ❌ Never                                | ✅ Only here                                            |
| RDS Multi-AZ              | No                              | No                                     | **Yes**                                                |
| RDS backup retention      | 7 days                          | 7 days                                 | **30 days**                                            |
| Deletion protection (RDS) | No                              | No                                     | **Yes**                                                |

---

## Deployment Flow

```
feature branch → Pull Request (code review) → merge to develop
                                                      │
                                                 Auto-deploy → DEV
                                                      │
                                               Automated tests pass
                                                      │
                                                 Auto-deploy → STAGING
                                                      │
                                         QA sign-off + DevOps verification
                                                      │
                                         Written approval from Shivam
                                                      │
                                           Manual gate → PRODUCTION
```

---

