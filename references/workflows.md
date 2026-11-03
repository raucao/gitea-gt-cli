# Workflows (recipes)

End-to-end recipes for common tasks. For per-command flags and JSON fields, read
the group file in this directory (listed in SKILL.md's command map). All flags
are from `gt` 1.0.0; `gt <cmd> --help` is authoritative.

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
gt pr list --local                        # which local branches belong to a merged/closed PR
gt pr checkout 13 --update                # local branch tracking the PR head
gt pr diff 13 | git apply                 # pipe clean diff (git apply works)
gt pr checks 13 --watch                   # waits; exit 8 while pending, 1 if a check failed
gt pr review --approve
gt pr review 13 --request-changes -b "Please hash the password"
gt pr update-branch 13 --rebase           # bring the base branch in
gt pr merge 13 --squash --delete-branch
gt pr clean 13                            # delete the branch of a merged/closed PR
```

A branch that is not on the remote is pushed first, after a confirmation; in CI
pass `--yes` (or `--no-input` will fail rather than prompt). See
[pull-requests.md](pull-requests.md) for fork handling.

## Backport a merged PR

```bash
gt pr backport 42 --to release/v1.2 --to release/v1.1 --yes
# On conflict, resolve yourself:
gt pr backport 42 --to release/v1.1 --keep
git add . && git cherry-pick --continue
gt pr backport --continue
```

## Mark a PR merged outside Gitea

The repository must allow it (`gt repo edit --allow-manual-merge`).

```bash
git push origin main                      # the commit that merged the PR
gt pr merge 13 --method manually-merged --commit 4b825dc --delete-branch
```

## Actions: dispatch, watch, logs, artifacts

```bash
gt workflow list
gt workflow run deploy.yml --ref main --field env=staging
gt run list --workflow CI --branch main --status failure
gt run watch 34 --timeout 10m && echo deployed
gt run watch 34 --expect-head HEAD        # exit 6 if the run is for another commit
gt run view 34 --log
gt run logs 34 --job build --follow
gt run logs 34 --failed --tail 50
gt run download 34 --name build --dir dist
gt secret set DEPLOY_TOKEN
```

`gt run watch` and `gt workflow run --follow` **exit 5 when the run completes
without success**, so they chain well with `&&`; exit 1 means `gt` itself
failed. See [actions.md](actions.md). Runners are in [runners.md](runners.md).

## Projects

```bash
gt project create Roadmap --template basic_kanban
gt project list --state open
gt issue move 47 to "In progress"
```

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
gt repo fork --add-remote     # inside a clone: add fork as origin, rename old origin to upstream
gt repo clone alice/project src --depth 1
gt repo clone alice/project --find-instance   # pick the instance that has it
gt repo sync                  # fast-forward the default branch from its remote
gt repo sync bob/project      # fork without a local remote: update it on the server
```

In a clone with both `upstream` and `origin`, `gt` targets `upstream` (like
`gh`). See [repos.md](repos.md).

## Extensions and keeping `gt` current

```bash
gt extension install alice/gt-hello
gt extension list
gt upgrade                    # self-upgrade (verified checksum)
```

See [extensions.md](extensions.md).

## Machine-readable output: JSON and CSV

Text output is for people and may change; scripts should use `--json`/`--jq` or
`--csv`.

```bash
gt issue list --state all --json number,title,state
gt issue list --state all --csv number,title,state > issues.csv
gt issue list --json number,title --jq '.[] | "\(.number)\t\(.title)"'
```
