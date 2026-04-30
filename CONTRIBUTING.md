# Contributing to pi-obscura-skill

Thanks for contributing.

## Before opening an issue

1. Run the compatibility self-check first:

```bash
node skills/obscura-cdp/scripts/obscura-cdp.mjs check https://example.com
```

2. If the site is reported as `risky` or `incompatible`, prefer opening a **compatibility** issue instead of a generic bug.
3. Include the exact command, URL, and output you saw.

## Local verification

Run the same checks used by CI:

```bash
node --check skills/obscura-cdp/scripts/obscura-cdp.mjs
node skills/obscura-cdp/scripts/obscura-cdp.mjs check https://example.com
```

If you touched compatibility heuristics, also test at least one known-problem page.

## Scope guidance

This repository is for the **Obscura-first** path.

Good fits:
- Obscura runtime improvements
- compatibility heuristics
- low-memory automation flows
- docs for when to stay on Obscura vs move to hybrid

Not a fit:
- Chrome-only fallback orchestration across backends
- hybrid routing behavior

Those belong in `pi-browser-hybrid-skill`.

## Pull requests

Please keep PRs focused and include:
- what changed
- why it changed
- how you verified it
- whether docs / release notes should mention it

Use the provided PR template.

## Release-related changes

If a change affects public installation or release behavior, also review:
- [RELEASE_CHECKLIST.md](./RELEASE_CHECKLIST.md)
- [ROADMAP.md](./ROADMAP.md)
