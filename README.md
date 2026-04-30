# pi-obscura-skill

> Lightweight, low-memory browser automation for Pi using [Obscura](https://github.com/h4ckf0r0day/obscura).

**Languages / 语言 / 言語**

- [简体中文](./README.zh-CN.md)
- [English](./README.en.md)
- [日本語](./README.ja.md)

**Pi skill name:** `obscura-cdp`

## Product positioning

- `pi-obscura-skill`: lightweight, low-memory, default first choice
- `pi-browser-hybrid-skill`: probe first, then automatically switch to Chrome when needed

## What this package is for

Use this package when you want the cheapest, simplest browser path first:

- inspect pages with HTML / Markdown / JS eval
- fill forms and click by selector
- keep memory use far below headless Chrome
- quickly decide whether a site is safe for Obscura before automating it

## Quick install

### Before pushing to GitHub

```bash
pi install /Users/daidai/ai/pi-obscura-skill
```

### After pushing to GitHub

```bash
pi install git:github.com/daidai118/pi-obscura-skill
```

### Install into the current project instead of global Pi settings

```bash
pi install -l git:github.com/daidai118/pi-obscura-skill
```

## Compatibility self-check

Before automating a new site, run:

```bash
skills/obscura-cdp/scripts/obscura-cdp.mjs check https://example.com
skills/obscura-cdp/scripts/obscura-cdp.mjs check --json https://example.com
```

Status meanings:

- `compatible` → use Obscura normally
- `risky` → Chrome fallback is safer
- `incompatible` → switch to `pi-browser-hybrid-skill`

## Updating later

Update just this package:

```bash
pi update git:github.com/daidai118/pi-obscura-skill
```

Update all non-pinned Pi packages:

```bash
pi update --extensions
```

> If you want easy future updates, install **without** pinning a git ref. If you install `@v0.1.0`, Pi treats it as pinned and skips automatic package updates.

## GitHub repository metadata

Suggested GitHub description and topics are in:

- [GITHUB_REPO_METADATA.md](./GITHUB_REPO_METADATA.md)

## Docs

- [简体中文文档](./README.zh-CN.md)
- [English docs](./README.en.md)
- [日本語ドキュメント](./README.ja.md)
