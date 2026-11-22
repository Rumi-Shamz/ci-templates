# ci-templates

Reusable GitHub Actions workflows, one per project type, plus ready-to-copy
caller files ("starters"). Public on purpose: public reusable workflows can be
called from **private repos in any org or account**.

Every workflow: `permissions: contents: read` at the top (raised per job only
where required), third-party actions pinned to full commit SHAs, a
`timeout-minutes` on every job, `ubuntu-latest`, and **no secrets required**
(only the built-in `GITHUB_TOKEN`).

## Project type → starter → what it checks

| Project type | Starter | What it checks |
| --- | --- | --- |
| Node / TypeScript (`package.json`, `tsconfig.json`) | [`starters/node-ts/ci.yml`](starters/node-ts/ci.yml) | `npm ci`, then `lint`, `typecheck`, `test`, `build` — each only if that npm script exists. Node version from input, else `.nvmrc`, else 22. |
| Python (`requirements*.txt` / `pyproject.toml`) | [`starters/python/ci.yml`](starters/python/ci.yml) | Installs dependencies, `ruff check .` (ruff installed in the job if absent), `pytest` if a `tests/` (or `test/`) dir exists. |
| Go (`go.mod`) | [`starters/go/ci.yml`](starters/go/ci.yml) | Go version from `go.mod`; `go vet ./...`, `go test ./...`, `go build ./...`. |
| Anything with a `Dockerfile` | [`starters/docker-build/ci.yml`](starters/docker-build/ci.yml) | Builds the image (no push, no registry login) to prove it builds. Inputs `context`, `file`. |
| Static HTML site deployed to GitHub Pages | [`starters/static-pages/ci.yml`](starters/static-pages/ci.yml) | Optional build command, upload Pages artifact, deploy on push (build-only on PRs). Caller job needs `pages: write` + `id-token: write`; repo Pages source must be "GitHub Actions". |

The reusable workflows live in [`.github/workflows/`](.github/workflows/)
(`node-ts.yml`, `python.yml`, `go.yml`, `docker-build.yml`, `static-pages.yml`)
and take `working-directory` / version / timeout inputs — see the `inputs:`
block at the top of each file.

## Adopting a starter

1. Copy the matching `starters/<type>/ci.yml` to `.github/workflows/ci.yml` in your repo.
2. Set `push.branches` to the repo's default branch (the starters say `main`).
3. For several types (e.g. Python + Docker) put each starter's job under one
   `jobs:` block in the same `ci.yml` (job names are already distinct).
4. Open a PR; the workflow runs on `pull_request` and on pushes to the default branch.
   Doc-only changes (`docs/**`, `**.md`) are ignored. In-progress runs are
   cancelled on pull requests only.

## Bumping the pinned SHA

Starters call the reusable workflow as
`Rumi-Shamz/ci-templates/.github/workflows/<type>.yml@<full-sha> # v1`.
To move a repo to newer templates:

```bash
gh api repos/Rumi-Shamz/ci-templates/commits/v1 --jq .sha   # SHA the moving v1 tag points at
```

Replace the SHA in the repo's `ci.yml` (keep the `# v1` comment). Dependabot's
`github-actions` ecosystem can also bump these pins. Releases: `v1.0.0` is
immutable; `v1` moves forward with compatible changes; breaking changes get `v2`.

## Notes

- **Public reusable workflows are callable from private repos in any org.**
- **Minutes are billed to the calling repo's owner**, not to this repo. If the
  caller's account has no Actions minutes left, the run will not start.
- No PATs or secrets are needed or supported; the workflows only use `GITHUB_TOKEN`.
- Self-test: [`lint-workflows.yml`](.github/workflows/lint-workflows.yml) runs
  `actionlint` on every workflow and starter in this repo.
