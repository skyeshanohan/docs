### GitHub Repository Controls — Leadership Brief (Slides)

**Purpose**  
Equip senior leaders with the decisions, guardrails, and impacts in 6–10 concise slides. This is a condensed, slide-ready version of the one-pager.

---

### 1. Why this matters
- **Risk**: Inconsistent repo settings → drift, audit gaps, slower incident response.
- **Goal**: Standardize controls via policy‑as‑code to reduce risk without slowing delivery.

---

### 2. What leaders decided
- Central policy repo owns rulesets and security defaults.
- PRs required to the default branch; approvals and CODEOWNERS reviews enforced.
- SAST blocks only High/Critical; SCA is report‑only.

---

### 3. At‑a‑glance controls

| Area | Decision/Default | Merge gate? | Owner |
| --- | --- | --- | --- |
| Branch protections | PRs, ≥1 non‑committer approval, CODEOWNERS review, no force push/delete | Yes | Security/Infra |
| CODEOWNERS | 100% path coverage; wildcard fallback | Yes | Security/Infra |
| Compliance Settings Check | Required PR check validates Service ID + security features | Yes | Central GitHub App |
| SAST | Runs on PRs; blocks High/Critical | Yes | Policy workflows |
| SCA | Runs on PRs + nightly; report‑only | No | Policy workflows |
| Secrets/Push Protection | Enabled; bypass w/ justification; alerts to Slack | Yes (bypassable) | Security/Infra |

---

### 4. Delivery impact
- Teams merge normally; only High/Critical SAST issues block.
- Onboarding window: 5 days from repo creation; merges blocked if incomplete after window.
- Bypasses are visible to leadership channels; no silent exceptions.

---

### 5. Operating model
- Changes to protections/settings happen via PRs in the policy repo.
- App‑backed reconciler enforces rules and detects drift.
- Emergency kill switches: read‑only enforcement; pause archival/deletes.

---

### 6. Options

| Option | Summary | Tradeoff |
| --- | --- | --- |
| A — Centralized policy‑as‑code (recommended) | Highest consistency and evidence | Higher initial setup |
| B — Org‑level rulesets + templates | Lower lift | Higher drift, weaker evidence |
| C — Repo‑local autonomy | Highest autonomy | Inconsistent enforcement, weak auditability |

---

### 7. What leaders need to know
- Central ownership; auditable via PRs.
- Fast path for delivery; strong path for compliance.
- Clear escalation: Security/Infra owns policy changes.

---

### 8. FAQs (condensed)
- Will this slow merges? No — SAST blocks only High/Critical; SCA is report‑only.
- Who can change thresholds or protections? Only Security/Infra via PRs.
- Why not auto‑archive for onboarding gaps? Blocking merges keeps remediation fast.

---

References: See `github-repository-controls-one-pager.md` for full detail.

