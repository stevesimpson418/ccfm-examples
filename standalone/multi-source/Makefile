# Standalone — Multi Source
#
# Required environment variables:
#   CONFLUENCE_DOMAIN  — your Confluence domain (e.g. acme-corp.atlassian.net)
#   CONFLUENCE_EMAIL   — email address for the Confluence API
#   CONFLUENCE_TOKEN   — Confluence API token
#

.PHONY: install init-api plan-api deploy-api init-wiki plan-wiki deploy-wiki plan-all deploy-all

install:
	pip install ccfm-convert

init-api:
	ccfm --config ccfm-api.yaml init

plan-api:
	ccfm --config ccfm-api.yaml deploy --directory docs --plan

deploy-api:
	ccfm --config ccfm-api.yaml deploy --directory docs --changed-only --archive-orphans

init-wiki:
	ccfm --config ccfm-wiki.yaml init

plan-wiki:
	ccfm --config ccfm-wiki.yaml deploy --directory docs-wiki --plan

deploy-wiki:
	ccfm --config ccfm-wiki.yaml deploy --directory docs-wiki --changed-only --archive-orphans

plan-all: plan-api plan-wiki

deploy-all: deploy-api deploy-wiki
