---
page_meta:
  title: Engineer Onboarding
  labels:
    - engineering
    - onboarding
deploy_config:
  ci_banner: true
---

# Engineer Onboarding

Welcome to Acme Corp Engineering! This checklist walks you through your first two weeks. Your manager will set up a 1:1 on day 1 — bring questions.

> [!success]
> Most engineers are shipping to production within their first week. Don't be afraid to ask questions — the team expects it and values curiosity over false confidence.

## Before you start

Your hiring manager will send you a welcome email with:
- Laptop delivery tracking
- Google Workspace invite
- Slack invite

Make sure you can log in to both before day 1.

---

## Week 1 Checklist

:::expand Day 1 — Access & orientation
**Morning**
1. Collect your laptop from reception (bring photo ID)
2. Log in to Google Workspace — enable 2FA immediately
3. Join Slack — introduce yourself in `#general`
4. Accept your GitHub organisation invite (check email)

**Afternoon**
1. Read through this wiki — start with [Coding Standards](page:"Coding Standards")
2. Clone the `acme-platform` monorepo: `git clone git@github.com:acme-corp/acme-platform.git`
3. Follow the `README.md` to get the local dev environment running
4. Check in with your manager at end of day
:::

:::expand Day 2–3 — Tooling & access
1. Set up your local Python, Node, and Go environments (see `docs/local-setup.md` in the repo)
2. Request access to AWS sandbox account via `#it-helpdesk` — provide your IAM username
3. Install the Acme CLI: `pip install acme-cli` and run `acme auth login`
4. Verify you can run the full test suite: `make test`
5. Read the [Architecture Overview](page:"Architecture Overview") — ask your team lead to walk you through it
:::

:::expand Day 4–5 — First contribution
1. Pick up a `good-first-issue` from the backlog (your manager will point you to one)
2. Create a branch, make the change, open a PR
3. Go through the full review cycle — respond to feedback, iterate
4. Merge your first PR — you're now a contributor :white_check_mark:
:::

---

## Week 2 Checklist

:::expand Development environment deep-dive
1. Explore the service you'll be primarily working on — read its `README.md` top to bottom
2. Run it locally with dependencies using Docker Compose: `docker compose up`
3. Trigger at least one integration test against your local stack
4. Understand how secrets are managed — read `docs/secrets-management.md`
:::

:::expand Process & culture
1. Attend your first sprint planning (check with your manager for timing)
2. Read the [People & Culture handbook](page:"People & Culture Handbook")
3. Shadow an on-call shift — your manager will arrange this with the current on-call engineer
4. Set up your 30/60/90 day goals with your manager
:::

---

## Key contacts

| Role | Name | Slack |
| --- | --- | --- |
| Engineering Manager | Sam Okafor | @sam.okafor |
| Platform Lead | Priya Mehta | @priya.mehta |
| Security | Jordan Lee | @jordan.lee |
| IT Helpdesk | — | #it-helpdesk |

> [!note]
> If you're blocked on access or tooling during your first week, post in `#it-helpdesk` — we aim to respond within 2 hours during business hours.
