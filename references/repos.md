# Repositories (`gt repo`, `gt branch`)

Repository management plus server-side branches. `gt <cmd> --help` is
authoritative. Global/repository-selection flags are in SKILL.md.

## repo

```text
gt repo <command>
  archive  clone  create  delete  deploy-key  edit  fork  list  migrate
  rename  set-default  sync  transfer  unarchive  view
```

`repo list` aliases `repo ls`; `repo create` aliases `repo new`.

**Repository argument** (where accepted): `NAME` (one of yours), `OWNER/NAME`,
`HOST/OWNER/NAME`, or a URL. Without one, the current clone's repository.

**JSON fields (repo objects):** `actions_enabled, archived, clone_url,
default_branch, description, empty, fork, forks_count, full_name, id, mirror,
mirror_interval, name, open_issues_count, open_pr_count, original_url, parent,
private, ssh_url, stars_count, template, topics, updated_at, url, website`.

### Key subcommands

- `list [<owner>] [--starred] [--watched] [--type {fork|source|mirror}] [-L n]
  [--page n]`
- `view [<repo>] [-b branch] [-w]` — description, state, counts, clone URLs,
  then the rendered README.
- `create [<name>] [--org ORG] [--private|--public] [--auto-init]
  [--readme TMPL] [--gitignores TMPL] [--license TMPL] [--default-branch NAME]
  [-d DESC] [--template OWNER/REPO] [--source DIR] [--push] [--clone]
  [--as-template]` — `--source`/`--push` publish a local repo;
  `--template` copies another; `--auto-init` adds a first commit.
- `clone <repo> [<dir>] [-- <gitflags>] [-d depth] [--no-upstream]
  [-u upstream-name]` — a fork's parent is added as `upstream`; a mirror's
  source as `source`. `gt` commands in the clone then target the parent.
- `fork [<repo>] [--clone] [--org ORG] [--remote] [--remote-name NAME]
  [--fork-name NAME] [-- <gitflags>]` — inside a clone, `--remote` adds the fork
  as `origin` and renames the old `origin` to `upstream`.
- `sync [<repo>] [-b branch]` — fast-forward a local branch (default: default
  branch) from its remote without switching; for a fork/mirror without a local
  remote, updates it on the server. JSON: `branch, commit, previous, remote`.
- `set-default [<repo>] [-u|--unset] [-v|--view]` — store `gt.repository` in the
  clone so commands target it instead of the git remotes.
- `edit`, `delete`, `rename`, `transfer`, `archive`, `unarchive`, `migrate`,
  `deploy-key`.

```bash
gt repo list --starred --json full_name,url
gt repo create project --private --auto-init --license MIT
gt repo create acme/service --template acme/service-template
gt repo create project --source . --push
gt repo fork alice/project --clone -- --depth 1
gt repo sync --branch release/1.0
gt repo sync bob/project              # update the fork on the server
```

## branch

```text
gt branch <command>
  create  delete  list  protect  rename  unprotect  view
```

Server-side branches (use `git` for local branches). JSON: `can_merge,
can_push, message, name, protected, required_approvals, sha, updated_at`.

- `create <branch> [-s|--source branch|tag|commit]` — default branch unless
  `--source` names one.
- `list`, `view <branch> [-w]`, `rename`, `delete`
- `protect <branch>... [--pattern glob] [--required-approvals n]
  [--allow-push user|ORG/TEAM] [--status-check name] [--dismiss-stale-approvals]
  [--block-on-rejected-reviews]` — by default nobody pushes directly and PRs
  need one approval. `--pattern` protects every matching branch; existing rules
  change only the named settings.
- `unprotect <branch>...`

```bash
gt branch list
gt branch create release/1.0 --source main
gt branch protect main --required-approvals 2 --status-check ci/build
gt branch protect --pattern 'release/*' --allow-push acme/release
```
