---
name: obscura-cdp
description: Interact with a lightweight local Obscura headless browser from pi. Use when you want Chrome-CDP-style page inspection and automation with much lower memory usage.
license: MIT
compatibility: Node.js 22+. Auto-downloads Obscura release binaries on supported macOS/Linux/Windows targets, or use OBSCURA_BIN.
---

# Obscura CDP

Obscura is a lightweight headless browser that speaks the Chrome DevTools Protocol, but without launching Chrome itself.

This skill intentionally **does not mimic the Chrome skill one-for-one**:
- it auto-starts a local Obscura daemon when needed
- it reconnects to the browser websocket per command instead of holding a per-tab daemon
- it focuses on the features Obscura actually supports well today: page creation, navigation, HTML/markdown inspection, JS eval, form filling, selector clicks, and raw CDP calls
- it is the **default first choice** when you want the lowest-memory browser path
- it does **not** rely on screenshots, because current Obscura releases do not implement `Page.captureScreenshot`

## Runtime

All commands are in:

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs
```

## Common commands

### Start or inspect the local daemon

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs start
skills/obscura-cdp/scripts/obscura-cdp.mjs status
skills/obscura-cdp/scripts/obscura-cdp.mjs stop
```

### Open and list pages

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs list
skills/obscura-cdp/scripts/obscura-cdp.mjs open https://example.com
```

### Compatibility self-check

Run this before spending time automating a new site:

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs check https://example.com
skills/obscura-cdp/scripts/obscura-cdp.mjs check --json https://example.com
```

The check classifies a site as:
- `compatible` → Obscura is a good default
- `risky` → Chrome fallback is safer
- `incompatible` → use Chrome / hybrid instead

`check --json` also exposes machine-readable fields such as:
- `heuristicVersion`
- `riskLevel`
- `issueCounts`
- `decision.shouldFallbackToChrome`

### Inspect a page

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs md   <target>
skills/obscura-cdp/scripts/obscura-cdp.mjs html <target> [selector]
skills/obscura-cdp/scripts/obscura-cdp.mjs eval <target> "document.title"
```

`md` is the preferred semantic snapshot. It uses Obscura's `LP.getMarkdown`, which is usually more useful than raw HTML.

### Interact with a page

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs nav   <target> https://news.ycombinator.com
skills/obscura-cdp/scripts/obscura-cdp.mjs click <target> "button.submit"
skills/obscura-cdp/scripts/obscura-cdp.mjs fill  <target> "input[name=q]" "pi agent"
skills/obscura-cdp/scripts/obscura-cdp.mjs type  <target> "hello"
skills/obscura-cdp/scripts/obscura-cdp.mjs loadall <target> ".load-more"
```

Prefer `fill` over `type` whenever you have a stable selector.

### Advanced

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs net <target>
skills/obscura-cdp/scripts/obscura-cdp.mjs evalraw <target> "DOM.getDocument" '{}'
```

## Environment variables

```bash
OBSCURA_BIN=/path/to/obscura
OBSCURA_PORT=9223
OBSCURA_STEALTH=1
OBSCURA_WORKERS=4
OBSCURA_PROXY=http://127.0.0.1:8080
OBSCURA_AUTO_INSTALL=0
OBSCURA_VERSION=v0.1.1
```

## Notes

- The skill auto-downloads an Obscura binary if one is not already available.
- Default port is `9223` so it does not collide with a typical Chrome remote-debugging session on `9222`.
- Target IDs are resolved by prefix as long as the prefix is unique.
- Use `check` on unfamiliar sites first. If it recommends Chrome, switch to the hybrid package instead of forcing Obscura.
- If you need a feature Obscura does not expose yet, use `evalraw` and verify the CDP method directly.
