# CCFM Examples

Example projects demonstrating real-world usage of
[ccfm-convert](https://github.com/stevesimpson418/ccfm-convert) — a CLI tool that
converts Markdown to Atlassian Document Format (ADF) and deploys pages to Confluence Cloud.

## Which example is right for me?

| Use case | Example |
| --- | --- |
| Team wiki / knowledge base in Git | [`wiki/`](wiki/) |
| Customer or user-facing product docs | [`customer-docs/`](customer-docs/) |
| Microservice docs on GitHub | [`code-docs-github/`](code-docs-github/) |
| Microservice docs on GitLab | [`code-docs-gitlab/`](code-docs-gitlab/) |

## Examples

| Directory | Scenario | Integration method |
| --- | --- | --- |
| [`wiki/`](wiki/) | Company knowledge base — team wiki in Git, CI deploys on every merge | GitHub Action |
| [`customer-docs/`](customer-docs/) | Product documentation — PR plan check, Docker deploy on release | PyPI + Docker |
| [`code-docs-github/`](code-docs-github/) | Microservice docs — ADRs, API reference, runbooks alongside code | GitHub Action |
| [`code-docs-gitlab/`](code-docs-gitlab/) | Microservice docs — ADRs, API reference, runbooks alongside code | Docker via GitLab CI |

Each example is self-contained: its own `ccfm.yaml`, CI pipeline config, and committed
state file(s).

## Quick start in 3 steps

### 1. Clone and choose an example

```bash
git clone https://github.com/stevesimpson418/ccfm-examples.git
cd ccfm-examples/wiki   # or customer-docs / code-docs-github / code-docs-gitlab
```

### 2. Configure your Confluence credentials

Edit `ccfm.yaml` with your space key, then export credentials:

```bash
export CONFLUENCE_DOMAIN=your-org.atlassian.net
export CONFLUENCE_EMAIL=you@example.com
export CONFLUENCE_TOKEN=your-api-token
```

### 3. Run a plan locally (no Confluence changes)

```bash
pip install ccfm-convert

# Preview what would be deployed — safe, read-only
ccfm --config ccfm.yaml --directory docs --plan
```

When you're ready to deploy for real, drop `--plan`:

```bash
ccfm --config ccfm.yaml --directory docs
```

## Installation

```bash
pip install ccfm-convert
ccfm --help
```

Or with Docker:

```bash
docker pull ghcr.io/stevesimpson418/ccfm-convert:latest
docker run ghcr.io/stevesimpson418/ccfm-convert --help
```

## Quick reference

```bash
# Deploy a directory
ccfm --domain acme.atlassian.net --email user@acme.com --token $TOKEN \
     --space WIKI --directory docs

# Preview what would deploy (no API calls)
ccfm ... --directory docs --plan

# Only deploy changed files
ccfm ... --directory docs --changed-only

# Archive pages for deleted markdown files
ccfm ... --directory docs --archive-orphans
```

See [ccfm-convert](https://github.com/stevesimpson418/ccfm-convert) for full documentation.
