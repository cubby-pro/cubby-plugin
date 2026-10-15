# Cubby for AI agents

This repository is Cubby's plugin for AI agents. It gives your agent Cubby's MCP server and
the build-on-cubby skill, so it can build web apps, deploy them to Cubby and send you a live,
private link. [Cubby](https://cubbyapps.com) is the universal dev server for AI-built
applications.

The repository is two things at once: a Claude Code plugin marketplace named `cubby` that
lists one plugin, `cubby`, and a skills source for any other agent.

## Install

Claude Code and claude.ai each install the plugin on their own.

### Claude Code

```bash
claude plugin marketplace add cubby-pro/cubby-plugin
claude plugin install cubby@cubby
```

Inside a session, run `/plugin marketplace add cubby-pro/cubby-plugin`, then
`/plugin install cubby@cubby` and install it from the panel, which loads it. After a shell
install, run `/reload-plugins` in Claude Code: it loads the plugin into the session you have
open, with no restart, and keeps the conversation (restarting Claude Code also loads it).
Then run `/mcp`, pick the Cubby server and choose Authenticate. You sign in to Cubby in your
browser.

### claude.ai and Cowork

Open Customize → Plugins → Add → Add marketplace and paste
`https://github.com/cubby-pro/cubby-plugin` into its address box, which is easy to miss.
Install Cubby, then connect Cubby on the plugin's Connectors tab. On Team and Enterprise
plans, an Owner adds the connector for the organization first. On claude.ai, start a new chat
after adding the plugin: a chat that is already open doesn't load a newly added plugin.

### Other agents

```bash
npx skills add cubby-pro/cubby-plugin
```

installs the build-on-cubby skill for the agents you pick. Then add Cubby's MCP server to your
agent as a remote HTTP server at `https://mcp.cubbyapps.com/mcp`. It signs in with OAuth in
your browser; there is no API key.

## Use it

Ask your agent for what you need, for example: "build a PTO tracker for my team and put it
online". It checks which Cubby account and workspace it is working in, plans the app, builds
it from Cubby's docs, deploys it and sends you the link. Later, ask it to share the app,
connect it to a system you already use, or change it.

## What's inside

```text
.claude-plugin/marketplace.json     the marketplace, cubby
plugins/cubby/
  .claude-plugin/plugin.json        the plugin, cubby
  .mcp.json                         Cubby's MCP server
  skills/build-on-cubby/SKILL.md    the build flow the agent follows
  README.md                         what the plugin does and what data it sends
```
