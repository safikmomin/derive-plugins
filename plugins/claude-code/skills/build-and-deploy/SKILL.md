---
name: build-and-deploy
description: Build an app on Derive and put it online. Use when the owner says "build me an app", "make an app that…", "deploy this", or asks to change, preview, roll back or debug an app on Derive.
---

# Build and deploy a Derive app

Derive holds the owner's business data. An app is a web app that runs on Derive hosting
and calls Derive's API as the signed-in member. Read the app's `AGENTS.md` before you change
it: it has the platform's rules.

## Choose the workspace first

One Derive connection works in every workspace the owner belongs to: each store, a
personal workspace, and their sandbox. Every tool that acts in one (`register_app`,
`start_deploy`, `install_app`, `find`, `query`, the change tools and the rest) takes a
required `workspace`, its `id`. When it isn't clear which workspace the owner means, call
`list_workspaces` first and ask them; never guess between two stores. Use the same
`workspace` for every call in a task, and what you may do there is the owner's role in it.
A Personal workspace has no store data (`not_a_store`); `not_a_member` means the id is
wrong, so call `list_workspaces` again.

## Build the app

1. Start from the Derive app template, `templates/app` in Derive's repository
   (github.com/safikmomin/derive-core). Its `@derive` packages aren't published to npm yet,
   so build the app inside a checkout of that repository: copy `templates/app` to a new
   folder under `templates/`, give it its own `name` in its `package.json` and its own `id`
   in `derive.app.json`, add the folder to `pnpm-workspace.yaml`, then run `pnpm install` at
   the repository root. If you can't reach the repository, tell the owner; don't rebuild
   the packages by hand.
2. Describe the app's data as a custom table in `derive.app.json`, and its scopes with a
   reason each. The owner approves the scopes when they install the app.
3. Build the screens with `@derive/ui` and read or write data through `@derive/sdk`. Never
   keep business data in the app.
4. Try it in the owner's sandbox (the workspace `list_workspaces` marks `sandbox`), which
   holds sample data. Never test against the real
   business's data unless the owner asks for a copy.
5. Check it: `pnpm build` must pass before any deploy.

## Put it online

1. Register the app once: call `register_app` with the contents of `derive.app.json`.
   Registering again updates it. Derive keeps the app secret; it's never shown to you.
2. Call `start_deploy` with the site name and `app` set to the manifest's `id`. It returns a
   single-use ticket and the exact command to run.
3. Run that command in the app's folder, such as `npx @derive/cli deploy --ticket <ticket>`.
   The ticket expires in 10 minutes; ask `start_deploy` for a new one if it does.
   If it fails, it prints `Error (<code>): …` and `Next: …`: do what the `Next:` line says.
   Exit code 1 is something you can fix (a failed build shows its own errors above that
   line); exit code 2 is Derive's problem, so wait and retry, or tell the owner.
4. Call `install_app` with the scopes the owner approves. Read each scope's reason to the
   owner first, and install only the ones they agree to.
5. Tell the owner the live URL the deploy printed.

Use `start_deploy` with a `preview` name to try a change without replacing the live app.
Use `rollback` to return to an earlier deploy, and `get_recent_errors` before guessing at a
bug.
`delete_site` removes a site and its previews for good; confirm with the person first.

## Apps whose source is with Derive (create_app)

- **Try a change on a branch.** `write_files` with `branch` (such as `new-colours`) commits
  there, and Derive builds it as a preview at `<site>--<branch>.derivesites.com`, without
  touching the live app. `deploy_status` with the commit says when it's live. A site has at
  most 3 previews, each deleted 7 days after its last change.
- **Make it live only when the owner says so:** `publish` with the branch moves `main` to it
  (`dry_run` lists the commits it brings in), and Derive builds `main`. If `main` changed
  since the branch started, it's refused with `branch_behind`: write the change again on
  top of `main`, then publish.
- `delete_branch` deletes a branch and its preview; confirm with the owner first unless
  it was published.
- **Undo:** `revert` undoes a change in the source with a new commit, and that's built and
  goes live like any change: use it to undo for good. `rollback` puts an earlier stored
  deploy back online in seconds without touching the source: use it when the live app is
  broken right now, then fix or `revert` the source, since the next change to `main`
  replaces the rollback.

## The rules

- Never ask the owner for the app secret, and never write it into code, logs or messages.
  The platform gives the app its credentials as bindings: `DERIVE_CLIENT_ID`,
  `DERIVE_CLIENT_SECRET`, `DERIVE_AUTH_URL` and `DERIVE_CORE_URL`.
- Never ask for an API key in chat. If the site needs one (an email or payments provider,
  say), ask the person to add `NAME=value` to `.env.production` in the site's folder
  themselves, and don't read or print its values. Every deploy sets exactly what's in that
  file as the site's secrets (`env.NAME`); `list_secret_names` shows their names.
- Only an owner can deploy an app, or install one into the org.
- Apps get no database of their own. Data lives in custom tables, reached through the API.
- Impactful changes (deleting core records, bulk changes) wait for the owner's approval.
  Show the owner the preview and call `approve` only when they say yes.
- Deploys are refused in a sandbox. Deploy in the business's own workspace (`workspace` set
  to its id), after the owner installs the app there.
