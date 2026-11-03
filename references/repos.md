# Repositories and branches (`gt repo`, `gt branch`, `gt browse`)

Repository management, server-side branches, and opening things in the browser.
`gt <cmd> --help` is authoritative. Global/repository-selection flags are in
SKILL.md.

## repo

```text
gt repo <command>
  archive  clone  create  delete  deploy-key  edit  fork  list  migrate
  rename  set-default  sync  transfer  unarchive  view
```

`repo list` aliases `repo ls`; `repo create` aliases `repo new`; `repo delete`
aliases `repo rm`.

**Repository argument** (where accepted): `NAME` (one of yours), `OWNER/NAME`,
`HOST/OWNER/NAME`, or a URL. Without one, the current clone's repository.

**JSON fields (repo objects):** `actions_enabled, archived, clone_url,
default_branch, description, empty, fork, forks_count, full_name, id, mirror,
mirror_interval, name, open_issues_count, open_pr_count, original_url, parent,
private, ssh_url, stars_count, template, topics, updated_at, url, website`.

### clone

`clone <repository> [<directory>] [-- <gitflags>...] [-d|--depth n]
[--find-instance] [--no-upstream] [-u|--upstream-remote-name NAME]` — clones
with `gt`'s credentials. A fork's parent is added as the `upstream` remote (or
the `--upstream-remote-name` one), and `gt` commands in the clone then target
the parent; a pull mirror's source is added as `source`. `--find-instance`
looks the repository up on every instance you have an account on when none is
selected, and asks which one to clone from if several have it (fails without
prompts).

```bash
gt repo clone alice/project src --depth 1
gt repo clone alice/project --find-instance
gt repo clone git@gitea.com:alice/project.git
gt repo clone alice/project -- --branch develop --recurse-submodules
```

### list / view

- `list [<owner>] [--starred] [--watched] [--type {fork|source|mirror}] [-L n]
  [--page n]`.
- `view [<repo>] [-b branch] [-w]` — description, state, counts, clone URLs,
  then the rendered README (`--branch` shows another branch's README).

### create

`create [<name>] [--org ORG] [--private|--public] [--auto-init]
[--readme TMPL] [--gitignores TMPL] [--license TMPL] [--default-branch NAME]
[-d DESC] [--source DIR] [--push] [--remote-name NAME] [--clone]
[--as-template] [--template OWNER/REPO] [--copy-avatar] [--copy-content]
[--copy-git-hooks] [--copy-labels] [--copy-topics] [--copy-webhooks]
[--label-template NAME] [--object-format {sha1|sha256}] [--team NAME]
[--trust-model {default|collaborator|committer|collaboratorcommitter}]`.

`--source`/`--push` publish a local repo; `--template` copies another and its
content/hooks/labels/topics by default; `--auto-init` adds a first commit from
the README/gitignore/license templates.

```bash
gt repo create project --private --auto-init --license MIT
gt repo create acme/service --template acme/service-template
gt repo create project --source . --push
```

### fork

`fork [<repo>] [--clone] [--org ORG] [--add-remote] [--remote-name NAME]
[--fork-name NAME] [-- <gitflags>]` — inside a clone, `--add-remote` adds the
fork as `origin` and renames the old `origin` to `upstream`. An existing fork
is reported instead of failing.

```bash
gt repo fork --add-remote
gt repo fork alice/project --org acme --fork-name project-fork
gt repo fork alice/project --clone -- --depth 1
```

### sync

`sync [<repo>] [-b|--branch branch]` — fast-forward a local branch (default:
the default branch) from its remote without switching. For a fork/mirror
without a local remote, updates it on the server; `--branch` is refused for a
mirror. JSON: `branch, commit, previous, remote`.

```bash
gt repo sync --branch release/1.0
gt repo sync bob/project              # update the fork on the server
```

### set-default

`set-default [<repo>] [-u|--unset] [-v|--view]` — store `gt.repository` in the
clone so commands target it instead of the git remotes. `--remote` saves the
repository of that Git remote.

```bash
gt repo set-default --remote upstream
gt repo set-default --view
gt repo set-default --unset
```

### edit / delete / rename / transfer / archive / unarchive / migrate / deploy-key

- `edit [<repo>]` — only the flags you pass change; on/off settings take
  `=false`. Flags include `-d/--description`, `--website`, `--private`
  (`=false` makes it public), `--visibility {public|private}`,
  `--default-branch`, `--enable-issues`, `--enable-pull-requests`,
  `--enable-wiki`, `--enable-projects`, `--actions`, `--as-template`,
  `--add-topic`/`--remove-topic`, `--default-merge-style
  {merge|rebase|rebase-merge|squash|fast-forward-only}`,
  `--allow-merge-commit`, `--allow-rebase`, `--allow-squash`,
  `--delete-branch-on-merge`, `--allow-manual-merge`,
  `--autodetect-manual-merge`.
- `delete [<repo>] [-y|--yes]`, `rename <new-name>`, `transfer <new-owner>
  [--team NAME]`, `archive`, `unarchive`.
- `migrate [<clone-url>] [--service {git|gitea|gitlab|gogs|github}]
  [--owner OWNER] [--mirror] [--mirror-interval 8h] [--issues] [--labels]
  [--milestones] [--pull-requests] [--releases] [--wiki] [--lfs]
  [--auth-token-file file] ...` — import or mirror a repository, and with
  `--service` copy issues, PRs, releases and more.
- `deploy-key {add|list|delete}` — repository-scoped SSH keys; `add ...
  --allow-write` lets the key push.

```bash
gt repo edit --default-merge-style squash --add-topic cli --allow-manual-merge
gt repo migrate https://github.com/acme/tool --service github --issues --pull-requests --auth-token-file token.txt
gt repo rename new-name
gt repo delete alice/old-project --yes
```

## branch

```text
gt branch <command>
  create  delete  list  protect  rename  unprotect  view
```

Server-side branches (use `git` for local branches). `branch list` aliases
`branch ls`; `branch delete` aliases `branch rm`. JSON: `can_merge, can_push,
message, name, protected, required_approvals, sha, updated_at`.

- `create <branch> [-s|--source branch|tag|commit]` — default branch unless
  `--source` names one.
- `list [-L n] [--page n]`, `view <branch> [-w]`, `rename <branch> <new-name>`,
  `delete <branch>... [-y]`.
- `protect <branch>... [--pattern glob] [--required-approvals n]
  [--allow-push user|ORG/TEAM] [--status-check name] [--dismiss-stale-approvals]
  [--block-on-rejected-reviews]` — by default nobody pushes directly and PRs
  need one approval. `--pattern` protects every matching branch; existing rules
  change only the named settings. An empty `--allow-push=` lets nobody push,
  and an empty `--status-check=` stops requiring status checks.
- `unprotect <branch>... [-y]`.

```bash
gt branch list
gt branch create release/1.0 --source main
gt branch protect main --required-approvals 2 --status-check ci/build
gt branch protect --pattern 'release/*' --allow-push acme/release
```

## browse

```text
gt browse [<number> | <path>[:<line>] | <commit-sha>] [flags]
```

Open the repository, or one of its issues, pull requests, files, directories or
commits, in the browser. A number opens that issue or PR, and a hex SHA of at
least 7 characters that commit; anything else is a path, opened on the default
branch unless `-b/--branch` or `-c/--commit` selects another. In a clone a path
is relative to the working directory, otherwise to the repository root. Append
`:LINE` or `:START-END` to a file path to select lines.

Flags: `-a/--actions`, `-p/--projects`, `-r/--releases`, `-s/--settings`,
`-w/--wiki`, `-b/--branch`, `-c/--commit [SHA]`, `-n/--no-browser` (print the
URL only).

```bash
gt browse
gt browse 217                         # the issue or PR
gt browse docs/README.md --branch develop
gt browse internal/app/app.go:42
gt browse --releases -R alice/project
gt browse --no-browser docs/README.md
```
