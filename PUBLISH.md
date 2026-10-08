# Publishing this repository

Repository: `cubby-pro/cubby-plugin`

That line is the one place this repository's slug is defined. It is a working name: confirm
it before the first publish. The slug also appears in README.md's install commands, and a
test in Cubby's platform repository fails if any file here names a different slug, or names
this one anywhere but these two files.

## Before the first publish

1. **License.** MIT, copyright "The Cubby Authors" (ruled 2026-10-07). The same `LICENSE`
   text sits at this repository's root and at the plugin's root (`plugins/cubby/LICENSE`, so it
   travels with every install and the claude.ai zip), and `plugins/cubby/.claude-plugin/plugin.json`
   says `"license": "MIT"`: Anthropic's plugin directory lists a plugin only with one of the two.
2. **Slug.** Confirm the line above. If it changes, change it here and in README.md.
3. **Names.** The marketplace `cubby` and the plugin `cubby` are permanent: people install
   `cubby@cubby`, so never rename either. Change `displayName` for a different label.

## Publish

1. Create the public repository at the slug above and make the contents of this directory
   its root, with `.claude-plugin/` at the top.
2. From a clean clone, run `claude plugin validate .` and
   `claude plugin validate plugins/cubby`; each ends with `✔ Validation passed`.
3. Test both install paths from GitHub in a throwaway Claude Code config, so nothing lands
   in your own settings (the pre-publish local proof used the same three variables):

   ```bash
   T=$(mktemp -d)
   (export HOME="$T" CLAUDE_CONFIG_DIR="$T/.claude" CLAUDE_CODE_PLUGIN_CACHE_DIR="$T/.claude/plugins"
     claude plugin marketplace add cubby-pro/cubby-plugin
     claude plugin install cubby@cubby
     claude plugin details cubby)
   npx skills add cubby-pro/cubby-plugin --list
   ```

   `details` lists the build-on-cubby skill and the cubby MCP server; the last command lists
   build-on-cubby and installs nothing.
4. For claude.ai, zip the `plugins/cubby` folder and upload it in Customize → Plugins → Add →
   Upload plugin to check the skill loads and the Connectors tab shows Cubby.

## Each release

Raise `version` in `plugins/cubby/.claude-plugin/plugin.json`, validate both paths again,
and run `claude plugin tag plugins/cubby` to tag the release as `cubby--v<version>`.

## When the organization moves

GitHub redirects a transferred repository's old URL, but change the slug here and in
README.md in the same change anyway; the test names any file that still disagrees.
