# Derive plugins

Plugins for Claude Code, Codex and Cursor that connect your coding agent to [Derive](https://derivecode.com): your business's data, sign-in and hosting. Install one, then say "build me an app that…".

Each plugin connects the agent to Derive's MCP server at `https://app.derivecode.com/mcp` and gives it the `build-and-deploy` skill. You sign in to Derive the first time the agent uses it. Building and deploying needs a Derive account whose business is allowed to deploy.

## Install

- **Claude Code:** `/plugin install derive --marketplace safikmomin/derive-plugins`. Or, from a shell: `claude plugin marketplace add safikmomin/derive-plugins`, then `claude plugin install derive@derive`.
- **Codex:** `codex plugin marketplace add safikmomin/derive-plugins`, then install Derive from the Plugins Directory.
- **Cursor:** add this repository as a plugin marketplace, then install Derive.

## About this repository

This repository is a copy. The plugins are developed in Derive's main repository and copied here by `scripts/sync-plugins.sh`, so changes made here directly are overwritten.
