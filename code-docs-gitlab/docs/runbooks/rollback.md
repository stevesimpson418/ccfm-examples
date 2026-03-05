---
page_meta:
  title: Rollback Procedure
  labels:
    - order-service
    - runbook
    - rollback
deploy_config:
  ci_banner: true
  include_page_metadata: true
---

# Rollback Procedure

Steps to roll back the Order Service to a previous version in production.

**Decision authority**: On-call engineer (does not require manager approval during an incident)

> [!warning]
> Rollback reverts the application code only. Database migrations are **not** automatically reversed. If the new version ran a migration, assess whether the old code is compatible before rolling back. If not, contact the Orders team lead before proceeding.

---

## When to roll back

Roll back if, after a production deploy, you observe:

- Order creation error rate > 1% for more than 2 minutes
- p99 latency > 2 seconds for more than 3 minutes
- Crash-looping pods that don't self-recover within 5 minutes
- A confirmed bug in a critical path that cannot be hot-patched

If you're uncertain, err toward rollback — it's faster to re-deploy a fix than to debug a degraded production system.

---

## Rollback steps

### Step 1: Confirm the previous stable version

```bash
# List recent tags
git tag --sort=-version:refname | head -10

# Check what's currently deployed
kubectl get deployment order-service -n orders \
  -o jsonpath='{.spec.template.spec.containers[0].image}'
```

### Step 2: Trigger the rollback

**Option A: Helm rollback (fastest — no CI)**

```bash
# List recent Helm releases
helm history order-service -n orders

# Roll back to the previous revision
helm rollback order-service -n orders

# Or roll back to a specific revision
helm rollback order-service 14 -n orders
```

**Option B: Re-tag and re-deploy via CI**

```bash
# Find the previous stable image digest
git show v1.8.1:charts/order-service/values.yaml | grep image

# Trigger a new pipeline on the previous tag via GitLab API or UI
```

> [!info]
> Option A (Helm rollback) is faster and preferred during an active incident. Option B is preferable when you need a full audit trail in CI.

### Step 3: Verify the rollback

```bash
# Confirm pods are running the previous image
kubectl get pods -n orders -o jsonpath='{.items[*].spec.containers[0].image}'

# Check rollout completed
kubectl rollout status deployment/order-service -n orders

# Run smoke tests
make smoke-test ENV=production
```

### Step 4: Monitor for 15 minutes

Same metrics as post-deploy checks:

| Metric | Expected |
| --- | --- |
| Order creation success rate | > 99.5% |
| p99 latency | < 500ms |
| Error rate (5xx) | < 0.1% |

### Step 5: Post in #incidents

```
:rewind: Order Service rolled back to v1.8.1
Reason: [describe the issue]
Impact window: [time from deploy to rollback completion]
Next steps: [investigation / fix plan]
```

---

## Post-rollback actions

1. Open a P1 Jira ticket in `ORDERS` describing the issue
2. Tag the bad release: `git tag v1.8.2-bad && git push origin v1.8.2-bad` (prevents accidental redeploy)
3. Schedule an incident review within 48 hours
4. Do not re-deploy until the root cause is confirmed and fixed

> [!warning]
> Do not re-deploy the same image that caused the incident without a code fix. If the issue is infrastructure-related (not code), document the evidence clearly before re-deploying.
