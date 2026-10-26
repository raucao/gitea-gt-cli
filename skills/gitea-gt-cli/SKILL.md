---
name: gitea-gt-cli
description: "Gitea CLI (gt) reference for working with Gitea from the terminal: repositories, issues, pull requests, reviews, releases, Actions runs/workflows, secrets, variables, packages, organizations, and auth. Use when managing Gitea with the `gt` command (the official rewrite of `tea`), or when a task mentions Gitea repos, issues, PRs, Actions, or authentication."
license: Unlicense
metadata:
  version: "0.1.0"
  upstream: https://gitea.com/gitea/cli
  gt-version: ">=0.1.0"
  gitea-version: ">=28"
---

# Gitea CLI (`gt`)

`gt` is Gitea's official command line tool, a rewrite of `tea` with a new name,
repository ([gitea.com/gitea/cli](https://gitea.com/gitea/cli)) and a consistent
command surface. It manages repositories, issues, pull requests, releases,
Actions, packages and the rest of Gitea from the terminal, and behaves the same
at your desk and in CI.

- **Version covered:** `gt` 0.1.0 (2026-10-23).
- **Requires:** Gitea **28 or newer**; older versions are not supported.
- **0.x:** flags and text output may still change. **Scripts must use `--json`**,
  which is stable; text output is for people.

Use this skill whenever a task involves running `gt` or managing a Gitea
instance from the shell. Do not guess flags: every command's help lists its
flags and JSON fields, and the reference files below mirror that help.

## Install `gt`

```bash
# Linux, macOS, FreeBSD
curl -fsSL https://gitea.com/gitea/cli/raw/branch/main/scripts/install.sh | sh

# Windows (PowerShell)
irm https://gitea.com/gitea/cli/raw/branch/main/scripts/install.ps1 | iex

# Docker
docker run --rm gitea/gitea-cli --help

# From source
go install gitea.dev/cli/cmd/gt@latest
```

Verify with `gt version`. Binaries are also at
[dl.gitea.com/gitea-cli](https://dl.gitea.com/gitea-cli/).

## Authenticate

`gt` keeps accounts in `hosts.yml` and tokens in the system keyring (or a
private file), under `~/.config/gitea-cli/` (`GITEA_CLI_CONFIG_DIR` overrides).
It never reads `tea`'s config.

```bash
# Interactive: asks for instance, OAuth or token, storage, and git setup
gt auth login

# CI / non-interactive: token on stdin
gt auth login --host gitea.example.com --with-token < token.txt
echo "$TOKEN" | gt auth login --host gitea.example.com --with-token

# OAuth in the browser with explicit scopes
gt auth login --host gitea.example.com --oauth --scopes read:repository,read:user

# Inspect and use accounts
gt auth status            # which accounts, active one, method (exit 4 if none/invalid)
gt auth status --verify   # check credentials against the instance
gt auth token             # print the active token (for curl/scripts)
gt auth switch            # change the active account of an instance
gt auth setup-git         # configure git to use gt for https push/fetch
gt doctor                 # diagnose settings, instance, account, repo and git
```

### Environment variables (CI)

- `GITEA_TOKEN` + `GITEA_INSTANCE_URL` (or `GITEA_CLI_HOST`): use a token
  instead of stored credentials.
- `GITEA_CLI_*` prefixed variables mirror global flags and settings; see
  `gt help environment` for the full list (config dir, user, repo, editor,
  pager, color, http timeout, ...).
- `GITEA_INSTANCE_INSECURE`, `GITEA_INSTANCE_SSH_HOST`: pair with `GITEA_TOKEN`.

## Core concepts

### Instance and repository selection

`gt` picks the **instance** from: repository URL, `--host`, `GITEA_CLI_HOST` /
`GITEA_INSTANCE_URL`, `default_host`, the only instance with an account, or the
instance of the clone's repository.

It picks the **repository** from: `--repo`/`-R OWNER/REPO`, `GITEA_CLI_REPO`,
`git config gt.repository` (set by `gt repo set-default`), or a git remote
(`--remote`, else the base remote `pr checkout` recorded, the current branch's
remote, upstream, or origin). Like `gh`, a clone with `upstream` and `origin`
targets `upstream`.

```bash
gt -R alice/project issue list
gt issue list --repo alice/project      # same, flag after the command
gt repo set-default upstream/name       # make this clone target the upstream
```

### Global flags

Work with every command, before or after the command name:

| Flag | Meaning |
| --- | --- |
| `-R, --repo OWNER/REPO` | Select a repository (accepts a URL) |
| `--host URL` | Gitea instance URL, including any base path |
| `--user name` / `--sudo user` | Act as another account / as another user (admin) |
| `--json fields` | Output JSON: comma-separated fields, or all |
| `-q, --jq expr` | Filter JSON with a jq expression; implies `--json` |
| `-y, --yes` | Accept confirmations without prompting |
| `--no-input` | Disable prompts and implicit editor/browser |
| `--interactive` | Allow prompts with machine output |
| `--debug` | Log HTTP method, URL, status, duration to stderr |
| `--cwd dir` / `--remote remote` | Repository discovery |

### JSON output

Text output is for people and may change; **use `--json` in scripts**. `--jq`
runs inside `gt`, so jq need not be installed. An unknown field reports the
available ones, and each command's help lists its JSON fields.

```bash
gt issue list --json number,title --jq '.[] | "\(.number)\t\(.title)"'
gt pr list --json number,title,head --jq '.[] | select(.head == "feature/login")'
gt run list --since 24h --json id,status,conclusion,workflow
```

Some fields cost extra requests (e.g. `pr list` reviewers, `issue view`
blocked_by/blocks) and are fetched **only when the output names them**; name
them in `--json` rather than relying on `--jq`. See `gt help formatting`.

### Exit codes

| Code | Meaning |
| --- | --- |
| 0 | success (including an empty list) |
| 1 | command failed |
| 2 | usage error (unknown command/flag, bad argument) |
| 3 | canceled (declined confirmation, Esc, timeout) |
| 4 | authentication missing or rejected |
| 8 | `gt pr checks`: some checks still pending |
| 130 | interrupted with Ctrl-C |

Commands may document further codes. See `gt help exit-codes`.

## Command map

| Group | Commands | Reference |
| --- | --- | --- |
| Auth | `auth` (login, logout, refresh, setup-git, status, switch, token) | `references/commands-core.md` |
| Repositories | `repo` (create, clone, fork, sync, view, list, edit, migrate, deploy-key, ...) | `references/commands-core.md` |
| Issues | `issue` (create, list, view, edit, close, reopen, comment, timeline, attachment) | `references/commands-core.md` |
| Pull requests | `pr` (create, list, view, checkout, diff, checks, review, merge, status, clean) | `references/commands-core.md` |
| Releases | `release` (create, list, view, edit, upload, download, asset, publish) | `references/commands-core.md` |
| Orgs & labels | `org`, `label`, `milestone`, `branch` | `references/commands-core.md` |
| Actions | `run`, `workflow`, `secret`, `variable`, `package`, `admin` | `references/commands-actions.md` |
| Everything else | `team`, `notification`, `search`, `status`, `time`, `webhook`, `wiki`, `ssh-key`, `api`, `config`, `alias`, `completion` | `references/commands-misc.md` |

**Read the reference file** when you need exact flags, arguments, accepted
`--json` fields, or examples for a command. `gt <group> <cmd> --help` is always
authoritative for the installed version.

## Common workflows

### Issue lifecycle

```bash
gt issue list --assignee @me --state open
gt issue view 12 --comments
gt issue create --title "Crash on start" --body "Steps..." --label bug --milestone v1.0
gt issue edit 12 --add-label triage --add-assignee @me
gt issue comment 12 --body "Fixed in #14"
gt issue close 12 --comment "Done in #14"
```

### Pull requests (create → checks → review → merge)

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

### Actions: run, watch, logs, artifacts

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

### Releases

```bash
gt release create v1.0.0 dist/tool-linux dist/tool-darwin --notes-file CHANGELOG.md
gt release list
gt release upload v1.0.0 dist/tool-darwin --clobber
gt release download v1.0.0
```

### Fork / clone / keep in sync

```bash
gt repo fork --remote          # inside a clone: add fork as origin, rename old origin to upstream
gt repo clone alice/project src --depth 1
gt repo sync                   # fast-forward the default branch from its remote
gt repo sync bob/project       # fork without a local remote: update it on the server
```

## Coming from `tea`

`gt` is a new tool, not an upgrade of `tea`; nothing is migrated and both can be
installed side by side.

| `tea` | `gt` |
| --- | --- |
| `tea login add`, `tea logins`, `tea whoami` | `gt auth login`, `gt auth status` |
| `tea open` | `--web` on the relevant command |
| `tea clone` | `gt repo clone` |
| `tea comment` | `gt issue comment`, `gt pr comment` |
| `tea actions ...` | `gt run`, `gt workflow`, `gt secret`, `gt variable` |

## Gotchas

- **Scripts:** always parse `--json`, never text output. Fields are stable;
  columns and wording are not, in 0.x.
- **`gt pr checks`** exits **8** while checks are pending and **1** when one
  failed — chain `gt pr checks 13 --watch && gt pr merge 13`.
- **First-run auth in CI** fails with exit **4**; set `GITEA_TOKEN` and
  `GITEA_INSTANCE_URL`, or `gt auth login --with-token`.
- **Fork vs upstream:** in a clone with `upstream` and `origin`, commands target
  `upstream`. Use `gt repo set-default --view`/`--unset`, or `-R`, to change.
- **Non-interactive prompts:** pass `--no-input` or `--yes`; with `TERM=dumb`
  every prompt is asked as a plain line, which works with screen readers.
- **`--json` extra-request fields** are empty unless named in `--json`. Name
  them (e.g. `--json number,title,reviewers`), don't rely on `--jq` traversal.
- **`release create` never pushes:** push the tag first, or pass `--target`.
- **Labels/milestones** must already exist in the repository; create them with
  `gt label create` / `gt milestone create` (or `gt label import`).
- **Help topics** for the details above: `gt help environment`,
  `gt help formatting`, `gt help exit-codes`.
