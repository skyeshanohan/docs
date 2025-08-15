### Enterprise GitHub App: Compliance Settings Check — Technical Guide

Purpose
- Enforce baseline repository settings without per-repo workflow files by using org rulesets and an enterprise GitHub App that publishes a required status check.

Summary
- Org Rulesets require three stable checks on default-branch PRs: "SAST Compliance Check", "SCA Compliance Check", and "Compliance Settings Check".
- The App evaluates repository settings and custom properties on PR events and publishes "Compliance Settings Check". GitHub blocks merges until all required checks succeed.

App Configuration
- Permissions (minimum):
  - Checks: write
  - Pull requests: read
  - Contents: read
  - Security events: read
  - Repository administration: write (rulesets)
  - Organization administration: write (org rulesets/variables)
  - Actions: read (variables)
- Webhooks: pull_request (opened, synchronize, reopened, ready_for_review), check_suite (requested, rerequested), repository (created), optional: code_scanning_alert, secret_scanning_alert, secret_scanning_push_protection_bypass

Rulesets
- Organization → Settings → Rules:
  - Require pull requests to default branch
  - Require reviews: +1 non-committer and Code Owners
  - Require status checks: "SAST Compliance Check", "SCA Compliance Check", "Compliance Settings Check"
  - Disable admin bypass

Org-wide Security Features (no repo PRs)
- Organization → Settings → Code security and analysis:
  - Enable Secret scanning and Push Protection for all repos
  - Enable Code scanning Default setup for all repos

App Behavior (PR evaluation)
- On PR events, the App reads repository settings and properties via REST API:
  - Custom property: `Service ID` present and regex-valid
  - Secret scanning: enabled
  - Push Protection: enabled
  - Code scanning default setup: enabled
- It posts a check run named "Compliance Settings Check" to the PR head SHA with conclusion success/failure and a markdown summary of any missing items.

Failure semantics
- If any required setting/property is missing, "Compliance Settings Check" fails. Because the ruleset requires this check, the PR cannot merge until fixed. The repository remains writable for remediation.

SAST/SCA without repo workflows
- SAST: Use Code Scanning Default setup (CodeQL). Optionally normalize results into "SAST Compliance Check" via the App.
- SCA: Use Dependency Review API on the PR diff; publish "SCA Compliance Check" as report-only (success) with annotations.

Re-runs and Drift Handling
- Users can "Re-run checks" which triggers `check_suite` rerequested for a fresh evaluation.
- A scheduled job can periodically re-evaluate open PRs.
- On `repository.created`, the App can perform an initial evaluation and optionally auto-enable defaults if permitted.

Security & Operations
- Store App private key and installation IDs in a protected environment in the central policy repo.
- Audit all changes through PRs to the policy repo and App logs.

API Endpoints (reference)
- GET /repos/{owner}/{repo}
- GET /repos/{owner}/{repo}/properties/values
- POST /repos/{owner}/{repo}/check-runs
- GET /repos/{owner}/{repo}/code-scanning/alerts (optional)
- POST /orgs/{org}/rulesets (management)


Examples

Python (GitHub App) — create/update the Compliance Settings Check
```python
import json
import os
import time
import jwt
import requests

GITHUB_APP_ID = os.environ["GITHUB_APP_ID"]
GITHUB_APP_PK = os.environ["GITHUB_APP_PRIVATE_KEY_PEM"]
INSTALLATION_ID = os.environ["GITHUB_APP_INSTALLATION_ID"]

def _app_jwt() -> str:
    now = int(time.time())
    payload = {
        "iat": now - 60,
        "exp": now + 9 * 60,
        "iss": GITHUB_APP_ID,
    }
    return jwt.encode(payload, GITHUB_APP_PK, algorithm="RS256")

def _installation_token() -> str:
    headers = {
        "Authorization": f"Bearer {_app_jwt()}",
        "Accept": "application/vnd.github+json",
    }
    url = f"https://api.github.com/app/installations/{INSTALLATION_ID}/access_tokens"
    resp = requests.post(url, headers=headers, timeout=30)
    resp.raise_for_status()
    return resp.json()["token"]

def publish_compliance_check(owner: str, repo: str, head_sha: str, missing: list[str]):
    conclusion = "failure" if missing else "success"
    summary_md = "\n".join(["All required settings present." if not missing else "Missing:"] + [f"- {m}" for m in missing])
    body = {
        "name": "Compliance Settings Check",
        "head_sha": head_sha,
        "status": "completed",
        "conclusion": conclusion,
        "output": {
            "title": "Compliance Settings Check",
            "summary": summary_md,
        },
    }
    token = _installation_token()
    headers = {
        "Authorization": f"token {token}",
        "Accept": "application/vnd.github+json",
    }
    url = f"https://api.github.com/repos/{owner}/{repo}/check-runs"
    resp = requests.post(url, headers=headers, json=body, timeout=30)
    resp.raise_for_status()
    return resp.json()
```

JSON — check-run payloads
```json
{
  "name": "Compliance Settings Check",
  "head_sha": "<PR_HEAD_SHA>",
  "status": "completed",
  "conclusion": "success",
  "output": {
    "title": "Compliance Settings Check",
    "summary": "All required settings present."
  }
}
```

```json
{
  "name": "Compliance Settings Check",
  "head_sha": "<PR_HEAD_SHA>",
  "status": "completed",
  "conclusion": "failure",
  "output": {
    "title": "Compliance Settings Check",
    "summary": "Missing:\n- Service ID custom property (invalid/missing)\n- Secret scanning not enabled\n- Push Protection not enabled\n- Code scanning default setup not enabled"
  }
}
```

