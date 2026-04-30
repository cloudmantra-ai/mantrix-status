# Mantrix Status Page (Upptime)

Public uptime + incident page for the Mantrix platform. Built on
[Upptime](https://upptime.js.org) — runs on GitHub Actions, published to GitHub Pages,
hosted on infrastructure independent from the systems it monitors. This independence
is what makes it valid SOC2 availability evidence.

## How it works

- GitHub Actions probes each URL in `.upptimerc.yml` every 5 minutes.
- Results are committed to `history/` (immutable git trail = audit evidence).
- A static status site is rebuilt and published to GitHub Pages.
- When a probe fails, an Issue is opened and assigned to the on-call user.
- Incident updates posted as Issue comments appear on the public page.

## One-time setup

1. Create a **new public** GitHub repo named `mantrix-status` under the `cloudmantra-ai` org.
2. Push the contents of this directory to that repo:
   ```bash
   cd status-page
   git init && git add . && git commit -m "init upptime"
   git branch -M master
   git remote add origin git@github.com:cloudmantra-ai/mantrix-status.git
   git push -u origin master
   ```
3. In the repo settings:
   - **Actions → General → Workflow permissions** → "Read and write permissions" + "Allow GitHub Actions to create and approve pull requests"
   - **Pages → Source** → `gh-pages` branch (created automatically on first run)
   - **Secrets and variables → Actions → New secret** → `GH_PAT` = a fine-grained PAT with `contents:write`, `issues:write`, `pages:write`, `actions:write` on this repo
4. (Optional) Point `status.cloudmantra.ai` (CNAME) at `<owner>.github.io`. Remove the
   `cname:` line in `.upptimerc.yml` if not using a custom domain.
5. (Optional) Add a Slack webhook for alerts:
   ```bash
   gh secret set NOTIFICATION_SLACK_WEBHOOK_URL --body "https://hooks.slack.com/..."
   ```
   Then add a `notifications:` block to `.upptimerc.yml` (see Upptime docs).

## SOC2 mapping

| Control                           | Evidence                                       |
| --------------------------------- | ---------------------------------------------- |
| CC7.2 — System monitoring         | GitHub Actions cron + commit history of probes |
| CC7.3 — Incident detection        | Auto-opened Issues on probe failure            |
| CC7.4 — Incident response & comms | Issue comments → public status page            |
| A1.2 — Availability commitment    | 90-day uptime % published per component        |

## Adding components

Edit `.upptimerc.yml`, add to `sites:`. The next scheduled run picks it up. Keep
internal services (Postgres, Mongo, Neo4j) **out** of this list — they're not publicly
reachable and exposing them is a security risk. Internal observability belongs in
CloudWatch / ELK, not the public status page.
