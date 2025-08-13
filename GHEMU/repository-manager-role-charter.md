### Repository Manager — Role Charter (Internal)

Purpose
- Enable day-to-day repository operations without granting destructive, access, or security administration rights.
- Balance developer velocity with least privilege and separation of duties.

Rationale
- Reduce blast radius and configuration drift by removing full Admin from individual contributors.
- Centralize high-risk changes (security, access, rulesets) under `secprodinfra` for consistency and auditability.
- Preserve team autonomy for delivery tasks (PRs, releases, labels, workflow runs) within enforced guardrails.

Scope of Authority (Allowed)
- Repository metadata: edit description, topics, homepage, custom properties (e.g., Service ID once provided by owning team).
- Issues and PRs: open/close, triage, label, assign, manage milestones; merge PRs when required checks pass and +1 non-committer approval is present.
- Releases: draft and publish releases; manage release notes and tags (no visibility changes).
- Projects/Boards: create and manage repository project boards (classic/Projects).
- GitHub Actions (operational use only): view logs/artifacts, re-run and dispatch existing workflows; manage merge queue within protections.
- Insights/Reporting: view repository traffic, contributors, dependency graph (read-only security insights).

Explicitly Out of Scope (Not Allowed)
- Destructive/ownership: delete, archive, or transfer repository; change repository visibility; change default branch.
- Access management: add/remove collaborators; grant/revoke team permissions; invite outside collaborators.
- Security & compliance: create/edit rulesets or branch protections; enable/disable/reconfigure Secret Scanning, Push Protection, Code Scanning/SAST, SCA/Dependabot; dismiss security alerts; manage environments, secrets, variables; manage webhooks; change Actions settings or permissions.
- Admin bypass: cannot bypass rulesets/branch protection; cannot force-push or delete protected branches.

Assignment & Governance
- Assignment: via team membership only (no direct individual assignment). Recommended team name: `<org>-repo-managers`.
- Source of truth: Organization-level custom repository role named "Repository Manager" (based on Maintain with targeted removals; see Implementation Notes below).
- Change control: Any governance/policy change requires PR approval by GitHub team `secprodinfra` (no self-approval). No policy exceptions.

Guardrails & Escalation
- Break-glass: time-bound elevation (≤ 24h) with dual approval and ticket reference; automatic revocation; fully audited.
- Central oversight: `secprodinfra` owns security features, rulesets, branch protections, and kill-switches.
- Evidence: role definition, changes, and audits stored in the central policy repo (e.g., `DV/security-policy`).

Operating Model
- Merges: All default-branch merges require passing status checks ("SAST Compliance Check", "SCA Compliance Check") and +1 non-committer approval.
- Scanning: SAST runs on every PR across all branches; SCA runs on PRs and nightly on default branch.
- Secrets Push Protection: Bypass allowed with justification; events reported to `#team-spi-alerts` and reviewed weekly by security.

Implementation Notes (GitHub Enterprise)
- Start from GitHub’s "Maintain" baseline and explicitly remove: Delete/Transfer/Visibility change; Manage Access; Manage Security & Analysis; Manage Rulesets; Edit Branch Protections; Manage Webhooks; Manage Environments/Secrets/Variables; Manage Pages; Change Default Branch; Actions settings.
- Retain: Issues, PRs, Releases, Labels, Milestones, Projects, Metadata; Workflow run re-run/dispatch; Merge queue operations.
- Assign only via Teams; conduct quarterly access reviews.

FAQs (Anticipated)
- Why not grant Admin? Admin enables destructive and security-impacting operations that undermine policy controls and auditability.
- Can Repository Managers change branch protections or rulesets? No. Those are owned by `secprodinfra` to ensure consistent enforcement.
- Can they add users or invite contractors? No. Access changes go through team membership managed by organization owners/security.
- Can they manage webhooks for integrations? Default: No. If needed, request `secprodinfra` to evaluate and install.
- Who approves policy or threshold changes (e.g., SAST severity gates)? Only `secprodinfra` can approve or modify these settings.
- How do we handle emergencies? Use break-glass with dual approval, ≤ 24h duration, followed by post-incident review.
- Can they merge without checks? No. Required checks and +1 non-committer approval are mandatory; no self-approval.

Owner
- Security Engineering (role definition); `secprodinfra` (governance and enforcement).

Version & Review
- Effective: upon publication
- Review cycle: semiannual or upon material policy change
