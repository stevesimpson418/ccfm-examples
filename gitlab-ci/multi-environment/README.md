# GitLab CI -- Multi-Environment

Staging-to-production deployment of Confluence docs using GitLab Environments and environment-scoped CI/CD variables. A single `ccfm.yaml` config drives both environments -- the `CONFLUENCE_SPACE` variable resolves to a different space key depending on which environment the job targets.

## Prerequisites

- GitLab project with CI/CD enabled
- Two Confluence spaces: one for staging (e.g. `ENG-STAGE`), one for production (e.g. `ENG`)
- Confluence API token ([create one here](https://id.atlassian.com/manage-profile/security/api-tokens))

## Setup

### 1. Copy the sample docs

```bash
cp -r ../../_shared/docs docs/
```

### 2. Configure GitLab CI/CD Variables

In **Settings > CI/CD > Variables**, add:

| Variable | Value | Scope | Options |
|----------|-------|-------|---------|
| `CONFLUENCE_DOMAIN` | `acme-corp.atlassian.net` | All environments | |
| `CONFLUENCE_EMAIL` | `you@acme-corp.com` | All environments | |
| `CONFLUENCE_TOKEN` | Your API token | All environments | Masked |
| `CONFLUENCE_SPACE` | `ENG-STAGE` | `Staging` | |
| `CONFLUENCE_SPACE` | `ENG` | `Production` | |

The `CONFLUENCE_SPACE` variable is defined twice -- once per environment scope. GitLab injects the correct value based on the `environment:` declared in each job.

### 3. Create GitLab Environments

In **Operate > Environments**, create two environments:

- **Staging**
- **Production**

These must exist before the first pipeline run so that environment-scoped variables resolve correctly.

### 4. Add the pipeline config

Copy `.gitlab-ci.yml` and `ccfm.yaml` to the root of your repository.

## How It Works

| Stage | Job | Trigger | Environment |
|-------|-----|---------|-------------|
| `lint` | `lint:markdown` | MR + default branch | -- |
| `plan` | `plan:staging` | MR only | Staging (`action: prepare`) |
| `plan` | `plan:production` | MR only | Production (`action: prepare`) |
| `apply` | `apply:staging` | Default branch (auto) | Staging |
| `apply` | `apply:production` | Default branch (manual) | Production |

- **Merge request** -- lint runs first, then plan jobs show what would change in both staging and production.
- **Push to default branch** -- staging applies automatically. Production is a manual gate (click the play button in the pipeline UI).

## Why `action: prepare` on plan jobs?

Adding `environment: { name: Staging }` to a job without `action: prepare` causes GitLab to create a deployment record every time the job runs. On MR pipelines this produces phantom "deployments" that clutter the Environments page and can trigger unwanted notifications.

Using `action: prepare` tells GitLab to resolve environment-scoped variables (so `CONFLUENCE_SPACE` gets the correct value) without creating a deployment record. This is the recommended approach for read-only or informational jobs.

See [GitLab docs: deployment safety](https://docs.gitlab.com/ee/ci/environments/#environment-action) for details.

## Bootstrap with `ccfm init`

The `.ccfm-init` hidden job runs `ccfm --config ccfm.yaml init` as a `before_script`. This idempotently creates the management pages ccfm needs in the target Confluence space. Because every plan and apply job uses `extends: .ccfm-init`, the bootstrap happens automatically on the first run and is a no-op thereafter.

State is managed remotely in Confluence, so there are no local state files to pass between jobs via artifacts.

## Troubleshooting

### Protected variables don't resolve on feature branches

GitLab CI/CD variables marked as **Protected** are only injected into pipelines running on protected branches or tags. If your plan jobs run on MR pipelines from unprotected feature branches, ccfm will fail because `CONFLUENCE_DOMAIN`, `CONFLUENCE_TOKEN`, or `CONFLUENCE_SPACE` are empty.

**Fix:** In **Settings > CI/CD > Variables**, uncheck the **Protected** flag on all ccfm-related variables. Alternatively, protect the feature branches that need access to these variables.

### Environment name casing must match variable scopes

GitLab matches environment-scoped variables by exact name, including casing. If a variable is scoped to `Staging` (capital S) in **Settings > CI/CD > Variables**, the job must declare `environment: { name: Staging }` -- not `staging` (lowercase). A case mismatch means the scoped variable won't resolve and ccfm will receive an empty `CONFLUENCE_SPACE`.

---

See [gitlab-ci/README.md](../README.md) for an overview of all GitLab CI patterns.
