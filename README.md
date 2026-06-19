# Daily University Email Crawler (GitHub Actions)

Runs your Scrapy email-extractor automatically every day. No server, no hosting,
no maintenance — GitHub runs it for you on a schedule and commits results
straight into this repo.

## ⚠️ Required setup: GitHub Secrets (for email notifications)

Before the workflow runs successfully, add these two secrets to your repo:
**Repo → Settings → Secrets and variables → Actions → New repository secret**

| Secret name      | Value                                  |
|------------------|------------------------------------------|
| `SMTP_USERNAME`  | `support@onegrasp.com`                  |
| `SMTP_PASSWORD`  | your mailbox password (change it first since it was shared in plaintext earlier — reset it in Hostinger's email panel, then use the new one here) |

Never put the password directly into the `.yml` file or commit it — secrets are
the only safe place for it. The workflow references them as
`${{ secrets.SMTP_USERNAME }}` / `${{ secrets.SMTP_PASSWORD }}`, which GitHub
injects securely at runtime and masks in logs.

## One-time setup

1. **Create a new GitHub repo** (private is fine) and push this folder to it:
   ```bash
   cd email-crawler-repo
   git init
   git add .
   git commit -m "Initial commit"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<repo-name>.git
   git push -u origin main
   ```

2. **Edit `websites.txt`** with the university URLs you want to crawl (one per line).
   Commit and push any time you want to change the list:
   ```bash
   git add websites.txt
   git commit -m "Update website list"
   git push
   ```

3. That's it. The workflow in `.github/workflows/daily-crawl.yml` is already
   configured to run automatically — nothing else to do.

## What happens daily

Every day at **3:00 AM IST** (21:30 UTC), GitHub automatically:
1. Spins up a fresh Ubuntu runner
2. Installs dependencies
3. Runs `scrapy crawl email_spider` using whatever is in `websites.txt`
4. Saves results to:
   - `results/YYYY-MM-DD/extracted_emails.txt`
   - `results/YYYY-MM-DD/report.txt`
   - `latest_extracted_emails.txt` and `latest_report.txt` (always the newest run, easy to find)
5. Commits and pushes these files back into the repo automatically

You'll see a new commit appear in the repo each morning. You can browse
`results/` for the full history, or always just open `latest_extracted_emails.txt`
for today's output.

## Checking it worked
Go to your repo → **Actions** tab → you'll see "Daily Email Crawl" runs listed
with green checkmarks (success) or red X (failure, click in to see logs).

## Running it manually / testing right now
You don't have to wait for 3 AM to test it:
1. Go to repo → **Actions** tab → click "Daily Email Crawl" in the left sidebar
2. Click **Run workflow** (top right) → **Run workflow**
3. Wait ~1-2 minutes, refresh, click into the run to see logs and confirm it worked

## Changing the schedule
Edit the `cron` line in `.github/workflows/daily-crawl.yml`:
```yaml
- cron: '30 21 * * *'   # this is UTC time, not IST
```
Cron format is `minute hour day month weekday`, always in **UTC**.
Use https://crontab.guru to build a different schedule if needed.

## Getting notified (optional)
If you want an email/Slack ping whenever new emails are found instead of
manually checking GitHub, that can be added as an extra step in the workflow —
just ask and it can be wired in (e.g. using a free service like Resend or
a Slack webhook).

## Why this approach over a hosted web app
- **Zero hosting cost or maintenance** — no Render/Fly.io account to keep alive,
  no server to monitor for downtime.
- **Free tier on GitHub Actions** (2,000 minutes/month on private repos,
  unlimited on public repos) is far more than a daily few-minute crawl needs.
- **Full history is version-controlled** — every day's results are a git commit,
  so you can always look back at what was found on any past date.
- If you later also want the on-demand "upload a custom list right now" web UI,
  that can run alongside this (separate, optional) — they don't conflict.
