### CODEOWNERS Control — Path Ownership and Coverage

Purpose
- Establish clear, auditable code ownership for every path in each repository to ensure the right teams review changes and are accountable for quality and security.

Hard Requirements
- **Location**: A `CODEOWNERS` file must exist at `.github/CODEOWNERS`.
- **Coverage**: 100% path coverage is required. Include a final wildcard fallback owner line to catch any unmatched paths.
- **Fallback owner**: Use a dedicated team (for example): `@org/codeowners-fallback`.
- **Team-based ownership**: Use GitHub Teams only; do not list individual usernames. Cross-org external teams are not allowed.
- **Review gating**: Branch protections/rulesets must enable "Require review from Code Owners" for the default branch.
- **Change control**: PRs that modify `CODEOWNERS` must receive at least one approval from a non‑committer (no self-approval).

Rationale
- Using `.github/CODEOWNERS` standardizes the location across repositories, improves discoverability in the GitHub UI, and simplifies automation by scanning a single canonical path. It also avoids root-level clutter, especially in monorepos.

Validation and Enforcement (central policy)
- **Presence**: Reconciler ensures `.github/CODEOWNERS` exists.
- **Coverage**: Reconciler verifies at least one fallback wildcard owner exists and that every top-level directory in `HEAD` is owned directly or by a matching pattern. Gaps raise a compliance issue.
- **Team-only**: Reconciler flags individual usernames; only teams are permitted.
- **Change review**: Reconciler enforces that PRs modifying `CODEOWNERS` require at least one approval from a non‑committer.
- **Drift response**: Non-compliance opens an issue, labels the repo, and triggers reminders. Do not auto-archive for CODEOWNERS gaps; merges remain blocked by rules that require Code Owners review.

References
- Baseline controls: `github_repository_controls`
- Centralized management: `github_repository_controls_centralized_management`
