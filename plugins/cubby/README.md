# Cubby

Build, deploy and share web apps on [Cubby](https://cubbyapps.com), the universal dev server
for AI-built applications, from your agent. The plugin bundles two things: Cubby's MCP
server (`https://mcp.cubbyapps.com/mcp`), which gives your agent Cubby's tools, and the
build-on-cubby skill, which teaches it the build flow: plan the app, scaffold it, write it
from Cubby's docs, check it, deploy it, and hand you a live, private link.

## Use it

Connect Cubby once. In Claude Code, run `/reload-plugins` after installing from the shell (it
loads the plugin into the open session, with no restart), then run `/mcp`, pick the Cubby
server and choose Authenticate. On claude.ai or in Cowork, connect Cubby on this plugin's
Connectors tab. Either way you sign in with your Cubby account in your browser. On claude.ai,
start a new chat after adding the plugin: a chat that is already open doesn't load a newly
added plugin. Claude Code and claude.ai each install the plugin on their own.

Then ask for what you need, for example: "build a PTO tracker for my team and put it
online". Your agent plans the app with you, builds it, deploys it and sends you the link.
Later, ask it to share the app with a colleague's email address, connect it to a system you
already use, or change it.

## Data

The plugin itself stores nothing and runs no code. Your agent talks to Cubby's MCP server at
mcp.cubbyapps.com and sends it what building your apps takes: app names and descriptions,
the source files it writes, and the email addresses you ask it to share with. An agent that
can run commands may send those source files with one upload command Cubby gives it, which
sends the app's source folders (`src/`, `public/`, `design/tokens.json`, `README.md`) from
the folder it runs in, to the same address. The plugin sends nothing anywhere else.
Credentials for outside systems never pass through the plugin or the chat: you type them
into Cubby's vault through a one-time setup link. Your apps run on Cubby, at their own
addresses on cubby.pro.
