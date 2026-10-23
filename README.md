# Storeflight for Claude Code

A Claude Code plugin for Storeflight, the Mac app that reads App Store Connect and tells you what blocks your app's next release.

The plugin adds:

- **The `storeflight` MCP server**, with read-only tools: `list_apps`, `check_app`, `get_fix_prompt` and `get_listing`.
- **The `release-check` skill**, which walks Claude through checking a release, fixing what blocks it with your own App Store Connect tools, and checking again until nothing blocks it. Claude asks before changing anything in App Store Connect.

## Requirements

- **The Storeflight app**, installed in `/Applications`. The plugin runs the command-line tool inside it, at `/Applications/Storeflight.app/Contents/Helpers/storeflight`.
- **Storeflight Pro**, to check your own apps. Without it you can try everything with three made-up sample apps (see below).
- An App Store Connect API key, added in the Storeflight app.
- To let Claude make fixes, an App Store Connect tool of your own: the App Store Connect API, the `asc` CLI, fastlane `deliver`, or an App Store Connect MCP server. Storeflight itself never changes anything.

## Install

In Claude Code:

```
/plugin marketplace add chansencc/storeflight-claude-plugin
/plugin install storeflight@storeflight
```

Or from a terminal:

```bash
claude plugin marketplace add chansencc/storeflight-claude-plugin
```

```bash
claude plugin install storeflight@storeflight
```

Restart Claude Code, then ask something like "Is my app ready to submit?" or run `/storeflight:release-check`.

## Try it without Pro

The `--sample` flag uses three made-up apps and needs no API key or Pro:

```bash
/Applications/Storeflight.app/Contents/Helpers/storeflight check sample-tidewater --sample
```

To give Claude a sample server to try the loop with:

```bash
claude mcp add storeflight-sample -- /Applications/Storeflight.app/Contents/Helpers/storeflight mcp --sample
```

## If Storeflight isn't in /Applications

Move the app to `/Applications`, or add a server that points to your copy and turn off the plugin's `storeflight` server in `/mcp`:

```bash
claude mcp add storeflight -- ~/Applications/Storeflight.app/Contents/Helpers/storeflight mcp
```

## API usage

Each check reads App Store Connect fresh and uses about 20–30 of Apple's 3,600 hourly API requests per key.
