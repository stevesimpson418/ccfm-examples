# Standalone — Single Environment
#
# Required environment variables:
#   CONFLUENCE_DOMAIN  — your Confluence domain (e.g. acme-corp.atlassian.net)
#   CONFLUENCE_EMAIL   — email address for the Confluence API
#   CONFLUENCE_TOKEN   — Confluence API token
#

.PHONY: install init plan apply apply-force

install:
	pip install ccfm-convert

init:
	ccfm --config ccfm.yaml init

plan:
	ccfm --config ccfm.yaml plan --directory docs

apply:
	ccfm --config ccfm.yaml apply --directory docs --auto-approve

apply-force:
	ccfm --config ccfm.yaml apply --directory docs --auto-approve --force
