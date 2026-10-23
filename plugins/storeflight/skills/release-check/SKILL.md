---
name: release-check
description: Check whether an app's next App Store release is ready to submit, using Storeflight, and fix what blocks it. Use when the user asks "is my app ready to submit", "what's blocking my release", "check my App Store listing", "fix my App Store Connect issues", "get my app ready for review", or names an app and asks about its next App Store release.
---

# Storeflight release check

Storeflight reads App Store Connect and reports what blocks an app's next release. Its MCP server, `storeflight`, is read-only: it never changes anything. You make the fixes with the user's own App Store Connect tools, then use Storeflight to confirm them.

## Tools

The `storeflight` MCP server has four tools:

- `list_apps`: the apps the user's App Store Connect API key can see, with name, bundle ID and App Store Connect app ID.
- `check_app(app, platform?)`: a `headline`, each platform's version and App Store Connect state (`status`), `counts`, the `issues` (blockers and warnings, each with its fix), the `checkByHand` items, and what to do `next`.
- `get_fix_prompt(app, platform?)`: the rules for editing App Store Connect, the character limits, each issue with its fix, and what to leave to the person. It reuses the last check in this session.
- `get_listing(app, platform?, locale?)`: the current listing text per locale. It also reuses the last check.

`app` takes a name, a bundle ID or an App Store Connect app ID. `platform` is `iOS`, `macOS`, `tvOS` or `visionOS`; leave it out to cover every platform.

## The loop

1. **Find the app.** Call `list_apps` unless the user already gave a bundle ID or app ID. If several apps match their words, ask which one.
2. **Check it.** Call `check_app` and read the `headline` and `status` for each platform's state:
   - "Prepare for Submission" (headline "N blockers before you can submit" or "Ready to submit"): not submitted yet. This is what you can fix.
   - Rejected (headline "Rejected, …"): App Review rejected it. Fix what's listed; the user reads App Review's notes and resubmits.
   - "Waiting for Review", "In Review" or approved and waiting for release: it's with Apple. Don't change that version's listing.
   - "Live, no version in progress": the live version was checked. There's nothing to submit until the user creates a new version.

   An app can have platforms in different states; the headline names the one furthest from release. Pass `platform` to look at one at a time.
3. **Read the rules.** Call `get_fix_prompt` and follow its "How to work" rules and character limits: app name 30, subtitle 30, promotional text 170, keywords 100 (comma-separated, no spaces), description 4,000, What's New 4,000. Write each locale's text natively in that language.
4. **Read before you edit.** Call `get_listing` for the locale before changing a field, and change only what an issue asks for.
5. **Ask, then fix.** Show the user the exact changes you plan, field by field and locale by locale, and wait for a clear yes before changing anything in App Store Connect. Then make the changes with the tools the user has for App Store Connect: the App Store Connect API, the `asc` CLI, fastlane `deliver`, or an App Store Connect MCP server. If they have none, write out the exact text to paste, grouped by locale and field.
6. **Check again.** Call `check_app` again to confirm the fixes took. Repeat steps 3–6 until `counts.blockers` is 0.

## When to stop and report

Stop and tell the user what's left, instead of looping, when what remains is:

- **Check by hand**: the `checkByHand` items, also the "For the human" list in `get_fix_prompt`, such as App Privacy labels and whether screenshots show the current UI. The App Store Connect API can't check these, so ask the user to confirm them before submitting.
- **Needs a person**: uploading and selecting a build in Xcode, screenshots that have to be captured, agreements, tax and banking, or anything else you can't do with the user's tools.
- **Waiting on App Review**: the version is waiting for review, in review or approved, or an in-app purchase is waiting for review.

If a fix you made doesn't show up on the next check, App Store Connect may not have caught up yet, or the fix went to the wrong locale or platform. Look at `get_listing` once before trying again, and don't loop on the same issue.

Never submit the app for review, release it, or change pricing unless the user asks. If you add or edit an in-app purchase or subscription localization, tell the user which products changed: that sends them back to App Review and they'll need to submit them.

Warnings don't block a release. Fix them only if the user wants to.

## API budget

Each `check_app` reads App Store Connect fresh and uses about 20–30 of Apple's 3,600 hourly API requests per key. `get_fix_prompt` and `get_listing` reuse the last check, so they cost nothing extra. Check again once after a batch of fixes, not after every field. If App Store Connect returns a rate-limit error, wait before checking again.

## Errors

**Storeflight Pro.** The MCP tools and the `storeflight` command are part of Storeflight Pro. Without it, every tool returns an error pointing to Upgrade to Pro (the command exits with code 6). Tell the user:

- To upgrade, open Storeflight and choose Storeflight → Upgrade to Pro…
- If they already have Pro, or just renewed, open Storeflight once so the command can see their plan.
- To try it first with three made-up sample apps and no API key, run the command with `--sample`:

  ```bash
  /Applications/Storeflight.app/Contents/Helpers/storeflight list --sample
  /Applications/Storeflight.app/Contents/Helpers/storeflight check sample-tidewater --sample --prompt
  ```

  Or add a sample server for Claude Code and use it the same way as `storeflight`:

  ```bash
  claude mcp add storeflight-sample -- /Applications/Storeflight.app/Contents/Helpers/storeflight mcp --sample
  ```

  The sample apps aren't real, so don't try to fix them in App Store Connect.

**No API key.** Storeflight shares the API key the user added in the Storeflight app. If there's none, ask them to open Storeflight and add one. The command also reads `ASC_*` environment variables or `~/.asc/config.json`.

**Storeflight isn't in /Applications.** If the `storeflight` server fails to start or its tools are missing, check that the app is there:

```bash
ls /Applications/Storeflight.app/Contents/Helpers/storeflight
```

- If it's missing, the user needs to install Storeflight from the Mac App Store, then restart Claude Code.
- If they keep the app somewhere else, such as `~/Applications`, either move it to `/Applications` or add a server that points to their copy, then restart Claude Code:

  ```bash
  claude mcp add storeflight -- ~/Applications/Storeflight.app/Contents/Helpers/storeflight mcp
  ```

  Then turn off this plugin's copy of the server in `/mcp` so the two don't overlap.
