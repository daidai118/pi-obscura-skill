# pi-obscura-skill

[简体中文](./README.zh-CN.md) | [English](./README.en.md) | [日本語](./README.ja.md)

Use [Obscura](https://github.com/h4ckf0r0day/obscura) as a lightweight headless browser for Pi instead of driving a full Chrome instance.

Skill name in Pi: `obscura-cdp`

## Product positioning

- `pi-obscura-skill`: lightweight, low-memory, default first choice
- `pi-browser-hybrid-skill`: probe first, then fall back to Chrome when Obscura looks unsafe for the site

## Why this package exists

This package is the Obscura-oriented sibling of `chrome-cdp-skill`, but it is intentionally adapted to Obscura instead of copying Chrome assumptions blindly.

Key differences:

- much lower memory use than headless Chrome
- auto-starts a local Obscura daemon on demand
- no per-tab keepalive daemon
- uses Obscura strengths: markdown snapshots, HTML inspection, JS eval, selector clicks, form filling, navigation, and raw CDP access
- avoids screenshot-first workflows because current Obscura releases do not implement `Page.captureScreenshot`

## Installation

### Local path install, before pushing to GitHub

```bash
pi install /Users/daidai/ai/pi-obscura-skill
```

### Install from GitHub after publishing

```bash
pi install git:github.com/daidai118/pi-obscura-skill
```

### Install into project-local Pi settings

```bash
pi install -l git:github.com/daidai118/pi-obscura-skill
```

## Updating

Update just this package:

```bash
pi update git:github.com/daidai118/pi-obscura-skill
```

Update all non-pinned Pi packages:

```bash
pi update --extensions
```

If you want easy updates later, do **not** pin a git ref during install.

Good for auto-updates:

```bash
pi install git:github.com/daidai118/pi-obscura-skill
```

Pinned, so `pi update` will skip it:

```bash
pi install git:github.com/daidai118/pi-obscura-skill@v0.1.2
```

## Requirements

- Node.js 22+
- Either a supported Obscura release binary for auto-download, or a local Obscura build via `OBSCURA_BIN`

If `obscura` is already on your `PATH`, the skill uses it.
Otherwise it downloads the tested release binary automatically.

## Compatibility self-check

Before automating a new site, run:

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs check https://example.com
skills/obscura-cdp/scripts/obscura-cdp.mjs check --json https://example.com
```

Status meanings:

- `compatible` → use Obscura normally
- `risky` → Chrome fallback is safer
- `incompatible` → use the hybrid / Chrome route

The compatibility checker was specifically added because some sites render correctly in Obscura while others partially break. For example, `https://100t.xiaomimimo.com/` showed missing applied stylesheets, collapsed layout metrics, and failed interactive flow rendering during local testing.

## Commands

All commands use:

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs
```

### Lifecycle

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs start
skills/obscura-cdp/scripts/obscura-cdp.mjs status
skills/obscura-cdp/scripts/obscura-cdp.mjs stop
```

### Pages

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs list
skills/obscura-cdp/scripts/obscura-cdp.mjs open https://example.com
skills/obscura-cdp/scripts/obscura-cdp.mjs close <target>
skills/obscura-cdp/scripts/obscura-cdp.mjs nav <target> https://example.com
```

### Inspection

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs md   <target>
skills/obscura-cdp/scripts/obscura-cdp.mjs snap <target>
skills/obscura-cdp/scripts/obscura-cdp.mjs html <target> [selector]
skills/obscura-cdp/scripts/obscura-cdp.mjs eval <target> "document.title"
skills/obscura-cdp/scripts/obscura-cdp.mjs net  <target>
skills/obscura-cdp/scripts/obscura-cdp.mjs evalraw <target> "DOM.getDocument" '{}'
```

`md` / `snap` returns a markdown snapshot using Obscura's `LP.getMarkdown`.

### Interaction

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs click <target> "button.submit"
skills/obscura-cdp/scripts/obscura-cdp.mjs fill  <target> "input[name=q]" "pi agent"
skills/obscura-cdp/scripts/obscura-cdp.mjs type  <target> "hello world"
skills/obscura-cdp/scripts/obscura-cdp.mjs loadall <target> ".load-more" [ms]
```

Prefer `fill` over `type` when you have a stable selector.

## Project docs

- [Changelog](./CHANGELOG.md)
- [Contributing guide](./CONTRIBUTING.md)
- [Roadmap](./ROADMAP.md)
- [Release checklist](./RELEASE_CHECKLIST.md)

## Environment variables

```bash
OBSCURA_BIN=/path/to/obscura
OBSCURA_PORT=9223
OBSCURA_STEALTH=1
OBSCURA_WORKERS=4
OBSCURA_PROXY=http://127.0.0.1:8080
OBSCURA_AUTO_INSTALL=0
OBSCURA_VERSION=v0.1.1
OBSCURA_CHECK_SETTLE_MS=1500
```

## Design notes

### Why port 9223 by default?

Obscura itself defaults to `9222`, but `9222` is also the usual Chrome remote-debugging port. This package defaults to `9223` so both can coexist during migration.

### Why no screenshots?

Because current Obscura releases do not implement `Page.captureScreenshot`. Markdown and HTML inspection are the reliable path today.

### Why no per-tab daemon?

Chrome needed one in the original skill to suppress repeated debugging prompts and keep tab sessions warm. Obscura is already a local headless daemon, so this package simply reconnects to the browser websocket and re-attaches to targets per command.
