# Safe Release SOP + Change Log Template — Adventure Asia App

**Prepared by:** Nghiep | **Mentor:** Shivam (QuanSkill) | **Date:** 10/03/2026

---

## Safe Release SOP

**Scope:** All code deployments to Staging and Production — API, admin dashboard, infrastructure changes. No exceptions, including urgent hotfixes or one-line changes.

---

### Preconditions (must ALL be true before starting)

- [ ] Automated tests pass in GitHub Actions
- [ ] Change reviewed via Pull Request — at least 1 approver who is not the author
- [ ] Release notes written in plain English (what changed and why)
- [ ] Rollback plan written out specifically — not just "we will roll back if needed"
- [ ] **Production only:** written approval received from Shivam

---

### Steps

| #   | Step                     | Who         | How                                                                                      |
| --- | ------------------------ | ----------- | ---------------------------------------------------------------------------------------- |
| 1   | **Develop**              | Developer   | Create `feature/[name]` branch → Pull Request → merge to `develop` after approval        |
| 2   | **Build & Test**         | Automated   | GitHub Actions: install → lint → tests → build. Any failure = fix first, no skip allowed |
| 3   | **Deploy to Staging**    | Automated   | Pipeline auto-deploys after green build. No manual action needed                         |
| 4   | **Validate Staging**     | DevOps + QA | Run Post-Deployment Checklist. Fix all bugs before proceeding                            |
| 5   | **Request Approval**     | DevOps      | Send to Shivam: PR link, release notes, risk level, rollback plan                        |
| 6   | **Deploy to Production** | DevOps      | Confirm window → notify team → trigger manual gate in GitHub Actions → stay at keyboard  |
| 7   | **Verify Production**    | DevOps      | Run Post-Deployment Checklist. Monitor 30–60 min. Report result to Shivam                |
| 8   | **Document**             | DevOps      | Fill in Change Log. Notify team of outcome                                               |

---

### Rollback Rules

**Roll back IMMEDIATELY if any of these occur:**
- Health check returns non-200 for more than 5 minutes
- Error rate in CloudWatch increases > 200% over pre-deployment baseline
- Any critical user flow fails completely (login, search, booking)
- Shivam requests rollback

**Do NOT roll back for:**
- Minor visual glitches with no functional impact
- Performance slightly slower than baseline → investigate first

---

### Communication Templates

| Moment                   | Message                                                                 |
| ------------------------ | ----------------------------------------------------------------------- |
| Before Production deploy | `"Starting Production deployment — commit [SHA] — [time]"`              |
| After successful deploy  | `"Production deployment complete — all checks passed — [time]"`         |
| After rollback           | `"Production rollback executed — [brief reason] — investigating cause"` |

**Escalate to Shivam immediately for:** any Production incident · deployment failure mid-way · any action affecting live users without precedent · any Production decision you are uncertain about.

---

## Change Log Template

| Field                | Value                  |
| -------------------- | ---------------------- |
| Date & Time          |                        |
| Environment          | Staging / Production   |
| Version / Commit SHA |                        |
| Change Description   |                        |
| Risk Level           | Low / Medium / High    |
| Approved By          |                        |
| Deployed By          |                        |
| Verification Result  | Pass / Fail / Rollback |
| Issues Observed      |                        |
| Rollback Taken       | Yes / No               |
| Rollback Reason      | N/A                    |
| Notes                |                        |

---

### Example Entry

| Field                | Value                                                                  |
| -------------------- | ---------------------------------------------------------------------- |
| Date & Time          | 2025-03-10 14:30 UTC+7                                                 |
| Environment          | Production                                                             |
| Version / Commit SHA | v1.2.3 / abc123def456                                                  |
| Change Description   | Added trip status tracking — 10 new statuses, 1 migration (ADD COLUMN) |
| Risk Level           | Medium                                                                 |
| Approved By          | Shivam                                                                 |
| Deployed By          | Nghiep                                                                 |
| Verification Result  | Pass                                                                   |
| Issues Observed      | Migration took 45 seconds — app response was slow during this window   |
| Rollback Taken       | No                                                                     |
| Notes                | Consider adding migration timeout alert for future large migrations    |

---

