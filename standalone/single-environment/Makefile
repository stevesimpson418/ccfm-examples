# Standalone — Single Environment
#
# Required environment variables:
#   CONFLUENCE_DOMAIN  — your Confluence domain (e.g. acme-corp.atlassian.net)
#   CONFLUENCE_EMAIL   — email address for the Confluence API
#   CONFLUENCE_TOKEN   — Confluence API token
#

.PHONY: install init plan deploy deploy-full

install:
	pip install ccfm-convert

init:
	ccfm --config ccfm.yaml init

plan:
	ccfm --config ccfm.yaml deploy --directory docs --plan

deploy:
	ccfm --config ccfm.yaml deploy --directory docs --changed-only --archive-orphans

deploy-full:
	ccfm --config ccfm.yaml deploy --directory docs --archive-orphans
