<!-- Thanks for contributing to Fortress. Keep PRs focused; see CONTRIBUTING.md. -->

## What this changes

<!-- One or two sentences. Link the issue it closes, e.g. "Closes #123". -->

## Type

- [ ] Fingerprint patch (touches `patches/`)
- [ ] SDK / tooling / packaging
- [ ] Docs
- [ ] CI / infra

## Checklist

- [ ] `python tools/check_patches.py` passes
- [ ] `python -m pytest sdk/python/tests -q` passes (if SDK touched)
- [ ] If this touches `patches/`: it is **one file per patch**, added to `patches/series`, and uses
      only the `uxr-` switch prefix (no brand strings baked into the binary)
- [ ] Any limitation or partial fix is written down (no oversold "undetectable" claims)
- [ ] For a surface change: before/after value on Fortress vs stock Chrome is in the description
