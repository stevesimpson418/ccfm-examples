# Standalone — Multi Environment
#
# Required environment variables:
#   CONFLUENCE_DOMAIN  — your Confluence domain (e.g. acme-corp.atlassian.net)
#   CONFLUENCE_EMAIL   — email address for the Confluence API
#   CONFLUENCE_TOKEN   — Confluence API token
#
# CONFLUENCE_SPACE is set per-target to switch between staging and production.
#

.PHONY: install init-staging plan-staging deploy-staging init-production plan-production deploy-production

install:
	pip install ccfm-convert

init-staging:
	CONFLUENCE_SPACE=ENG-STAGE ccfm --config ccfm.yaml init

plan-staging:
	CONFLUENCE_SPACE=ENG-STAGE ccfm --config ccfm.yaml deploy --directory docs --plan

deploy-staging:
	CONFLUENCE_SPACE=ENG-STAGE ccfm --config ccfm.yaml deploy --directory docs --changed-only --archive-orphans

init-production:
	CONFLUENCE_SPACE=ENG ccfm --config ccfm.yaml init

plan-production:
	CONFLUENCE_SPACE=ENG ccfm --config ccfm.yaml deploy --directory docs --plan

deploy-production:
	CONFLUENCE_SPACE=ENG ccfm --config ccfm.yaml deploy --directory docs --changed-only --archive-orphans
