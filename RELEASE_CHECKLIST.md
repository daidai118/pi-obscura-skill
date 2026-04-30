# Release checklist

## Before tagging

- [ ] Update `package.json` and `package-lock.json` version
- [ ] Update README install snippets if the tag example changed
- [ ] Review `GITHUB_REPO_METADATA.md` if repo positioning changed
- [ ] Review `ROADMAP.md` and move completed items if needed
- [ ] Confirm issue / PR templates still match current workflow

## Local verification

```bash
node --check skills/obscura-cdp/scripts/obscura-cdp.mjs
node skills/obscura-cdp/scripts/obscura-cdp.mjs check https://example.com
```

Optionally test one known difficult site before tagging.

## Publish steps

```bash
git push
git tag vX.Y.Z
git push origin vX.Y.Z
gh release create vX.Y.Z --repo daidai118/pi-obscura-skill --title "vX.Y.Z" --notes "..."
```

## After publishing

- [ ] Confirm the GitHub release page looks correct
- [ ] Confirm the Release badge resolves to the new tag
- [ ] Confirm CI is green on `main`
- [ ] Test `pi install git:github.com/daidai118/pi-obscura-skill@vX.Y.Z`
