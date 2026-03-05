---
page_meta:
  title: Deployment
  labels:
    - order-service
    - runbook
    - deployment
deploy_config:
  ci_banner: true
  include_page_metadata: true
---

# Deployment Runbook

Procedure for deploying the Order Service to staging and production.

**Owner**: Orders Team
**On-call**: PagerDuty → Orders rotation

> [!info]
> Staging deploys happen automatically on merge to `main`. This runbook covers production deploys (triggered via the GitLab pipeline UI) and manual staging re-deploys.

---

## Pre-flight checklist

> [!warning]
> Complete all pre-flight checks before triggering a production release. Skipping steps has caused incidents.

:::expand Pre-flight checklist
- [ ] All CI checks passing on `main` (lint, tests, security scan)
- [ ] Staging deploy successful and smoke tests passing (check the GitLab pipeline)
- [ ] No active incidents in `#incidents` that could be worsened by a deploy
- [ ] Kafka consumer lag on `orders.order-events` is < 10 seconds (check Datadog)
- [ ] Inventory Service health check passing (order creation depends on it)
- [ ] Change communicated to `#deployments` with estimated deployment window
- [ ] Rollback plan reviewed (see [Rollback Runbook](page:"Rollback Procedure"))
:::

---

## Production deployment steps

### 1. Verify staging looks correct in Confluence

Review the updated pages in the staging Confluence space (`ENG-STAGE`) to confirm content is correct before promoting to production.

### 2. Trigger the production deploy

1. Open **CI/CD → Pipelines** in GitLab
2. Find the pipeline for the merge commit on `main`
3. Click the play button next to `deploy:production`
4. Monitor the job log — it runs `ccfm --changed-only --archive-orphans` against the production space

### 3. Verify the deployment

Once `deploy:production` completes:

```bash
# Run smoke tests against production
make smoke-test ENV=production
```

Check the following in Datadog:

| Metric | Expected | Alert if |
| --- | --- | --- |
| Order creation success rate | > 99.5% | < 99% |
| Order creation p99 latency | < 500ms | > 800ms |
| Error rate (5xx) | < 0.1% | > 0.5% |
| Kafka event lag | < 30s | > 120s |
| Pod restart count | 0 | Any restart |

### 4. Announce in #deployments

```
:rocket: Order Service docs deployed to production
Changes: [describe what changed]
Confluence: [link to updated pages]
```

---

## Manual staging redeploy

To re-deploy docs to staging without a code change:

1. Navigate to **CI/CD → Pipelines → Run pipeline**
2. Select the `main` branch
3. The `deploy:staging` job will run automatically (if docs files changed since last state)

---

## If the deploy fails

1. Check the GitLab job log for the failure step
2. If credentials are wrong: verify `CONFLUENCE_DOMAIN`, `CONFLUENCE_EMAIL`, `CONFLUENCE_TOKEN` variables
3. If the space key is wrong: check `CONFLUENCE_SPACE` is correctly scoped to the environment
4. Post status in `#incidents` if the issue blocks a release
