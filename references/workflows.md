# Workflows (recipes)

End-to-end recipes for common tasks. For per-command flags and JSON fields, read
the group file in this directory (listed in SKILL.md's command map). All flags
are from `gt` 0.1.0; `gt <cmd> --help` is authoritative.

## Issue lifecycle

```bash
gt issue list --assignee @me --state open
gt issue view 12 --comments
gt issue create --title "Crash on start" --body "Steps..." --label bug --milestone v1.0
gt issue edit 12 --add-label triage --add-assignee @me
gt issue comment 12 --body "Fixed in #14"
gt issue close 12 --comment "Done in #14"
```

## Pull requests: create → checks → review → merge

```bash
gt pr create --fill                       # push branch, open PR, title/body from commits
gt pr create --fill --yes                 # non-interactive: also push without asking
gt pr list --author @me --state all
gt pr checkout 13 --update                # local branch tracking the PR head
gt pr diff 13 | git apply                 # pipe clean diff (git apply works)
gt pr checks 13 --watch                   # waits; exit 8 while pending, 1 if a check failed
gt pr review --approve
gt pr review 13 --request-changes -b "Please hash the password"
gt pr merge 13 --squash --delete-branch
```

A branch that is not on the remote is pushed first, after a confirmation; in CI
pass `--yes` (or `--no-input` will fail rather than prompt). See
[pull-requests.md](pull-requests.md) for fork handling.

## Actions: dispatch, watch, logs, artifacts

```bash
gt workflow list
gt workflow run deploy.yml --ref main --field env=staging
gt run list --workflow CI --branch main --status failure
gt run watch 34 --timeout 10m && echo deployed
gt run view 34 --log
gt run logs 34 --job build --follow
gt run download 34 --name build --dir dist
gt secret set DEPLOY_TOKEN
```

`gt run watch` and `gt workflow run --follow` **fail when the run completes
without success**, so they chain well with `&&`. See [actions.md](actions.md).

## Releases

```bash
gt release create v1.0.0 dist/tool-linux dist/tool-darwin --notes-file CHANGELOG.md
gt release list
gt release upload v1.0.0 dist/tool-darwin --clobber
gt release download v1.0.0
```

`gt release create` never pushes a tag: push it first (`git push origin v1.0.0`)
or let Gitea create it with `--target`. See [releases.md](releases.md).

## Fork, clone and keep in sync

```bash
gt repo fork --remote          # inside a clone: add fork as origin, rename old origin to upstream
gt repo clone alice/project src --depth 1
gt repo sync                   # fast-forward the default branch from its remote
gt repo sync bob/project       # fork without a local remote: update it on the server
```

In a clone with both `upstream` and `origin`, `gt` targets `upstream` (like
`gh`). See [repos.md](repos.md).
