# Adventure Asia (AA) — AWS DevOps Practice Report

# Final Submission · Pham Quoc Nghiep

**Date:** 18/03/2026
**Mentee:** Pham Quoc Nghiep
**Mentor:** Shivam (QuanSkill)
**Guide Reference:** AA AWS DevOps Setup Guide · 15/03/2026

---

## EXECUTIVE SUMMARY

Practice completed for Steps 1–9 (DEV environment fully operational end-to-end). Steps 1–6 verified across all 3 environments at infrastructure level. CI/CD pipeline running with OIDC authentication — no static AWS keys anywhere.

**DEV:** 100% complete — Laravel API deployed, DB connected, monitoring active, CI/CD pipeline working.
**STG:** Infrastructure complete (VPC/SGs/NAT/EC2/ALB). Application deployment blocked by SSM agent connectivity issue on Ubuntu 22.04 (to be resolved).
**PROD:** Infrastructure complete (VPC/SGs/EC2/ALB). Application deployment pending.

---

## §0 — BIG PICTURE DECISIONS ✅

**Compute Strategy:** Phase 1 — Laravel API on EC2 behind ALB (implemented). Phase 2 — ECS Fargate (planned). Infrastructure designed migration-friendly: VPC, SGs, ALB, Target Groups all compatible with ECS.


---

## §1 — ENVIRONMENTS & ISOLATION ✅

Implemented **Option A (Best Practice)** — 4 separate AWS accounts:

| Account    | ID           | Purpose                     |
| ---------- | ------------ | --------------------------- |
| Management | 867490540162 | Organizations, SSO, billing |
| AA-DEV     | 710590321660 | Development                 |
| AA-STAGING | 593110023608 | Pre-production testing      |
| AA-PROD    | 007050358335 | Production                  |

**Verification:** Resources clearly identified by `aa-[env]-[component]` naming. SCP locks all accounts to ap-southeast-1. PROD isolated in separate OU.

> 📸 **[Screenshot 1.1]** — AWS Organizations structure
>
> ![1773838612981](image/AAA_Practice_Report_Final/1773838612981.png)
> 📸 **[Screenshot 1.2]** — 3 member accounts created
>
> ![1773838680105](image/AAA_Practice_Report_Final/1773838680105.png)
> 📸 **[Screenshot 1.3]** — Organizational Units (Non-Production + Production)
>
> ![1773838697343](image/AAA_Practice_Report_Final/1773838697343.png)
> 📸 **[Screenshot 1.4]** — Service Control Policy RegionLock
>
> ![1773838703658](image/AAA_Practice_Report_Final/1773838703658.png)

---

## §2 — ACCESS MODEL: SSO + ROLES ✅

**IAM Identity Center** configured at ap-southeast-1. No long-lived access keys for humans — SSO only with temporary credentials.

| Permission Set | Session | Purpose                                                                                                                           |
| -------------- | ------- | --------------------------------------------------------------------------------------------------------------------------------- |
| AA-Admin       | 1 hour  | Break-glass emergency only                                                                                                        |
| AA-DevOps      | 8 hours | Daily infra changes                                                                                                               |
| AA-ReadOnly    | 8 hours | Read-only + CW Logs for developers (=AA-Developer-Read per guide — AWS does not support renaming Permission Sets after creation) |

**MFA:** Enforced for every sign-in (TOTP).
**CI/CD:** AA-CICD role via OIDC — zero static AWS keys.

**Verification:** DevOps can deploy infra ✅ · ReadOnly cannot modify ✅ · CI/CD no static keys ✅

> 📸 **[Screenshot 1.5]** — IAM Identity Center setup
>
> ![1773838917445](image/AAA_Practice_Report_Final/1773838917445.png)
> 📸 **[Screenshot 1.6]** — Permission Sets + user assignments
>
> ![1773840396232](image/AAA_Practice_Report_Final/1773840396232.png)
> 📸 **[Screenshot 1.7]** — AWS CLI SSO login working
>
> ![1773838943560](image/AAA_Practice_Report_Final/1773838943560.png)
> 📸 **[Screenshot 1.8]** — sts get-caller-identity output
>
> ![1773838950756](image/AAA_Practice_Report_Final/1773838950756.png)
> 📸 **[Screenshot 2.1]** — MFA enforcement enabled
>
> ![1773838967804](image/AAA_Practice_Report_Final/1773838967804.png)
> 📸 **[Screenshot 2.2]** — MFA device registered
>
> ![1773838975811](image/AAA_Practice_Report_Final/1773838975811.png)
> 📸 **[Screenshot 3.1~3.18]** — IAM roles for DEV/STG/PROD + OIDC providers
>
> ![1773838993018](image/AAA_Practice_Report_Final/1773838993018.png)
>
> ![1773838997742](image/AAA_Practice_Report_Final/1773838997742.png)
>
> ![1773839004478](image/AAA_Practice_Report_Final/1773839004478.png)
>
> ![1773839008676](image/AAA_Practice_Report_Final/1773839008676.png)
>
> ![1773839012870](image/AAA_Practice_Report_Final/1773839012870.png)![1773839015970](image/AAA_Practice_Report_Final/1773839015970.png)![1773839019495](image/AAA_Practice_Report_Final/1773839019495.png)![1773839022630](image/AAA_Practice_Report_Final/1773839022630.png)
>
> ![1773839041828](image/AAA_Practice_Report_Final/1773839041828.png)
>
> ![1773839046706](image/AAA_Practice_Report_Final/1773839046706.png)
>
> ![1773839052256](image/AAA_Practice_Report_Final/1773839052256.png)
>
> ![1773839060581](image/AAA_Practice_Report_Final/1773839060581.png)
>
> ![1773839067821](image/AAA_Practice_Report_Final/1773839067821.png)
>
> ![1773839071720](image/AAA_Practice_Report_Final/1773839071720.png)
>
> ![1773839077945](image/AAA_Practice_Report_Final/1773839077945.png)
>
> ![1773839084110](image/AAA_Practice_Report_Final/1773839084110.png)
>
> ![1773839088909](image/AAA_Practice_Report_Final/1773839088909.png)
>
> ![1773839092987](image/AAA_Practice_Report_Final/1773839092987.png)
>

---

## §3 — NETWORK BASELINE ✅

3 VPCs (one per environment), 2 AZs each, full public/private subnet separation:

| VPC         | CIDR        | Public Subnets            | Private Subnets             |
| ----------- | ----------- | ------------------------- | --------------------------- |
| aa-dev-vpc  | 10.0.0.0/16 | 10.0.1.0/24 + 10.0.2.0/24 | 10.0.11.0/24 + 10.0.12.0/24 |
| aa-stg-vpc  | 10.1.0.0/16 | 10.1.1.0/24 + 10.1.2.0/24 | 10.1.11.0/24 + 10.1.12.0/24 |
| aa-prod-vpc | 10.2.0.0/16 | 10.2.1.0/24 + 10.2.2.0/24 | 10.2.11.0/24 + 10.2.12.0/24 |

**Security Group chain (all 3 environments):**

- `aa-sg-alb`: inbound 80+443 from 0.0.0.0/0
- `aa-sg-app`: inbound 80 from aa-sg-alb SG only (no public inbound)
- `aa-sg-db`: inbound 5432 from aa-sg-app SG only
- `aa-sg-redis`: inbound 6379 from aa-sg-app SG only

**NAT:** NAT Instance (t2.micro, free tier) for DEV and STG. NAT Gateway planned for PROD go-live.

**Verification:** Only ALB is public ✅ · EC2 has no public IP ✅ · RDS/Redis not publicly reachable ✅

> 📸 **[Screenshot 4.1]** — aa-dev-vpc detail
>
> ![1773839187854](image/AAA_Practice_Report_Final/1773839187854.png)
> 📸 **[Screenshot 4.2]** — aa-dev-vpc resource map
>
> ![1773839194900](image/AAA_Practice_Report_Final/1773839194900.png)
> 📸 **[Screenshot 4.3]** — DEV aa-sg-alb inbound rules
>
> ![1773839201699](image/AAA_Practice_Report_Final/1773839201699.png)
> 📸 **[Screenshot 4.4]** — DEV aa-sg-app inbound rules (from SG ID only)
>
> ![1773839209476](image/AAA_Practice_Report_Final/1773839209476.png)
> 📸 **[Screenshot 4.5]** — DEV aa-sg-db inbound rules
>
> ![1773839226652](image/AAA_Practice_Report_Final/1773839226652.png)
> 📸 **[Screenshot 4.6]** — DEV aa-sg-redis inbound rules
>
> ![1773839239025](image/AAA_Practice_Report_Final/1773839239025.png)
> 📸 **[Screenshot 4.7~4.12]** — STG VPC + SGs
>
> ![1773839245232](image/AAA_Practice_Report_Final/1773839245232.png)
>
> ![1773839248597](image/AAA_Practice_Report_Final/1773839248597.png)
>
> ![1773839254111](image/AAA_Practice_Report_Final/1773839254111.png)
>
> ![1773839259324](image/AAA_Practice_Report_Final/1773839259324.png)
>
> ![1773839263509](image/AAA_Practice_Report_Final/1773839263509.png)
>
> ![1773839267915](image/AAA_Practice_Report_Final/1773839267915.png)
>
>
> 📸 **[Screenshot 4.13~4.18]** — PROD VPC + SGs
>
> ![1773839277650](image/AAA_Practice_Report_Final/1773839277650.png)
>
> ![1773839282833](image/AAA_Practice_Report_Final/1773839282833.png)
>
> ![1773839287765](image/AAA_Practice_Report_Final/1773839287765.png)
>
> ![1773839296384](image/AAA_Practice_Report_Final/1773839296384.png)
>
> ![1773839311135](image/AAA_Practice_Report_Final/1773839311135.png)
>
> ![1773839315488](image/AAA_Practice_Report_Final/1773839315488.png)
>
>
> 📸 **[Screenshot 4.19]** — Terminal verify all 3 VPCs + 12 SGs
>
> ![1773839321918](image/AAA_Practice_Report_Final/1773839321918.png)

---

## §4 — LOAD BALANCER & HTTPS ⚠️

**ALBs** deployed across all 3 environments (internet-facing, public subnets).

| ALB         | DNS                                                    | Status |
| ----------- | ------------------------------------------------------ | ------ |
| aa-alb-dev  | aa-alb-dev-1308152565.ap-southeast-1.elb.amazonaws.com | active |
| aa-alb-stg  | aa-alb-stg-1752496847.ap-southeast-1.elb.amazonaws.com | active |
| aa-alb-prod | aa-alb-prod-203362903.ap-southeast-1.elb.amazonaws.com | active |

**Health check:** `/api/health` — DEV target **healthy** ✅

**Note on HTTPS:** ACM certificate and HTTPS listener (port 443) not yet configured — no domain registered. Confirmed with Shivam (17/03/2026): HTTP acceptable for DEV/STG at practice stage. HTTPS will be added for PROD when domain is confirmed.

**Current:** HTTP:80 → forward to Target Group (temporary)
**Required for PROD:** HTTP:80 → redirect 301 → HTTPS:443 + ACM certificate

**Verification:** Health check shows DEV healthy ✅ · Listener configured ✅ · HTTPS pending domain ⚠️

> 📸 **[Screenshot 5.1]** — aa-alb-dev detail
>
> ![1773839564850](image/AAA_Practice_Report_Final/1773839564850.png)
> 📸 **[Screenshot 5.2]** — ALB DEV listener configuration
>
> ![1773839570728](image/AAA_Practice_Report_Final/1773839570728.png)
> 📸 **[Screenshot 5.3]** — ALB DEV target group (/api/health)
>
> ![1773839578645](image/AAA_Practice_Report_Final/1773839578645.png)
> 📸 **[Screenshot 5.4]** — EC2 DEV instance detail (no public IP)
>
> ![1773839592629](image/AAA_Practice_Report_Final/1773839592629.png)
> 📸 **[Screenshot 5.5]** — aa-alb-stg
>
> ![1773839600333](image/AAA_Practice_Report_Final/1773839600333.png)
> 📸 **[Screenshot 5.6~5.8]** — STG ALB + TG + EC2
>
> ![1773839607073](image/AAA_Practice_Report_Final/1773839607073.png)
>
> ![1773839611932](image/AAA_Practice_Report_Final/1773839611932.png)
>
> ![1773839616916](image/AAA_Practice_Report_Final/1773839616916.png)
> 📸 **[Screenshot 5.9~5.12]** — PROD ALB + TG + EC2
>
> ![1773839647165](image/AAA_Practice_Report_Final/1773839647165.png)
>
> ![1773839651959](image/AAA_Practice_Report_Final/1773839651959.png)
>
> ![1773839672145](image/AAA_Practice_Report_Final/1773839672145.png)![1773839677491](image/AAA_Practice_Report_Final/1773839677491.png)
>
>
> 📸 **[Screenshot 5.13]** — Terminal verify all 3 ALBs + EC2s healthy
>
> ![1773839687116](image/AAA_Practice_Report_Final/1773839687116.png)

---

## §5 — DATA LAYER ✅ (DEV) / ⏳ (STG/PROD)

### RDS PostgreSQL

| Instance   | Env  | Endpoint                  | Backup  | Public | Status |
| ---------- | ---- | ------------------------- | ------- | ------ | ------ |
| aa-dev-db  | DEV  | aa-dev-db.cjye026a0zfr... | 1 day   | No     | ✅     |
| aa-stg-db  | STG  | ⏳ pending                | 7 days  | No     | ⏳     |
| aa-prod-db | PROD | ⏳ pending                | 30 days | No     | ⏳     |

### ElastiCache Redis

| Instance     | Env | Endpoint               | Encryption | Status |
| ------------ | --- | ---------------------- | ---------- | ------ |
| aa-dev-redis | DEV | aa-dev-redis.mtg7aw... | Off (DEV)  | ✅     |
| aa-stg-redis | STG | ⏳ pending             | Off (STG)  | ⏳     |

**Note on encryption:** `StorageEncrypted: false` for DEV — confirmed with Shivam as acceptable for practice stage. PROD will have encryption at rest mandatory.

**Verification:** DB not publicly reachable ✅ · Backup policies set ✅ · Private subnet placement ✅

> 📸 **[Screenshot 7.1]** — RDS aa-dev-db: status available, no public access
>
> ![1773839796982](image/AAA_Practice_Report_Final/1773839796982.png)
> 📸 **[Screenshot 7.2]** — RDS maintenance & backup (1 day retention)
>
> ![1773839802954](image/AAA_Practice_Report_Final/1773839802954.png)
> 📸 **[Screenshot 7.3]** — Deletion protection (disabled for DEV)
>
> ![1773839808658](image/AAA_Practice_Report_Final/1773839808658.png)
> 📸 **[Screenshot 7.4]** — ElastiCache aa-dev-redis: available
>
> ![1773839815019](image/AAA_Practice_Report_Final/1773839815019.png)
> 📸 **[Screenshot 7.5]** — S3 aa-dev-media: Block all public access ON
>
> ![1773839824203](image/AAA_Practice_Report_Final/1773839824203.png)

---

## §6 — APPLICATION DEPLOYMENT ✅ (DEV)

**EC2 DEV** fully operational:

- Ubuntu 22.04, PHP 8.2.30, Nginx, Composer 2.9.5
- Laravel 12 deployed with OIDC-based CI/CD
- Environment variables fetched from Secrets Manager at runtime (no hardcoded secrets)
- Database migrations complete (users, cache, jobs tables)

```bash
curl http://aa-alb-dev-1308152565.ap-southeast-1.elb.amazonaws.com/api/health
→ {"status":"ok","env":"local"}
```

**ALB Target:** healthy ✅

**Secrets Manager pattern:**

- `aa/dev/db` — RDS credentials
- `aa/dev/redis` — Redis connection
- `aa/dev/jwt` — JWT signing key

**STG:** Infrastructure ready. SSM agent connectivity issue on Ubuntu 22.04 AMI preventing automated deployment. Resolution: manual SSM agent installation via EC2 Instance Connect or next sprint.

**Verification:** API reachable via ALB DNS ✅ · No hardcoded secrets ✅ · Application logs visible in CloudWatch ✅

> 📸 **[Screenshot 6.1]** — NAT Instance running (aa-nat-dev)
>
> ![1773839848655](image/AAA_Practice_Report_Final/1773839848655.png)
> 📸 **[Screenshot 6.2]** — Source/Dest check disabled
>
> ![1773839856650](image/AAA_Practice_Report_Final/1773839856650.png)
> 📸 **[Screenshot 6.3]** — Private route table with NAT route
>
> ![1773839861972](image/AAA_Practice_Report_Final/1773839861972.png)
> 📸 **[Screenshot 6.4]** — SSM Session Manager connected
>
> ![1773839872739](image/AAA_Practice_Report_Final/1773839872739.png)
> 📸 **[Screenshot 6.5]** — SSM EC2 terminal
>
> ![1773839889658](image/AAA_Practice_Report_Final/1773839889658.png)
> 📸 **[Screenshot 6.6]** — PHP 8.2 version on EC2
>
> ![1773839896919](image/AAA_Practice_Report_Final/1773839896919.png)
> 📸 **[Screenshot 6.7]** — curl /api/health → {"status":"ok"}
>
> ![1773839905335](image/AAA_Practice_Report_Final/1773839905335.png)
> 📸 **[Screenshot 6.8]** — ALB Target Group: EC2 DEV Healt
>
> ![1773839935302](image/AAA_Practice_Report_Final/1773839935302.png)
> 📸 **[Screenshot 7.6]** — Secrets Manager: aa/dev/db + redis + jwt
>
> ![1773839947324](image/AAA_Practice_Report_Final/1773839947324.png)
> 📸 **[Screenshot 7.7]** — ALB Target Health: healthy (after DB connect)
>
> ![1773839990224](image/AAA_Practice_Report_Final/1773839990224.png)
> 📸 **[Screenshot 7.8]** — Laravel migrations success + /api/health
>
> ![1773839999735](image/AAA_Practice_Report_Final/1773839999735.png)

---

## §7 — CI/CD: GITHUB ACTIONS OIDC ✅

**Golden rule:** No AWS access keys stored in GitHub. OIDC role assumption only.

**Workflow:** `.github/workflows/deploy.yml`

- **Push to develop** → Run Tests → Deploy to DEV (auto)
- **Release tag** → Deploy to PROD (manual approval required)

**OIDC Flow:**

```
GitHub Actions → request OIDC token → AWS STS verify
→ Assume AA-CICD role (temp credentials, 1h)
→ S3 upload deploy script → SSM SendCommand → EC2
→ Health check HTTP 200
```

**Last successful run:** #7 `fix: add php opening tag to api.php`

- Run Tests: ✅ 25s
- Deploy to DEV: ✅ 1m 49s
- Health check: ✅ HTTP 200

**Verification:** Pipeline deploys without static keys ✅ · PROD deploy requires manual approval ✅

> 📸 **[Screenshot 9.1]** — GitHub Environments (staging + production)
>
> ![1773840136022](image/AAA_Practice_Report_Final/1773840136022.png)
> 📸 **[Screenshot 9.2]** — GitHub Secrets (no AWS_ACCESS_KEY)
>
> ![1773840144044](image/AAA_Practice_Report_Final/1773840144044.png)
> 📸 **[Screenshot 9.3]** — GitHub Actions run #7: all green
>
> ![1773840166203](image/AAA_Practice_Report_Final/1773840166203.png)
> 📸 **[Screenshot 9.4]** — Deploy to DEV job details (OIDC + health check 200)
>
> ![1773840172765](image/AAA_Practice_Report_Final/1773840172765.png)
> 📸 **[Screenshot 9.5]** — Terminal: curl health check after deploy
>
> ![1773840183478](image/AAA_Practice_Report_Final/1773840183478.png)

---

## §8 — FILE UPLOAD AT SCALE (Design Ready)

**S3 bucket created:** `aa-dev-media-710590321660` (Block all public access, SSE-S3 encryption).

**Pre-signed URL flow designed** (to be implemented with dev team):

1. Client requests pre-signed URL from Laravel
2. Client uploads directly to S3 (bypass EC2)
3. Laravel stores metadata in DB
4. Async queue processes (resize, scan)

**Verification:** Upload infrastructure ready ✅ · Implementation pending dev team ⏳

---

## §9 — NOTIFICATIONS & WEBSOCKET (Design Ready)

**Design:** Laravel → Redis queue → worker → email/push/WebSocket.

**When to split WebSocket:** When concurrent connections become high or WS failures impact API stability. Not needed at MVP stage.

**Infrastructure ready:** Redis available as queue backend ✅

---

## §10 — OTP POLICY (Design Ready)

**Redis TTL policy designed** (to be implemented with dev team):

- OTP TTL: 300 seconds (5 minutes)
- Max wrong attempts: 5 per OTP → invalidate + block 10-15 min
- Resend cooldown: 60s · max 3 per 10 min per email/IP

**P0 Risk mitigated:** CloudWatch alarm `aa-dev-redis-evictions > 0` configured — alerts immediately if Redis evicts OTP keys.

---

## §11 — MONITORING & OPERATIONS ✅

**CloudWatch setup complete for DEV:**

| Component        | Value                                               |
| ---------------- | --------------------------------------------------- |
| Log Groups       | /aa/dev/laravel + /aa/dev/nginx (7 day retention)   |
| CloudWatch Agent | Memory + Disk metrics (custom namespace)            |
| SNS Topic        | aa-dev-alerts → pqnghiep1354@gmail.com (confirmed) |
| Alarms           | 7 alarms, all state: OK                             |
| Dashboard        | AdventureAsia-DEV (4 widgets)                       |

**Key alarm:** `aa-dev-redis-evictions` (P0) — triggers immediately on any Redis eviction to protect OTP keys.

**Verification:** Can detect failures quickly ✅ · Can trace errors ALB → logs → DB/Redis ✅

> 📸 **[Screenshot 8.1]** — CloudWatch Log Groups
>
> ![1773840229689](image/AAA_Practice_Report_Final/1773840229689.png)
> 📸 **[Screenshot 8.2]** — Log Streams
>
> ![1773840234838](image/AAA_Practice_Report_Final/1773840234838.png)
> 📸 **[Screenshot 8.3]** — Log Streams entries (data flowing)
>
> ![1773840240044](image/AAA_Practice_Report_Final/1773840240044.png)
> 📸 **[Screenshot 8.4]** — SNS Subscription confirmed
>
> ![1773840245306](image/AAA_Practice_Report_Final/1773840245306.png)
> 📸 **[Screenshot 8.5]** — CloudWatch Alarms (7 alarms, all OK)
>
> ![1773840250524](image/AAA_Practice_Report_Final/1773840250524.png)
> 📸 **[Screenshot 8.6]** — Dashboard AdventureAsia-DEV
>
> ![1773840255373](image/AAA_Practice_Report_Final/1773840255373.png)
> 📸 **[Screenshot 8.7]** — Terminal verify output
>
> ![1773840261417](image/AAA_Practice_Report_Final/1773840261417.png)

---

## §12 — MICROSERVICES SPLIT CRITERIA

**Not needed at current stage.** Will split only when:

- Independent scaling pressure detected
- WS/media has different runtime profile
- Deployment independence required
- Reliability isolation needed

**Practical order if needed:** Upload pipeline + Notification/WS before Auth.

---

## BLOCKERS & KNOWN ISSUES

| # | Issue                               | Step    | Root Cause                                              | Resolution                                               |
| - | ----------------------------------- | ------- | ------------------------------------------------------- | -------------------------------------------------------- |
| 1 | SSM agent not connecting on STG EC2 | Step 10 | Ubuntu 22.04 AMI requires manual SSM agent installation | Next sprint: install via EC2 Instance Connect            |
| 2 | HTTPS/ACM pending                   | §4     | No domain registered                                    | When domain confirmed — add ACM + listener 443          |
| 3 | AA-ReadOnly rename                  | §2     | AWS does not support renaming Permission Sets           | Documented as functional equivalent of AA-Developer-Read |
| 4 | RDS encryption DEV                  | §5     | Confirmed acceptable with Shivam for practice           | Enable for PROD mandatory                                |

---

## PENDING QUESTIONS

1. **STG SSM:** Best approach for installing SSM agent on existing Ubuntu 22.04 EC2 without public access?
2. **Domain:** When is the domain for Adventure Asia confirmed? Required for ACM + HTTPS.
3. **PROD timeline:** When to create PROD data layer (RDS/Redis)?
4. **Developer onboarding:** When to assign AA-ReadOnly to Manh and Pavitra?

---

## APPENDIX — INFRASTRUCTURE REFERENCE

```
Accounts:
  AA-DEV:     710590321660
  AA-STG:     593110023608
  AA-PROD:    007050358335

SSO Start URL: https://d-9667a69473.awsapps.com/start

VPCs:
  DEV: vpc-08930f8f35962726e (10.0.0.0/16)
  STG: vpc-0c6397e17adf02007 (10.1.0.0/16)
  PROD: vpc-0c527df6ff7cd2ca9 (10.2.0.0/16)

EC2:
  DEV:  i-09556545ff6dcf169 (t3.micro, Ubuntu 22.04, Laravel 12)
  STG:  i-08ead6ff0e69560dd (t3.small)
  PROD: i-0d010f6aa4c5f467b (t3.medium)

ALB:
  DEV: aa-alb-dev-1308152565.ap-southeast-1.elb.amazonaws.com

RDS (DEV):
  aa-dev-db.cjye026a0zfr.ap-southeast-1.rds.amazonaws.com

Redis (DEV):
  aa-dev-redis.mtg7aw.ng.0001.apse1.cache.amazonaws.com

GitHub:
  Repo: pqnghiep1354/adventure-asia
  Last deploy: Run #7 (develop branch) — HTTP 200 ✅
```

---

*Adventure Asia App — AWS DevOps Practice Report Final*
*Prepared by: Pham Quoc Nghiep · 18/03/2026*
*Mentor: Shivam (QuanSkill) · QuanSolution*
