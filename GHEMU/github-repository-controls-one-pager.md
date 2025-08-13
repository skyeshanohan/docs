### GitHub Repository Controls — One‑Pager

**Executive summary**  
Standardize repository security across the enterprise with uniform default‑branch protections, mandatory onboarding and lifecycle controls, and a new least‑privilege `Repository Manager` role. Prefer centralized policy‑as‑code to eliminate drift, improve auditability, and preserve developer velocity.

### Repository controls (what changes)
- **Default‑branch protections**: All default‑branch changes via PR; at least **+1 non‑committer** approval; all PR conversations resolved; no force‑push/deletes on protected branches. Required checks (stable names): "SAST Compliance Check" (blocks on High/Critical), "SCA Compliance Check" (report‑only; always passes). Scans run on all PRs.
- **Creation requirements (within 5 days)**: Set `Service ID` (valid CMDB ID) custom property and onboard security scanning: SAST, SCA, Secrets Scanning with Push Protection. Non‑compliance auto‑archives the repository until fixed.
- **Scanning and alerts**: SAST on every PR (merge gate on High/Critical). SCA on PRs and nightly (no merge gate; alerts tracked). Secrets Push Protection enabled; bypass allowed with justification; all bypasses logged and reviewed.
- **Lifecycle & archival**: Auto‑archive on missing `Service ID`, incomplete onboarding after 5 days, 365‑day inactivity on default branch, CMDB decommission +30 days. Decommission: archive → export to S3 (checksum, evidence) → delete. Native restore window: 90 days post‑deletion.

### Roles model (least privilege)
- **Repository Manager (new, assigned via Teams only)**: Day‑to‑day operations within guardrails. Allowed: issues/PRs/releases/labels/milestones/projects/metadata; rerun/dispatch existing workflows; manage merge queue. Not allowed: delete/transfer/change visibility, change default branch, manage access, change rulesets/branch protections, manage security features, bypass protections, manage webhooks/secrets/variables/environments.
- **Admin (break‑glass only)**: Held by security/infra automation. Human elevation is time‑bound (≤ 24h), ticketed, audited. No direct individual Admin assignments.

### Implementation options
- **Option A — Centralized policy‑as‑code (recommended)**
  - How it works: Single policy repo (e.g., `DV/security-policy`) defines per‑org desired state in `policies/organizations.yml`; provides reusable SAST/SCA workflows exposing stable check names; a GitHub App‑backed reconciler enforces rulesets/variables and detects drift; event automation (e.g., n8n) opens onboarding PRs and posts notifications. Safety kill switches: `ENFORCEMENT_KILL_SWITCH` (read‑only mode) and `ARCHIVAL_KILL_SWITCH` (pause archival/deletes).
  - Advantages: Consistency, auditability via PRs, fast onboarding, minimal drift, vendor‑agnostic check names, controlled emergency pause.
  - Tradeoffs: Requires initial setup of policy repo, App permissions, and cross‑org workflow access.

- **Option B — Org‑level manual rulesets + templates (minimum viable)**
  - How it works: Security configures rulesets per org and ships workflow templates; periodic audits for drift.
  - Advantages: Lower initial lift; fewer moving parts.
  - Tradeoffs: Higher drift/variance, weaker evidence trail, slower remediation, heavier manual effort.

- **Option C — Repo‑local autonomy (not recommended)**
  - How it works: Teams own workflows and protections locally.
  - Tradeoffs: Inconsistent enforcement, high drift risk, difficult auditing, slower enterprise response.



### References
- Baseline controls: [github_repository_controls](./github_repository_controls)
- Centralized management details: [github_repository_controls_centralized_management](./github_repository_controls_centralized_management)
- Role charter: [repository-manager-role-charter.md](./repository-manager-role-charter.md)

### Quick FAQs
- **Will this slow merges?** No. SAST blocks only High/Critical; SCA is report‑only. A kill switch enables read‑only enforcement during outages.
- **Who can change protections or thresholds?** Only `secprodinfra`, via PRs in the policy repo.
- **What if onboarding is missed?** Repos auto‑archive after 5 days; restore by completing requirements and requesting unarchive.
- **Can Repository Managers add users or change rulesets?** No. Access is via Teams; rulesets/security settings are centrally owned.

