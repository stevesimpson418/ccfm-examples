# Multi-Space — Advanced Pattern
#
# Two doc trees in one repo, each deploying to a different Confluence space.
# Each config file specifies its own docs_root, so no --directory flag is needed.
#
# Required environment variables:
#   CONFLUENCE_DOMAIN  — your Confluence domain (e.g. acme-corp.atlassian.net)
#   CONFLUENCE_EMAIL   — email address for the Confluence API
#   CONFLUENCE_TOKEN   — Confluence API token
#

.PHONY: install init-api plan-api apply-api init-wiki plan-wiki apply-wiki init-all plan-all apply-all

install:
	pip install ccfm-convert

# --- API docs (ENG space) ---

init-api:
	ccfm --config ccfm-api.yaml init

plan-api:
	ccfm --config ccfm-api.yaml plan

apply-api:
	ccfm --config ccfm-api.yaml apply --auto-approve

# --- Wiki docs (WIKI space) ---

init-wiki:
	ccfm --config ccfm-wiki.yaml init

plan-wiki:
	ccfm --config ccfm-wiki.yaml plan

apply-wiki:
	ccfm --config ccfm-wiki.yaml apply --auto-approve

# --- Combined targets ---

init-all: init-api init-wiki

plan-all: plan-api plan-wiki

apply-all: apply-api apply-wiki
