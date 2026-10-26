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
instance from the shell. Do not guess flags: each command's help lists its flags
and JSON fields, and the reference files mirror that help.

## Install `gt`

```bash
# Linux, macOS, FreeBSD
curl -fsSL https://gitea.com/gitea/cli/raw/branch/main/scripts/install.sh | sh

# Windows (PowerShell)
irm https://gitea.com/gitea/cli/raw/branch/main/scripts/install.ps1 | iex

# From source
go install gitea.dev/cli/cmd/gt@latest
```

Verify with `gt version`. Binaries are also at
[dl.gitea.com/gitea-cli](https://dl.gitea.com/gitea-cli/).

## Authenticate

`gt` stores accounts in `hosts.yml` and tokens in the system keyring (or a
private file), under `~/.config/gitea-cli/`. It never reads `tea`'s config.

```bash
gt auth login                                       # interactive: OAuth or token
gt auth login --host gitea.example.com --with-token < token.txt   # CI
echo "$TOKEN" | gt auth login --host gitea.example.com --with-token
gt auth status --verify                             # check the active credential
gt doctor                                           # diagnose the setup here
```

In CI, `GITEA_TOKEN` + `GITEA_INSTANCE_URL` (or `GITEA_CLI_HOST`) replace stored
credentials; other `GITEA_CLI_*` variables mirror flags and settings. Full flags,
`setup-git`, `token`, and the environment list are in
[references/auth.md](references/auth.md).

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

Repository-scoped groups also inherit `--remote`, `--cwd`; list/view commands
add `--json`/`--jq`. See `gt help environment`.

### JSON output

Text output is for people and may change; **use `--json` in scripts**. `--jq`
runs inside `gt`, so jq need not be installed. An unknown field reports the
available ones, and each command's help lists its JSON fields.

```bash
gt issue list --json number,title --jq '.[] | "\(.number)\t\(.title)"'
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

Read the reference file for a group to get its per-command flags, arguments,
`--json` fields and examples. `gt <group> <cmd> --help` is always authoritative.

| Group | Commands | Reference |
| --- | --- | --- |
| Auth | `auth`, `doctor` | [auth.md](references/auth.md) |
| Repositories | `repo`, `branch` | [repos.md](references/repos.md) |
| Issues | `issue`, `label`, `milestone` | [issues.md](references/issues.md) |
| Pull requests | `pr` | [pull-requests.md](references/pull-requests.md) |
| Releases | `release` | [releases.md](references/releases.md) |
| Organizations | `org`, `team` | [organizations.md](references/organizations.md) |
| Actions | `run`, `workflow` | [actions.md](references/actions.md) |
| CI config | `secret`, `variable`, `package` | [ci-config.md](references/ci-config.md) |
| Discovery | `search`, `status`, `notification`, `time` | [discovery.md](references/discovery.md) |
| Instance | `config`, `api`, `alias`, `completion`, `webhook`, `wiki`, `ssh-key`, `admin` | [instance.md](references/instance.md) |
| Recipes | end-to-end workflows | [workflows.md](references/workflows.md) |

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
