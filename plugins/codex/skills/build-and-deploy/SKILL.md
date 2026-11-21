---
name: build-and-deploy
description: Build an app on Derive and put it online, with Derive's tools alone. Use when the owner says "build me an app", "make an app that…", "deploy this", or asks to change, preview, publish, undo, roll back or debug an app on Derive.
---

# Build and deploy a Derive app

Derive holds the owner's business data. An app is a web app that runs on Derive hosting
and calls Derive's API as the signed-in member. **You build it with Derive's MCP tools
alone.** Derive keeps the app's source, builds every change and hosts it, so you need no
checkout, no shell, no CLI and no `npm`, and the owner installs nothing.

## Choose the workspace first

One Derive connection works in every workspace the owner belongs to: each store, a
personal workspace, and their sandbox. Every tool that acts in one takes a required
`workspace`, its `id`. When it isn't clear which workspace the owner means, call
`list_workspaces` first and ask them; never guess between two stores. Use the same
`workspace` for every call in a task, and what you may do there is the owner's role in it.
A Personal workspace has no store data (`not_a_store`); `not_a_member` means the id is
wrong, so call `list_workspaces` again.

Apps are made in the business's own workspace, not the sandbox (`create_app` is refused
there). Only an owner can create, change, deploy or install an app.

## Build the app

1. **Create it:** `create_app` with the app's name as the owner says it ("Cooler log") and
   what it does in a sentence. Derive copies its app template (sign-in with Derive, the
   Core SDK and Derive's UI kit, already wired up) into the app's own repo, gives it an ID
   from its name (`cooler_log`), and registers it. `dry_run` makes the checks first.
2. **Read it:** `read_file` on `AGENTS.md` first: it has the platform's rules and where
   things are. `list_files` with `recursive` shows the rest; `llms.txt` describes the Core
   API, and `derive.app.json` is the manifest.
3. **Describe its data** in `derive.app.json`: a custom table under `tables` (the template's
   is `readings`), and the scopes the app needs on the store's data (such as
   `departments:read`), each with a reason the owner will read. Its own tables come with
   every install, so leave their scopes out. Never keep business data in the app.
4. **Change it** with `write_files`: one call is one commit, with each file's whole new
   content (or `null` to delete it) and a message in plain words for the owner ("Add a
   column for the cooler's location"). Write related files together, in one call. Paths
   are from the app's root. Changing `derive.app.json` on `main` registers the app again.
5. **Watch the build:** every change to `main` is built by Derive (`npm ci`, then
   `npm run build`) and goes live a minute or two later. Call `deploy_status` with the
   commit `write_files` returned every 15-30 seconds until it says `live` (with the URL) or
   `failed`. A failed build shows the step and the end of its log: fix the code with
   `write_files`, which is built in turn. A failed build leaves the last live version
   serving.
6. **Install it:** `install_app` with the scopes the owner approves. Read each scope's
   reason to the owner first, and install only the ones they agree to. The app's tables
   are made in the workspace then.
7. **Tell the owner the live URL** `deploy_status` gave.

Use `history` for "what changed?", and `get_recent_errors` (with the app's site) before
guessing at a bug.

## Try a change first, then publish

- **Preview on a branch:** `write_files` with `branch` (such as `new-colours`) commits
  there, and Derive builds it as a preview at `<site>--<branch>.derivesites.com`, without
  touching the live app. `deploy_status` with the commit says when it's live. A site has
  at most 3 previews, each deleted 7 days after its last change.
- **Make it live only when the owner says so:** `publish` with the branch moves `main` to
  it (`dry_run` lists the commits it brings in), and Derive builds `main`. If `main`
  changed since the branch started, it's refused with `branch_behind`: write the change
  again on top of `main`, then publish.
- `delete_branch` deletes a branch and its preview; confirm with the owner first unless
  it was published.

## Undo: revert or rollback

- `revert` undoes a change in the source with a new commit, and that's built and goes
  live like any change: use it to undo for good.
- `rollback` puts an earlier stored deploy back online in seconds without touching the
  source: use it when the live app is broken right now, then fix or `revert` the source,
  since the next change to `main` replaces the rollback.

## Keys and secrets

Never ask for an API key in the chat, and never write one into a file (`.env*` files are
refused with `secrets_in_repo`). If the app calls another service with the owner's own key
(an email provider, say), call `request_secret` with the name the code reads
(`RESEND_API_KEY`) and why, in plain words with where to find it. Give the owner the link it
returns: they paste the value into Derive, and you never see it. The app reads it on the
server as `env.NAME`; `list_secret_names` on the app's site shows the names, and
`delete_secret` removes one.

## The rules

- Never ask the owner for the app secret, and never write it into code, logs or messages.
  The platform gives the app its credentials as bindings: `DERIVE_CLIENT_ID`,
  `DERIVE_CLIENT_SECRET`, `DERIVE_AUTH_URL` and `DERIVE_CORE_URL`.
- Apps get no database of their own. Data lives in custom tables, reached through the API.
- Impactful changes (deleting core records, bulk changes) wait for the owner's approval.
  Show the owner the preview and call `approve` only when they say yes.
- Try the app against the owner's sandbox (the workspace `list_workspaces` marks
  `sandbox`), which holds sample data, by installing it there too. Never test against the
  real business's data unless the owner asks for a copy.
- `delete_site` removes an app's site and its previews for good; confirm with the owner
  first.
- When a tool fails, its answer has a code and a `next` step: do what it says.

## If you have a shell and prefer git (optional)

You never need this. An owner's agent with a shell may call `get_repo_access` for an
hour-long write token for the app's one repo, then clone, commit and push with plain git,
passing the token in a header (the answer has the exact clone command) and never writing it
into a file, a remote's URL or a message. Pushes are built like `write_files` changes
(`main` live, other branches previews), but skip its checks: commit no `.env` file or build
output, and after changing `derive.app.json` call `register_app` with it.

## Standalone sites (not apps)

Derive's CLI is for standalone sites and for Derive's staff, never for apps. A site that
isn't a Derive app (no sign-in with Derive, no Core data) is built on the
person's own machine and uploaded with Derive's CLI: `start_deploy` returns a ticket and the
exact command to run in the site's folder, such as `npx @derive/cli deploy --ticket <ticket>`.
Its keys go in `.env.production` in that folder, which the person edits themselves. Apps
never use the CLI.
