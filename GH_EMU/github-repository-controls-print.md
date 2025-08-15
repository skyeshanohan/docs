### GitHub Repository Controls — Print‑Friendly One‑Pager

This version mirrors the leadership one‑pager, optimized for printing and narrow margins.

**Executive summary**  
Standardize repository security with enterprise policy‑as‑code, default branch protections, and lifecycle controls. Repository administrators cannot change protected settings; only centralized policy can.

### At‑a‑glance controls

| Area | Decision/Default | Gate | Owner |
| --- | --- | --- | --- |
| Branch protections | PRs; ≥1 non‑committer approval; CODEOWNERS review; no force push/delete | PR + approvals | Security/Infra |
| CODEOWNERS | 100% path coverage; wildcard fallback | CODEOWNERS review | Security/Infra |
| Compliance Settings Check | Required PR check validates Service ID + security features | Required check | Central App |
| SAST | PR runs; blocks High/Critical | Severity gate | Policy workflows |
| SCA | PR + nightly; report‑only | None | Policy workflows |
| Secret Scanning & Push Protection | Enabled; bypass with justification; alerts to Slack | Push Protection (bypassable) | Security/Infra |
| Creation requirements | Service ID; auto PRs enable SAST/SCA; 5‑day onboarding | Merges blocked after 5 days if incomplete | Automation + App |
| Lifecycle & archival | Auto‑archive 365d inactivity or +30d post‑decom; export → delete (90d restore) | N/A | Policy + Automation |

### What leaders need to know
- Central policy repo controls settings via PRs; auditable changes.
- Delivery impact is minimal; only High/Critical SAST blocks.
- Onboarding grace period is 5 days; merges blocked if incomplete.
- Bypasses are visible in leadership channels.

### Implementation options

| Option | Summary | Tradeoff |
| --- | --- | --- |
| A — Centralized policy‑as‑code (recommended) | Best consistency + evidence | Higher initial setup |
| B — Org‑level rulesets + templates | Lower lift | Higher drift; weaker evidence |
| C — Repo‑local autonomy (not recommended) | Local control | Highest drift; weakest auditability |

### FAQs (condensed)
- Will this slow merges? No — SAST blocks only High/Critical; SCA is report‑only.
- Who can change protections or thresholds? Security/Infra via PRs.
- Why not archive early for onboarding gaps? Blocking merges keeps remediation fast.

For full detail, see `github-repository-controls-one-pager.md`.

