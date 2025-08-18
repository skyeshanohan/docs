### GitHub App Governance — One‑Pager

**Executive summary**  
Establish a predictable 180‑day review cadence for GitHub Apps and ensure timely visibility of new installation requests. A nightly job posts Slack reminders for due/overdue app reviews, and Slack notifications are sent when new installation requests are submitted (replacing email‑only visibility). Evidence of reviews is stored centrally via policy‑as‑code.

### At‑a‑glance controls

| Area                          | Decision/Default                                                                                 | Merge gate? | Owner                              |
|-------------------------------|--------------------------------------------------------------------------------------------------|-------------|------------------------------------|
| Review cadence                | Every 180 days from the last recorded security review                                            | N/A         | Application Security               |
| Nightly reminder job          | Scheduled job calculates due/overdue GitHub Apps and posts a Slack summary with owners/actions  | N/A         | Security Engineering (automation)  |
| New install request alerting  | Slack message on each new GitHub App installation request (replaces email‑only)                 | N/A         | Security Engineering (automation)  |
| Evidence & registry           | Central `apps-reviews.yml` registry in policy repo; PR updates capture approvals/sign‑off       | N/A         | Application Security               |
| Channels                      | `#sec-github-apps` (operations) and `#eng-announcements` (summary as needed)                    | N/A         | Application Security               |
| Exceptions                    | None; undocumented Apps or missing owners are flagged and escalated                             | N/A         | Application Security               |

### What leaders need to know
- Slack replaces email‑only visibility for GitHub App installation requests and periodic reviews.
- Reviews focus on usage and least‑privilege permissions; outcomes and next due dates are recorded in a central registry (auditable via PRs).
- The automation is read‑only for GitHub settings; humans make changes to App scopes/installs.

### Implementation options at a glance

| Option                                                    | Summary                                                                                              | Tradeoff                                      |
|-----------------------------------------------------------|------------------------------------------------------------------------------------------------------|-----------------------------------------------|
| A — GitHub Actions + Audit Log API (recommended)          | Nightly cron for 180‑day reminders; frequent cron polls org Audit Log for new install requests       | Easiest to host; polling granularity          |
| B — Org Webhook → n8n/server → Slack                      | Org webhook or audit‑log streaming to a small service that relays Slack notifications                | Requires running infra; lowest latency        |
| C — Email → Slack bridge (not recommended)                | Forward org owner emails to Slack                                                                    | Fragile; uneven formatting and filtering      |

<details>
<summary>GitHub App reviews — full detail</summary>

### Data and evidence model
- Central registry file in the policy repo (example path: `policy/github-apps/apps-reviews.yml`). Changes are PR‑reviewed by Application Security.
- Each record tracks: `app_slug`, `display_name`, `owner_team`, `install_scope` (orgs/repos), `permissions`, `last_reviewed_at` (UTC ISO 8601), `reviewed_by`, `notes`.

Example `apps-reviews.yml`:
```yaml
apps:
  - app_slug: example-app
    display_name: Example App
    owner_team: "@org/security-architecture"
    install_scope: ["org:dv", "repo:dv/example-repo"]
    permissions:
      - metadata: read
      - contents: read
      - pull_requests: write
    last_reviewed_at: "2025-01-15T00:00:00Z"
    reviewed_by: "@org/appsec-oncall"
    notes: "Used for PR labelling; confirmed least privilege."
```

### Nightly reminder job (180 days)
- Runs nightly and computes next‑due as `last_reviewed_at + 180 days`.
- Flags statuses: `due_in_7_days`, `due_today`, `overdue`.
- Posts to Slack with a summary and per‑app action links (view registry entry and owner team).

Example GitHub Actions workflow `github-app-review-reminders.yml`:
```yaml
name: GitHub App Review Reminders
on:
  schedule:
    - cron: "0 7 * * *"  # nightly at 07:00 UTC
  workflow_dispatch: {}
jobs:
  remind:
    runs-on: ubuntu-latest
    permissions:
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 1
      - name: Compute due reviews and post to Slack
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
          REGISTRY_PATH: policy/github-apps/apps-reviews.yml
        run: |
          python - <<'PY'
          import os, sys, json, datetime, yaml, urllib.request
          from dateutil.relativedelta import relativedelta
          from pathlib import Path

          registry_path = Path(os.getenv('REGISTRY_PATH', 'policy/github-apps/apps-reviews.yml'))
          data = yaml.safe_load(registry_path.read_text()) or {}
          apps = data.get('apps', [])

          now = datetime.datetime.utcnow().replace(tzinfo=datetime.timezone.utc)
          due, soon, overdue = [], [], []

          for app in apps:
            last = datetime.datetime.fromisoformat(app['last_reviewed_at'].replace('Z', '+00:00'))
            next_due = last + relativedelta(days=+180)
            days = (next_due - now).days
            record = {**app, 'next_due': next_due.date().isoformat(), 'days_until_due': days}
            if days < 0:
              overdue.append(record)
            elif days == 0:
              due.append(record)
            elif days <= 7:
              soon.append(record)

          def section(title, items, emoji):
            if not items:
              return []
            lines = [f"*{title}* ({len(items)})"]
            for r in sorted(items, key=lambda x: x['days_until_due']):
              lines.append(f"{emoji} {r['display_name']} (`{r['app_slug']}`) — next due {r['next_due']} — owner {r['owner_team']}")
            return [{"type": "section", "text": {"type": "mrkdwn", "text": "\n".join(lines)}}]

          blocks = []
          blocks.append({"type": "header", "text": {"type": "plain_text", "text": "GitHub App reviews — nightly status"}})
          blocks += section("Overdue", overdue, "❗")
          blocks += section("Due today", due, "📌")
          blocks += section("Due in next 7 days", soon, "⏳")
          if len(blocks) == 1:
            blocks.append({"type": "section", "text": {"type": "mrkdwn", "text": "All clear. No GitHub App reviews due."}})

          payload = json.dumps({"blocks": blocks}).encode('utf-8')
          req = urllib.request.Request(os.environ['SLACK_WEBHOOK_URL'], data=payload, headers={'Content-Type': 'application/json'})
          with urllib.request.urlopen(req) as resp:
            sys.stdout.write(f"Slack response: {resp.status}\n")
          PY
```

Notes
- The registry lives in a central policy repository to capture approvals as PR review evidence.
- A manual or scheduled weekly view can be sent to a leadership channel.

</details>

<details>
<summary>New installation requests — full detail</summary>

### Problem
Organization owners currently receive email notifications for GitHub App installation requests. Emails are often filtered and lack shared visibility.

### Desired behavior
- Post a Slack message to `#sec-github-apps` when a new GitHub App installation request is submitted, including requester, app, reason, and links to approve/deny.

### Implementation choices
1) GitHub Audit Log API polling (recommended for simplicity)
   - Poll the org Audit Log for recent events with category matching installation requests (for example: `integration_installation_request` or similar, depending on GitHub’s schema). Use a short interval (e.g., every 5–10 minutes) and a checkpoint to avoid duplicates.
2) Org webhook or Audit Log streaming to a small relay (n8n/server)
   - Lower latency and fewer API calls, but requires hosting a listener.

Example GitHub Actions workflow `github-app-install-requests-to-slack.yml` (polling):
```yaml
name: GitHub App Install Requests → Slack
on:
  schedule:
    - cron: "*/10 * * * *"  # every 10 minutes
  workflow_dispatch: {}
jobs:
  poll:
    runs-on: ubuntu-latest
    permissions:
      actions: read
      contents: read
      organization_administration: read  # needed for Audit Log API
    steps:
      - name: Determine window
        id: win
        run: |
          echo "since=$(date -u -v-15M +%Y-%m-%dT%H:%M:%SZ)" >> $GITHUB_OUTPUT
      - name: Fetch recent installation requests from Audit Log
        id: audit
        env:
          GH_TOKEN: ${{ secrets.ORG_ADMIN_TOKEN }}
          ORG: dv
        run: |
          # Query for events in the last ~15 minutes; filter by category matching installation requests
          # Adjust the phrase to match your org’s audit schema (e.g., category:integration_installation_request)
          url="/orgs/${ORG}/audit-log?phrase=category:integration_installation_request created:>=${{ steps.win.outputs.since }}"
          gh api -H "Accept: application/vnd.github+json" "$url" > events.json
          jq '[.[] | {created_at, actor, action, app: (.data.app_slug // .data.integration_name), note: .note}]' events.json > filtered.json
          cat filtered.json | jq
      - name: Post to Slack
        if: success()
        env:
          SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
        run: |
          python - <<'PY'
          import os, json, sys, urllib.request
          events = json.load(open('filtered.json'))
          if not events:
            sys.exit(0)
          blocks = [{"type": "header", "text": {"type": "plain_text", "text": "New GitHub App installation requests"}}]
          for e in events:
            line = f"{e.get('actor','someone')} requested install for {e.get('app','unknown app')} — {e.get('note','')}"
            blocks.append({"type": "section", "text": {"type": "mrkdwn", "text": line}})
          payload = json.dumps({"blocks": blocks}).encode('utf-8')
          req = urllib.request.Request(os.environ['SLACK_WEBHOOK_URL'], data=payload, headers={'Content-Type': 'application/json'})
          with urllib.request.urlopen(req) as resp:
            print(resp.status)
          PY
```

Notes
- Substitute the `phrase` filter to match your GitHub plan’s audit event names for app install requests. Validate by manually submitting a request and inspecting Audit Log entries.
- For a webhook‑based approach, relay the org webhook payload to Slack via n8n or a lightweight server; include guardrails (allowlist channels, rate limits).

</details>

### Ownership and Effective Date
- **Document Owner**: Application Security
- **Automation Owner**: Security Engineering
- **Effective Date**: 2025‑08‑01
- **Next Review**: 2026‑02‑01


