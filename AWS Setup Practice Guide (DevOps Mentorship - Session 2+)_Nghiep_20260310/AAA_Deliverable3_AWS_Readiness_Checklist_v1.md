# AWS Readiness Checklist v1 — Adventure Asia App

**Prepared by:** Nghiep | **Mentor:** Shivam (QuanSkill) | **Date:** DD/MM/YYYY
**Account:** Demo / Practice | **Region:** `ap-southeast-1` | **IAM User:** `admin_nghiep`

**Legend:** ✅ Done · 🟡 Documented / pending infrastructure · ❌ Gap — action required

---

## 1. Access & IAM

| # | Item | Status | Notes |
|---|---|---|---|
| 1.1 | Root account MFA enabled | ❌ Unconfirmed | Must verify at next session — highest priority |
| 1.2 | Root account not used for daily work | ✅ | `admin_nghiep` IAM user used for all operations |
| 1.3 | IAM user MFA enabled | ✅ | Device `admin_nghiep-mfa` assigned and tested → Screenshot 1.1 |
| 1.4 | IAM group `aa-devops` with least-privilege policies | ✅ | 4 ReadOnly policies: EC2, RDS, S3, CloudWatch → Screenshot 1.2 |
| 1.5 | EC2 IAM Roles created (no static access keys) | ✅ | `aa-role-readonly`, `aa-role-deploy-nonprod` → Screenshot 1.3 |
| 1.6 | Production Access Rules documented | ✅ | 5 rules — approvals, DB access, secrets, emergency logging |

---

## 2. Environment Separation

| # | Item | Status | Notes |
|---|---|---|---|
| 2.1 | All resources named `aa-[env]-[component]` | ✅ | Applied across all steps |
| 2.2 | All resources tagged (Project / Env / Owner / Component) | ✅ | Tagging standard defined in Step 0 |
| 2.3 | Security Groups environment-specific | ✅ | `aa-sg-app`, `aa-sg-db` → Screenshots 3.1, 3.2 |
| 2.4 | S3 bucket environment-specific | ✅ | `aa-dev-media-nghiep` → Screenshot 4.1 |
| 2.5 | Secrets namespaced per environment (`aa/dev/*`) | ✅ | Pattern established in Secrets Manager → Screenshot 6.1 |
| 2.6 | Environment Plan documented | ✅ | See Deliverable 2 — Environment Plan v1 |

---

## 3. Network Security

| # | Item | Status | Notes |
|---|---|---|---|
| 3.1 | `aa-sg-app`: HTTPS 443 open to `0.0.0.0/0` | ✅ | → Screenshot 3.1 |
| 3.2 | `aa-sg-app`: HTTP 80 open to `0.0.0.0/0` (redirect) | ✅ | → Screenshot 3.1 |
| 3.3 | `aa-sg-app`: SSH 22 restricted to my IP only | ✅ | Not `0.0.0.0/0` → Screenshot 3.1 |
| 3.4 | `aa-sg-db`: PostgreSQL 5432 from `aa-sg-app` SG only | ✅ | Source is SG ID, not IP range → Screenshot 3.2 |
| 3.5 | RDS `Public accessibility = No` | ✅ | Confirmed in DB Setup Checklist (Step 5) |
| 3.6 | S3 Block Public Access — all 4 settings enabled | ✅ | → Screenshot 4.1 |

---

## 4. Secrets & Configuration

| # | Item | Status | Notes |
|---|---|---|---|
| 4.1 | Secrets stored in AWS Secrets Manager | ✅ | `aa/dev/db_credentials` created → Screenshot 6.1 |
| 4.2 | No secrets in `.env` files, code, or chat | ✅ | Rule documented; `.gitignore` pattern established |
| 4.3 | Rotation plan documented | ✅ | Disabled for practice; Lambda rotation planned for production |
| 4.4 | Secrets Handling Rules written (5 rules) | ✅ | Covers Git commits, communication, rotation response |

---

## 5. Cost & Monitoring

| # | Item | Status | Notes |
|---|---|---|---|
| 5.1 | Monthly cost budget with alert thresholds | ✅ | `aa-practice-monthly`: alert at $5 (50%) and $10 (100%) → Screenshot 2.1 |
| 5.2 | CloudWatch Billing Alerts enabled | ✅ | Activated in Billing Preferences (root account) |
| 5.3 | SNS topic created with confirmed subscription | ✅ | `aa-dev-alerts` — email confirmed → Screenshot 7.2 |
| 5.4 | At least one CloudWatch alarm created | ✅ | `aa-practice-billing-alert`: EstimatedCharges > $8 → Screenshot 7.1 |
| 5.5 | EC2 CPU alarm (`CPUUtilization > 70%`, 3/3 periods) | 🟡 | Config documented in Step 7 — pending EC2 instance |
| 5.6 | RDS Connections alarm (`DatabaseConnections > 70`) | 🟡 | Config documented in Step 7 — pending RDS instance |
| 5.7 | Cleanup checklist followed after each session | ✅ | 7-item checklist in Step 2; RDS deleted after practice |

---

## 6. Operational Readiness

| # | Item | Status | Notes |
|---|---|---|---|
| 6.1 | Safe Release SOP documented | ✅ | 8-step SOP with rollback rules and communication templates (Step 8) |
| 6.2 | Change Log template ready | ✅ | Table with example entry (Step 8) |
| 6.3 | Architecture-to-DevOps Component Map completed | ✅ | 16 components mapped (Step 8) |
| 6.4 | Post-Deployment Verification Checklist (8 steps) | ✅ | Covers pipeline → health check → logs → metrics → smoke tests (Step 7) |

---

## Summary

| Area | Done | Total | Status |
|---|---|---|---|
| Access & IAM | 5 | 6 | 🟡 Root MFA unconfirmed |
| Environment Separation | 6 | 6 | ✅ |
| Network Security | 6 | 6 | ✅ |
| Secrets & Configuration | 4 | 4 | ✅ |
| Cost & Monitoring | 5 | 7 | 🟡 EC2/RDS alarms pending infrastructure |
| Operational Readiness | 4 | 4 | ✅ |
| **Total** | **30** | **33** | **🟡 91% — 3 items pending** |

**Outstanding actions:**
1. ❌ Confirm and enable MFA on root account (highest priority)
2. 🟡 Create EC2 CPU alarm once EC2 is provisioned
3. 🟡 Create RDS Connections alarm once RDS is available

---

*Adventure Asia App — Deliverable 3: AWS Readiness Checklist v1 | Prepared for review by Shivam (QuanSkill)*
