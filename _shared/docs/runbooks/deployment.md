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
> Staging deploys happen automatically on merge to `main`. This runbook covers production deploys (triggered by a git tag) and manual staging re-deploys.

---

## Pre-flight checklist

> [!warning]
> Complete all pre-flight checks before tagging a production release. Skipping steps has caused incidents.

:::expand Pre-flight checklist
- [ ] All CI checks passing on `main` (lint, tests, security scan)
- [ ] Staging deploy successful and smoke tests passing (automated — check GitHub Actions)
- [ ] No active incidents in `#incidents` that could be worsened by a deploy
- [ ] Kafka consumer lag on `orders.order-events` is < 10 seconds (check Datadog)
- [ ] Inventory Service health check passing (order creation depends on it)
- [ ] Change communicated to `#deployments` with estimated deployment window
- [ ] Rollback plan reviewed (see [Rollback Runbook](page:"Rollback Procedure"))
:::

---

## Production deployment steps

### 1. Create and push the release tag

```bash
git checkout main
git pull origin main

# Confirm you're on the right commit
git log --oneline -5

# Tag the release
git tag v1.8.2 -m "Release v1.8.2: bulk order API, inventory webhook events"
git push origin v1.8.2
```

### 2. Monitor the CI deploy pipeline

1. Open the GitHub Actions run for the tag push
2. Watch the **Build & Push** job — confirm the Docker image is built and pushed to ECR
3. Watch the **Deploy to Production** job — it applies the Helm chart to the production EKS cluster
4. Deployment is rolling — old pods are terminated gradually as new pods pass health checks

### 3. Verify the deployment

```bash
# Check pod rollout status
kubectl rollout status deployment/order-service -n orders

# Confirm running version
kubectl get pods -n orders -o jsonpath='{.items[*].spec.containers[0].image}'

# Run smoke tests against production
make smoke-test ENV=production
```

### 4. Post-deploy checks (first 15 minutes)

Monitor the following in Datadog after every production deploy:

| Metric | Expected | Alert if |
| --- | --- | --- |
| Order creation success rate | > 99.5% | < 99% |
| Order creation p99 latency | < 500ms | > 800ms |
| Error rate (5xx) | < 0.1% | > 0.5% |
| Kafka event lag | < 30s | > 120s |
| Pod restart count | 0 | Any restart |

### 5. Announce in #deployments

```
:rocket: Order Service v1.8.2 deployed to production
Changes: bulk order API (POST /orders/bulk), inventory.low_stock webhook event
Rollback: git tag v1.8.1 is the previous stable version
```

---

## Manual staging redeploy

If you need to re-deploy a specific commit to staging without merging:

```bash
# Trigger a manual workflow run targeting a branch or SHA
gh workflow run deploy-staging.yml \
  --ref feature/my-branch \
  -f reason="Testing webhook changes"
```

---

## If the deploy fails

1. Check the GitHub Actions logs for the failure step
2. If pods are crash-looping: `kubectl logs -n orders -l app=order-service --previous`
3. If the rollout is stuck: assess whether to wait or [rollback](page:"Rollback Procedure")
4. Post status in `#incidents` if customers are impacted
