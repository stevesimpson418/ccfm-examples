# Standalone — pip install
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
	ccfm init

plan:
	ccfm plan

apply:
	ccfm apply --auto-approve

apply-force:
	ccfm apply --auto-approve --force
