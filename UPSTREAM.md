# Upstream provenance

- Upstream repository: <https://github.com/aqlaboratory/openfold-3>
- Upstream base commit: `15869c1c57d3a767ee5837016d4a5db25196f949`
- Upstream base date: 2026-09-01
- Upstream base version: `0.5.0`
- This repository: <https://github.com/LambdaLabsML/openfold-3>
- License: Apache-2.0 (unchanged from upstream)

## Relationship to upstream

This is a standalone repository — not a GitHub fork — carrying the full upstream
commit history. It exists to provide a quick-start guide for **training OpenFold3
on Lambda GPU infrastructure**, produced jointly by the OpenFold consortium and
Lambda.

The complete upstream source tree is retained unmodified so that the OpenFold3
model, training recipe, configuration system, and data pipeline remain runnable
exactly as upstream intends.

## Changes relative to upstream

- `README.md` — replaced with the OpenFold x Lambda training quick-start guide.
  The original upstream README is preserved verbatim as `README_UPSTREAM.md`.
- `NOTICE` — added, per Apache-2.0 attribution practice.
- `UPSTREAM.md` — added (this file), recording the base commit and changes.

No changes have been made to the `openfold3/` package, training code,
configuration defaults, or dependency specifications.

## Syncing with upstream

```bash
git remote add upstream https://github.com/aqlaboratory/openfold-3.git   # once
git fetch upstream
git merge upstream/main
```

Because the full upstream history is present, these merges stay clean. Conflicts
should be limited to `README.md`; keep this repository's version and re-check
that the quick-start instructions still match upstream's CLI.
