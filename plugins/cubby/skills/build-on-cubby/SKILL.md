---
name: build-on-cubby
description: Build and deploy web apps with Cubby (cubbyapps.com), the universal dev server for AI-built applications, through Cubby's MCP tools. Use when the user wants an app, internal tool, tracker, dashboard or small site built and put online; wants a Cubby app changed, redeployed, fixed, shared, connected to another system, backed up or deleted; asks about their Cubby apps or workspace; or mentions Cubby.
---

# Build on Cubby

You are the builder; Cubby is where the app runs. Cubby builds each app against one pinned
template, gives it real sign-in and its own Postgres, and hosts it at a real address,
private by default. Work through the flow below in order.

## Before you start: connect Cubby

Cubby's tools come from its MCP server, `https://mcp.cubbyapps.com/mcp`. If they are missing,
or a call asks for authentication, the user has to connect. Signing in happens in their
browser, so tell them exactly what to do, then wait until they say they are signed in:

- **Claude Code** (with this plugin): if the plugin was just installed from the shell, run
  `/reload-plugins` or restart Claude Code (`claude --continue` keeps the conversation); an
  install from the `/plugin` panel loads it for you. Then run `/mcp`, pick the Cubby server
  and choose Authenticate.
- **claude.ai or Cowork**: open Customize → Plugins → Cubby and connect Cubby on the
  plugin's Connectors tab (on Team and Enterprise plans, an Owner adds the connector for the
  organization first).
- **Any other agent**: add `https://mcp.cubbyapps.com/mcp` as a remote HTTP MCP server. It
  signs in with OAuth in the browser; there is no API key.

With the root URL the user picks the workspace at sign-in; each workspace also has its own
URL, `https://mcp.cubbyapps.com/w/<workspaceId>/mcp`, printed on the dashboard's Apps page
(workspaces.md#workspaces.connections).

## The build flow

1. **`whoami`: who and where.** Call it first and tell the user the account, workspace and
   plan it names: the signed-in account, the one workspace this connection works in (by its
   name, with its kind and id) and that workspace's plan. Every tool acts there and none can
   reach another (workspaces.md#workspaces.connections). If it says the name and plan are
   unavailable, tell the user the workspace id it gives instead. For an app that already
   exists, call `list_apps` next and use its id.

2. **`plan_app`: before every new app.** Call it with the user's description (a sentence or
   two) and, if they have one, a name (a lowercase-hyphen slug). It returns the name's
   availability, a sketched schema to refine, likely connections, the plan's app headroom,
   which chat tools to declare, the doc sections to read before writing code (each by id,
   heading and why, never its text) and the next step. Before you write code, fetch each
   section `plan_app` names with `docs` and read it whole: the sections are verified against
   the template's pinned versions, and public tutorials and memory are not. If it says
   `scaffold` would be refused, do not call `scaffold` or fetch the sections yet; tell the
   user the options it names. Agree the name and the main screens with the user.

3. **`scaffold`: create the app.** Call it with the name (add `database: false` only if the
   app stores nothing). It creates the app on Cubby from the pinned template, with its own
   icon already generated, and returns the app id, the file tree, where to edit first and
   the house rules. A refused `scaffold` creates nothing.

4. **Write the app from Cubby's docs only.** Write every framework and SDK line from the
   sections you fetched with `docs`, never TanStack Start idioms from memory or public
   tutorials (house-rules.md#house-rules.version-lockstep). As you write:
   - Keep data access in server functions: the app's own Postgres through the SDK's `db`,
     outside systems through `connections.<name>`, identity through `requireUser()`
     (house-rules.md#house-rules.sdk-only).
   - Pages are private by default and sign-in is already built: never build a login. Name
     the user with `requireUser(getRequest())` in every server function that touches private
     data (auth.md#auth.require-user). One that creates, changes or deletes data uses
     `requireUser(getRequest(), { access: "edit" })` instead, so a view-only share cannot
     change anything; map its `CubbyAccessError` to the response status as the section shows
     (auth.md#auth.access).
   - Declare the chat tools as you build each feature (agent-tools.md#agent-tools.doctrine):
     a read tool for each server function that lists or reads app data, and a mutation tool
     with a confirm phrase for each action a colleague could ask an assistant to do. Each is
     one inline `agentTool(...)` line in the tools array of `src/agent/tools.ts`, over an
     existing server function (agent-tools.md#agent-tools.declaring) that a page also
     imports, or the tool fails at runtime (agent-tools.md#agent-tools.client-reachable).
     Never declare tools for sign-in, internal helpers or anything that returns secrets;
     scope reads of other users' data by `user.id` or declare them `access: "edit"`
     (agent-tools.md#agent-tools.read-scoping).
   - Style through `design/tokens.json` (styling.md#styling.tokens-pipeline). The template
     ships no chart library and `package.json` cannot be sent, so draw charts as your own
     components, rendered client-only (charts.md#charts.client-only).
   - Keep module scope free of work that can fail, and keep `requireUser()` and database
     calls out of the root route: every deploy boots the app before promoting it
     (deploy.md#deploy.boot-probe).

5. **Check before you build: stage, then `check_app`.** `check_app` reads the files stored
   on Cubby, so stage your files first; staging stores them as the app's source and builds
   nothing. With a shell, `deploy_app` with `files: []` and `upload: true` gives one command
   that uploads them from disk: run it from your app's folder (deploy.md#deploy.upload). In
   a chat, `deploy_app` with `build: false` stores the files you send in the call (one-line
   base64, see Sending files). Then call `check_app`: each violation comes with the doc
   section that fixes it, and the result lists the chat tools the app declares (compare it
   with your plan). Fix, stage, check again. Two rules also block every build: a server
   function or a server route anyone can call without signing in (auth.md#auth.require-user).

6. **Build with `deploy_app`.** Call `deploy_app` without `build: false`: it builds the
   stored files plus the files in the call (only files you wrote or changed; an empty list
   builds what is stored). Cubby installs, builds, checks the bundle size and boots the app
   before promoting it (deploy.md#deploy.build); the previous version serves until then. The
   build does not typecheck: a type error that still compiles ships.

7. **Confirm it is live with `list_apps`.** A build is queued: `deploy_app` returns a
   deployment id (or the URL itself when it builds inline). Call `list_apps` with the app id
   until the latest deployment reads live or failed (the reason follows it). The app, or a
   change to it, is live only when its latest deployment reads live or `deploy_app` returned
   its URL; never say so before. An app that was already live keeps its URL under hosts
   while a change is queued and after the change fails, so the URL alone proves nothing. If
   the build failed, fix what the reason names and build again. Once it is live, `get_logs`
   shows its runtime output.

8. **Tell the user.** See the last section.

When any tool refuses, read its teaching error and the doc section it includes before you
retry. A citation such as deploy.md#deploy.delta-files means: call `docs` with section
`deploy.delta-files`.

## Keep your own copy of the files

No tool returns an app's files. Cubby keeps the source, and your copy is the only one you
can edit from. The template's own files are already on Cubby: you only ever send files you
wrote or changed (deploy.md#deploy.delta-files).

- **A coding agent** (Claude Code, Cursor, Codex, any agent with a filesystem): keep the
  files you write in a folder that mirrors the app's paths (`src/routes/index.tsx`,
  `design/tokens.json`, …), edit them there, and send them from it with an upload (Sending
  files). Keep only files you wrote under its `src/` and `public/`. That folder is where you
  pick the app up next time.
- **A chat** (claude.ai, Claude Desktop): write each file in full in the conversation and
  send exactly those bytes. The conversation is the copy: to change the app later, continue
  in it, or rewrite whole files.

## Sending files

- **Paths.** A delta holds only files under `src/` or `public/`, plus `design/tokens.json`
  and `README.md`, spelled in exactly that case. Never send template plumbing
  (`package.json`, the lockfile, `vite.config.ts`, `wrangler.jsonc`, `tsconfig.json`,
  `src/router.tsx`, `src/components/ui/`) or generated files (`src/routeTree.gen.ts`,
  `src/tokens.css`): `deploy_app` refuses them (deploy.md#deploy.delta-files). You cannot
  add npm packages; build with what the template ships.
- **From disk, with a shell.** Call `deploy_app` with the app id, `files: []` and
  `upload: true`. It writes nothing and returns one command (tar piped to curl) that
  works once and expires within 10 minutes. Run it exactly as given from your app's folder (the
  one holding `src/`; from any other folder it sends nothing): it sends everything under
  `src/` and `public/`, plus `design/tokens.json` and `README.md` (never `.env*`, `*.pem`,
  `*.key` or `.git`), and Cubby stages it as `build: false` would, under the same path rules
  and limits. It answers JSON naming the
  staged files, or what went wrong and what to do next. A file there that a delta may not
  carry (template plumbing, a generated file, a dot-file, a link) refuses the whole upload.
  For the next change, ask for a new command (deploy.md#deploy.upload).
- **Encoding, in a chat.** Without a shell, send every file in the call with
  `encoding: "base64"`: the file's UTF-8 bytes, base64-encoded, on one line with no line
  breaks. Plain text can lose backslashes in transit with no error anywhere
  (deploy.md#deploy.file-encoding).
- **Too big for one call.** Stage it in pieces with `build: false`, or in several uploads,
  then build once (deploy.md#deploy.delta-files, deploy.md#deploy.upload).
- **Removing files.** Leaving a file out of a delta does not delete it. Add a file at
  `.cubby/remove` to the delta, listing the paths to delete, one per line
  (deploy.md#deploy.remove-files). In an upload, delete those files from your folder too (a
  file both sent and listed refuses the upload), add `.cubby/remove` to the command's list
  of paths, and delete `.cubby/remove` once that upload has staged.

## Changing an existing app

`list_apps` finds it: its id, status, URL, latest deployment and any staged files. Edit your
copy, then stage, check and build as above: an upload sends your folder as it is, and in a
chat you send only the files you change. Without the files (a new chat), rewrite each file
you change in full; Cubby cannot hand them back. No
tool rolls a deploy back: to undo a change, deploy the earlier files again
(deploy.md#deploy.versioning).

An app keeps the template it was created with, and `list_apps` with its id shows when that
was. One created before 2026-09-25 may predate the current sign-in guard and `/signin`
route, and its SDK may ignore `{ access: "edit" }` and have no `CubbyAccessError`
(`deploy_app` cannot update an app's SDK). So step 4's sign-in and edit-check points may not
hold for it: before you change such an app, read auth.md#auth.existing-apps, and authorize
its writes with its own data (auth.md#auth.rules).

## Connections, data and the other tools

- **Outside systems: `add_connection`.** When the app's server must call an existing API or
  database, call `add_connection` with the app id, the name the code will use
  (`connections.<name>`) and the type, `postgres` or `generic_http`. It returns a
  single-use setup link that expires in 15 minutes, where the user types the credentials
  into Cubby's vault. Never ask for, accept or pass a key, password or connection string;
  if one appears in chat, tell the user to rotate it and use the link
  (connections.md#connections.setup-link). Use connections only in server functions
  (connections.md#connections.usage). If only this conversation needs the data, use your
  own connectors and keep the app out of it. `list_connections` shows connections and their
  ids without their secrets; `remove_connection` deletes one by its id (it needs
  `confirm: true`; to rotate, remove and add again).
- **The app's database: `db`.** It works once the app's database exists, after the first
  deploy. `action: "snapshot"` saves a restore point (take one before a risky change);
  `action: "restore"` rolls back to one (destructive: it needs `confirm: true`, and the
  replaced state is saved as a snapshot first); `action: "export"` returns the tables and
  data as SQL, up to 16 MiB. Never build backup, restore or export into the app
  (data.md#data.lifecycle).
- **Icons: `generate_icon`.** Makes the icon pair `scaffold` generated again, replacing any
  icon there, including an uploaded one; the next deploy ships it (pwa.md#pwa.icons).
- **Deleting: `delete_app`.** It needs `confirm: true` and the app's exact name. Without the
  confirm flag it only reports what would be deleted, and a name that does not match deletes
  nothing. The URL stops serving at once; the database and files go after a 14-day recovery
  hold. Always show the user which app it is first.
- **Runtime output: `get_logs`.** Recent lines from the last hour, or from a time you give.
- **Docs: `docs`.** The index of every section id, or one section verbatim.

## Limits

| Limit | Value |
| --- | --- |
| Files in one `deploy_app` call | 100 |
| One file | 512 KiB |
| All files in one call | 2 MiB |
| One upload from disk, as sent | 4 MiB, and the three limits above |
| Paths in one `.cubby/remove` | 100 |
| An app's stored source | 1,000 files and 16 MiB |
| The built server bundle | 3,072 KiB gzip |
| Builds | 30 per hour, one at a time |
| Staging calls (`build: false` or `upload: true`) | 120 per hour |
| `db` calls | 60 per hour |
| `share` calls | 60 per hour |
| `get_logs` calls | 120 per hour |

Rates are per user and the same on every plan. A rate-limited call is refused with the
time it resets: tell the user; do not retry in a loop. Apps per workspace, new apps per day,
connections per app and builds per month depend on the workspace's plan: `plan_app` reports
the app headroom before you scaffold, and `add_connection` refuses when the plan has no
room. Builds per month are pooled across the workspace's apps and reset on the 1st (UTC).
When they run out, `deploy_app` refuses to build, naming the plan and the reset date; that
call stores nothing, but staging with `build: false` still works, so tell the user and build
after the reset or an upgrade. On Free, a personal workspace holds 1 app and creates at most
1 new app per rolling 24 hours, an organization on Free holds none, apps on Free have no
connections, and a workspace on Free has 40 builds a month. Upgrades:
https://app.cubbyapps.com/billing.

## Don'ts (the house rules)

Cubby's non-negotiable rules are in house-rules.md, and `check_app` checks many of them:

- Never write framework code from memory or tutorials, or another framework's idioms
  (Next.js, Remix, React Router). The stack is TypeScript, TanStack Start, Tailwind v4,
  shadcn/ui and `@cubby/sdk` (house-rules.md#house-rules.one-stack).
- No raw fetch, XMLHttpRequest or WebSocket calls, and no secrets in app code, tool calls or
  chat: the outside world comes only through the SDK (house-rules.md#house-rules.sdk-only).
- No hand-built login forms, sessions, JWT signing or OAuth flows
  (house-rules.md#house-rules.no-hand-rolled-auth), and no place to enter credentials inside
  the app (connections.md#connections.setup-link).
- No bindings in `wrangler.jsonc`, no `cloudflare:*` imports, and no direct use of Workers
  bindings, KV, Durable Objects, R2 or queues (house-rules.md#house-rules.one-stack).
- Never edit generated files (house-rules.md#house-rules.generated-files), change
  dependencies (house-rules.md#house-rules.dependencies) or add a Tailwind or PostCSS config
  (house-rules.md#house-rules.tailwind-once).
- Never generate tools wholesale, declare a raw-database or "run SQL" tool, or edit
  `src/routes/mcp.ts` or the well-known route
  (house-rules.md#house-rules.agent-tools-declared). Never add, remove or restyle the
  "Built on Cubby" badge (house-rules.md#house-rules.one-stack).

## Tell the user at the end

When the app is live (step 7), finish with:

1. **The link:** the app's URL, exactly as the tool shows it.
2. **How to open it:** in a browser; the first visit goes through Cubby's sign-in. The app
   is private by default: the workspace's members and the people it is shared with can open
   it. Ask the user to open it and try the signed-in pages, because you cannot sign in for
   them (auth.md#auth.sign-in).
3. **How to share it:** offer `share`. The user gives an email address and a level (view,
   edit or deploy; each includes the ones before it), and the person gets an email with the
   app's link. Someone without a Cubby account yet has 7 days to sign in with that address.
   Only workspace owners and admins can share, and there is no public link.
4. **Its chat door:** if the app declares tools, the user can add `<app URL>/mcp` as a
   custom connector so their assistant can use the app
   (workspaces.md#workspaces.app-endpoint).

For example: "Your PTO tracker is live at <the URL>. Open it and sign in with Cubby; only
your workspace and the people you share it with can get in. Want me to share it?"
