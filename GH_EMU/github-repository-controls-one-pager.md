### GitHub Repository Controls — One‑Pager

**Executive summary**  
Standardize repository security with enterprise policy‑as‑code, default branch protections, and lifecycle controls. Repository administrators cannot change protected settings; only centralized policy can.

### At‑a‑glance controls

| Area                              | Decision/Default                                                                                        | Merge gate?                                        | Owner                              |
|-----------------------------------|---------------------------------------------------------------------------------------------------------|----------------------------------------------------|------------------------------------|
| Default branch protections        | PRs required; ≥1 non‑committer approval; CODEOWNERS review; no force push/delete                        | Yes — PR + approvals                               | Enterprise Security/Infra (policy) |
| CODEOWNERS control                | 100% path coverage with Team owners; wildcard fallback to owning team                                   | Yes — CODEOWNERS review required                   | Enterprise Security/Infra (policy) |
| Compliance Settings Check         | Required PR check; validates Service ID, security features, SAST/SCA setup                              | Yes — required check                               | GitHub App (central)               |
| SAST                              | Runs on all PRs; blocks High/Critical                                                                   | Yes — severity gate                                | Reusable workflows (policy)        |
| SCA                               | Runs on PRs + nightly; report‑only                                                                      | No                                                 | Reusable workflows (policy)        |
| Secret Scanning & Push Protection | Enabled enterprise‑wide; bypass with justification; alerts to Slack                                     | Yes — Push Protection gate (bypassable)            | Enterprise Security/Infra          |
| Creation requirements             | Service ID set; auto PRs enable SAST/SCA; 5‑day onboarding window                                       | Yes — merges blocked if non‑compliant after 5 days | Automation + App                   |
| Lifecycle & archival              | Auto‑archive after 365 days inactivity or +30 after CMDB decommission; export → delete (90‑day restore) | N/A                                                | Policy + Automation                |
| Repository administrators         | Admins cannot change protected settings, security, or visibility                                        | N/A                                                | Enterprise Security/Infra          |

### What leaders need to know
- Decisions and enforcement live in a central policy repo; changes happen via PRs and are auditable.
- Teams keep working normally; only High/Critical SAST blocks merges. SCA is report‑only.
- Onboarding gaps do not lock teams out; merges to default branch are blocked after 5 days until completed.
- Push Protection can be bypassed with justification; all bypasses are surfaced to leadership channels.

### Implementation options at a glance

| Option                                       | Summary                                                                                | Tradeoff                                             |
|----------------------------------------------|----------------------------------------------------------------------------------------|------------------------------------------------------|
| A — Centralized policy‑as‑code (recommended) | Single policy repo + GitHub App reconciler; reusable SAST/SCA workflows; kill switches | Highest initial setup; best consistency and evidence |
| B — Org‑level rulesets + templates           | Manual rulesets and templates per org; periodic audits                                 | Lower lift; higher drift and weaker evidence         |
| C — Repo‑local autonomy (not recommended)    | Teams manage locally                                                                   | Highest drift; weakest auditability                  |


<details>
<summary>Repository controls — full detail</summary>

### Repository controls
- **Default branch protections**
  - All changes to the default branch must use pull requests.
  - Each PR requires at least one approval from a non‑committer and a mandatory Code Owners review.
  - Disallow force pushes and branch deletions on protected branches.
  - Required checks: "SAST Compliance Check" (blocks on High/Critical), "SCA Compliance Check" (report‑only; always passes), "Compliance Settings Check" (App‑emitted; fails when required settings/properties are missing).
  - SAST and SCA scans run on all PRs in the repository.
- **CODEOWNERS control**
  - Maintain `.github/CODEOWNERS` with 100% path coverage using Team owners with a wildcard fallback to the team that owns the repository (e.g., `* @org/codeowners-fallback`).
  - Changes to `CODEOWNERS` require at least one approval from a non-committer.
  - Rationale: a single canonical path improves discoverability and simplifies automation.
- **Creation requirements (Enforcement within 5 days)**
  - Set the `Service ID` custom property (valid CMDB ID).
  - Enable SAST and SCA via automatically created PRs.
  - If onboarding is incomplete, required checks block merges; the repository remains writable and automation posts reminders until finished. Teams cannot merge changes to the default branch if non-compliant after 5 days from repository creation. Once compliant, the status check will pass.
- **Scanning and alerts**
  - SAST runs on every PR and blocks merges for High or Critical findings.
  - SCA runs on PRs and nightly on the default branch without a merge gate; critical findings are reported in the pull request. No merge gates for SCA.
  - Secret Scanning and Push Protection are enabled enterprise‑wide. Bypass is allowed with justification; all bypasses are notified via n8n automation to the API alerts Slack channel.
- **Lifecycle and archival**
  - Auto‑archive only after 365 days of default branch inactivity or +30 days after a CMDB decommission event.
  - Onboarding gaps (e.g., missing `Service ID` or unmerged base controls) do not trigger archival, but instead block merges to the default branch.
  - Decommission flow: archive → export to S3 (with checksum and evidence) → delete. GitHub’s native restore window is 90 days post‑deletion.

</details>

<details>
<summary>Compliance Settings Check — full detail</summary>

### Compliance Settings Check — GitHub App
- **Purpose**
  - Enforce baseline repository settings without requiring per‑repository workflow files.
- **What it validates on each pull request**
  - `Service ID` custom property is present and valid.
  - `.github/CODEOWNERS` exists, and has 100% path coverage, and contains a fallback wildcard to a team.
  - Secret Scanning and Push Protection are enabled.
  - Code Scanning default setup (SAST) is enabled.
  - Software Composition Analysis (SCA) default setup is enabled.
- **How it gates merges**
  - Publishes the required "Compliance Settings Check" on the PR commit. Organization rulesets require this check, so merges are blocked until it succeeds.
- **When it runs**
  - On pull_request opened, synchronize, reopened, and ready_for_review events, and on check re‑runs. A periodic re‑evaluation of open PRs is optional.
- **What it does not require**
  - No per‑repository workflow files or PR templates are needed for this check to function.
- **Notifications and evidence**
  - The check output lists missing items with remediation guidance.

</details>

<details>
<summary>Administration and enforcement — full detail</summary>

### Administration and enforcement
- **Repository administrators**
  - May exist but cannot change protected settings.
  - Protected: rulesets/branch protections at the organization or enterprise level, security features, visibility.
- **Enterprise ownership**
  - Enterprise Security/Infra owns protected configurations and policy changes via the central policy repository.
- **Compliance Settings Check GitHub App**
  - See the dedicated section below; it publishes the required check and gates merges until repository settings and the `Service ID` are compliant.

</details>

<details>
<summary>Implementation options — full detail</summary>

### Implementation options
- **Option A — Centralized policy‑as‑code (Strongly recommended)**
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

</details>

### Quick FAQs
- **Will this slow merges?** No. SAST blocks only High/Critical; SCA is report‑only. A kill switch enables read‑only enforcement during outages.
- **Who can change org/enterprise protections or thresholds?** Only `secprodinfra`, via PRs in the policy repo.
- **What if onboarding is missed?** Merges to the default branch remain blocked by required checks until onboarding is complete. Automation sends reminders; repository stays writable for remediation.
- **Why no early archival for onboarding gaps?** Archiving removes write access and slows remediation. We rely on merge blocking and reminders so teams can complete onboarding without losing access.
- **Can repository administrators add users or change rulesets?** No. Access is via enterprise policy, GitHub Teams and security groups in Entra; rulesets/security settings are centrally owned.