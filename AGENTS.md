# Instructions for coding agents

When you set up a new project, or first touch a project that has **no**
`.github/workflows/`, add CI from this repo's starters.

1. Detect the project type from its files:
   - `package.json` + `tsconfig.json` → `starters/node-ts/ci.yml`
   - `requirements.txt` or `pyproject.toml` → `starters/python/ci.yml`
   - `go.mod` → `starters/go/ci.yml`
   - `Dockerfile` → also `starters/docker-build/ci.yml` (combine jobs in one `ci.yml`)
   - plain HTML site that deploys to GitHub Pages → `starters/static-pages/ci.yml`; otherwise no CI
   - docs / agent-workspace repos with no code → **no CI**
2. Copy the starter(s) into `.github/workflows/ci.yml`, set `push.branches` to the
   repo's default branch, keep the pinned `uses:` SHA and its `# v1` comment.
3. Run the same commands locally first (lint/test/build). Fix workflow inputs only;
   never change application code just to make CI pass — report failures instead.
4. Open a PR; never push directly to the default branch.
5. Never add PATs or other secrets. Use the built-in `GITHUB_TOKEN`; keep
   `permissions: contents: read` and raise permissions per job only when required.
6. Pin any additional third-party action to a full commit SHA with a `# vX.Y.Z` comment.
