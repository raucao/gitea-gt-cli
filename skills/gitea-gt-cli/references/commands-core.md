# Command reference: core

Per-command flags, arguments, JSON fields and examples for the core `gt` groups:
`auth`, `repo`, `issue`, `pr`, `release`, `org`, `label`, `milestone`, `branch`.

`gt <group> <cmd> --help` is authoritative for the installed version.

## Shared flags

- **Repository-scoped groups** (`repo`, `issue`, `pr`, `release`, `branch`,
  `label`, `milestone`) inherit:
  `-R, --repo OWNER/REPO`, `--remote remote`, `--cwd directory`.
- **List/view commands** additionally inherit `--json fields` and
  `-q, --jq expression` (see `gt help formatting`).
- **Mutating commands** that confirm inherit `-y, --yes`.
- Global flags (`--host`, `--user`, `--sudo`, `--no-input`, `--interactive`,
  `--debug`) work everywhere; see `gt help environment`.

---

## auth

```text
gt auth <command>
  login  logout  refresh  setup-git  status  switch  token
```

### gt auth login

Log in to a Gitea instance. Without a method flag on a terminal it asks for the
instance, login method, token storage and whether git should use `gt`.

| Flag | Meaning |
| --- | --- |
| `--with-token` | Read an access token from stdin (or prompt) |
| `--oauth` | Authorize in a browser; `--client-id ID` selects the OAuth app |
| `--scopes scopes` | OAuth scopes, e.g. `read:repository,read:user` (implies `--oauth`) |
| `--credential-store {keyring\|file}` | Where to keep the token (default `keyring`) |
| `--insecure` | Skip TLS verification for this instance |
| `--header "Name: value"` | Send a header with every request (repeatable; `$VAR` expansion) |
| `--ssh-key file`, `--ssh-agent-key fp`, `--ssh-agent-principal p` | Sign requests with SSH |
| `--client-cert file`, `--client-key file` | TLS client certificate |
| `-y, --yes` | Replace an existing account without confirming |

```bash
gt auth login
gt auth login --host gitea.com --with-token < token.txt
gt auth login --host gitea.com --oauth --scopes read:repository,read:user
gt auth login --host g.example.com --with-token \
  --header 'CF-Access-Client-Id: $CF_ID' --header 'CF-Access-Client-Secret: $CF_SECRET'
```

### gt auth status / token / switch / setup-git / logout / refresh

- `status [--verify] [--json ...]` — accounts per instance, active one, method.
  Fails exit 4 when there is no account or the credential is missing/rejected;
  with `--json` it exits 0 and you read `credential_available`, `verified`,
  `error`. JSON fields: `active, credential_available, credential_store, error,
  expires_at, git_credential_helper, host, method, scopes, user, verified`.
- `token` — print the active token (refreshing an expiring OAuth token). SSH-key
  accounts have no token.
- `switch` — change the active account of an instance.
- `setup-git [--keep-helpers] [--local]` — configure git to ask `gt` for
  credentials.
- `logout`, `refresh` (refresh an OAuth token).

```bash
gt auth status --verify
curl -H "Authorization: token $(gt auth token)" https://gitea.com/api/v1/user
```

---

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
- `view [<repo>] [-b branch] [-w]`
- `create [<name>] [--org ORG] [--private|--public] [--auto-init]
  [--readme TMPL] [--gitignores TMPL] [--license TMPL] [--default-branch NAME]
  [-d DESC] [--template OWNER/REPO] [--source DIR] [--push] [--clone]
  [--as-template]`
- `clone <repo> [<dir>] [-- <gitflags>] [-d depth] [--no-upstream]
  [-u upstream-name]` — a fork's parent is added as `upstream`; a mirror's
  source as `source`.
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

---

## issue

```text
gt issue <command>
  attachment  close  comment  create  edit  list  reopen  timeline  view
```

`issue list` aliases `issue ls`; `issue create` aliases `issue new`.

**Issue argument:** `123`, `#123`, a URL, or `OWNER/REPO#123`.

**JSON fields (issue):** `assignees, author, body, comments, created_at,
due_date, id, kind, labels, milestone, number, ref, state, title, updated_at,
url`. `view`/`edit` add `blocked_by, blocks` (extra requests, only when named).

### list

| Flag | Meaning |
| --- | --- |
| `-s, --state {open\|closed\|all}` | default `open` |
| `-a, --assignee`, `-A, --author`, `--mentions` | `@me` for yourself |
| `--no-assignee` | only issues without an assignee |
| `-l, --label names`, `-m, --milestone titles` | filters |
| `--kind {issues\|pulls\|all}` | include PRs (Gitea lists PRs among issues) |
| `-S, --search text` | search titles and bodies |
| `--since`, `--until` | updated window (date, timestamp, or duration ago) |
| `-L, --limit n` (1..1000, default 30), `--page n` | pagination |
| `-w, --web` | open in browser |

```bash
gt issue list --label bug --state all
gt issue list --assignee @me --json number,title
gt issue list --kind all --milestone v1.0 | cut -f1,2
```

### create

`-t/--title`, `-b/--body`, `-F/--body-file file|-`, `-e/--editor`,
`-a/--assignee login`, `-l/--label name` (prefix `repo:`/`org:` to disambiguate),
`-m/--milestone`, `-p/--project`, `--ref`, `--deadline DATE`,
`-T/--template name`, `--attachment File`, `-w/--web`.

Gitea applies labels/milestone/assignees/deadline only to users with issue write
access; otherwise `gt` reports the left-out fields and fails.

```bash
gt issue create --title "Crash on start" --body "Steps..." \
  --label bug --milestone v1.0
gt issue create --template "Bug report"
```

### edit

Adds/removes relative to current values, or replaces with `--set-*`:
`--add-label`, `--remove-label`, `--set-labels`, `--add-assignee`,
`--remove-assignee`, `--set-assignees`, `-m/--milestone`,
`--remove-milestone`, `--add-blocked-by`, `--remove-blocked-by`,
`--add-project`, `--remove-project`, `-t/--title`, `-b/--body`,
`-F/--body-file`, `-e/--editor`, `--ref`, `--deadline`. Accepts multiple issues.

```bash
gt issue edit 12 --title "Crash on start" --add-label bug
gt issue edit 12 13 --add-assignee @me --milestone v1.0
gt issue edit 12 --add-blocked-by 9
```

### close / reopen / comment / timeline / view / attachment

- `close {N}... [-c comment]`, `reopen {N}...`
- `comment {N} [-b text] [-F file] [-e] [--edit-last] [--delete-last]
  [--attachment File]`; subcommands `list|view|edit|delete`. JSON: `author,
  body, created_at, id, issue_number, updated_at, url`.
- `timeline {N} [-L n] [--page n]` — comments and changes, oldest first.
- `view {N} [-c|--comments] [-w]`
- `attachment {list|view|upload|download|edit|delete}`

```bash
gt issue view 12 --comments
gt issue comment 12 --body "Fixed in #14"
gt issue close 12 13 --comment "Done in #14"
```

---

## pr

```text
gt pr <command>
  attachment  checkout  checks  clean  close  comment  create  diff
  edit  list  merge  ready  reopen  review  status  timeline  view
```

`pr list` aliases `pr ls`; `pr create` aliases `pr new`; `pr checkout` aliases
`pr co`.

**PR argument:** `13`, `#13`, a URL, or a head branch name (e.g. `feature/login`
or `bob:feature/login`). Without one, the PR of the current branch.

**JSON fields (PR):** `assignees, author, base, body, comments, created_at,
draft, due_date, head, head_remote_deleted, head_repo, head_sha, id, labels,
merge_commit_sha, mergeable, merged, merged_at, merged_by, milestone, number,
review_decision, reviewer_teams, reviewers, reviews, state, title, updated_at,
url`. `reviewers`, `reviewer_teams`, `reviews`, `review_decision` and
`merged_by` on `list` cost extra requests and are fetched only when named.

### list

`-s, --state {open|closed|merged|all}` (default `open`), `-a/--assignee`,
`-A/--author`, `--no-assignee`, `-l/--label`, `-m/--milestone`,
`-B/--base branch`, `-H/--head branch|OWNER:BRANCH`, `-d/--draft`,
`-S/--search`, `--sort`, `-L/--limit`, `--page`, `-w/--web`.

### create

`--fill` (title/body from commits), `-t/--title`, `-b/--body`,
`-F/--body-file`, `-e/--editor`, `-B/--base`, `-H/--head`, `-d/--draft`,
`-a/--assignee`, `-r/--reviewer`, `--reviewer-team`, `-l/--label`,
`-m/--milestone`, `-p/--project`, `--deadline`, `--attachment`,
`--allow-maintainer-edits` (default true), `--agit` (+ `--topic`), `-w/--web`,
`-y/--yes`.

Pushes a not-yet-remote branch after confirmation (`--yes` in CI). A branch
without a same-named upstream goes to the fork when origin is a fork, or to the
only remote of your own fork; otherwise to the repository's remote.

```bash
gt pr create --fill --yes
gt pr create -t "Add login" -b "Closes #1" -B develop -r carol -l enhancement
```

### checkout / diff / checks

- `checkout {N|url|branch} [-b branch] [--detach] [--update]` — local branch
  tracking the head. JSON: `branch, commit, detached, remote, tracking`.
- `diff [{N|url|branch}] [--name-only] [--color auto|always|never] [-w]` — clean
  diff when piped (`gt pr diff 13 | git apply`).
- `checks [{N|url|branch}] [--watch] [--interval d] [--timeout d] [-w]` — exit
  **8** while pending, **1** on failure. JSON: `created_at, description, id,
  name, sha, state, updated_at, url`.

```bash
gt pr checkout 13 --update
gt pr diff 13 | git apply
gt pr checks 13 --watch && gt pr merge 13
```

### merge

`-s/--method {merge|rebase|rebase-merge|squash|fast-forward-only}` (default:
repo's style), shortcuts `--merge`, `--rebase`, `--squash`, `--auto`
(after checks), `-d/--delete-branch` (also deletes the local head branch when
checked out), `-t/--title`, `-b/--message`, `-F/--message-file`, `-e/--editor`.

```bash
gt pr merge 13 --squash --delete-branch
gt pr merge 13 --auto --yes
```

### review

Choose the type with `-a/--approve`, `-r/--request-changes` (needs a body), or
`-c/--comment`. Flags: `-b/--body`, `-F/--body-file`, `-e/--editor`,
`--pending` (save for later), `--commit sha`,
`--comments-file file` (JSON array of `path`, `body`, `new_position` or
`old_position`), `-y/--yes`. Subcommands: `create, list, view, delete, submit,
request, cancel-request, comments, reply, resolve, unresolve, dismiss,
undismiss`. JSON: `author, body, comments_count, commit, dismissed, id,
official, stale, state, submitted_at, team, url`.

On a terminal, no type opens an interactive review that can annotate the diff
in your editor.

```bash
gt pr review --approve
gt pr review 13 -r -b "Please hash the password"
gt pr review 13 --pending --comments-file inline.json
gt pr review submit 13 --approve
```

### status / clean / ready / comment

- `status [--limit n]` — current branch's PR with checks/review, then your
  authored/assigned/review-requested PRs. JSON: `assigned, authored,
  current_branch, review_requested`.
- `clean [{...}] [--ignore-sha]` — delete the branch of a merged/closed PR
  locally and on Gitea. JSON: `default_branch, local_branch, remote,
  remote_branch, remote_deleted, remote_skip_reason, remote_skipped`.
- `ready [{...}] [--undo]` — mark ready/back to draft (toggles `WIP:`/`[WIP]`).
- `comment` — like `issue comment` (JSON: `author, body, created_at, id,
  issue_number, updated_at, url`).
- `view`, `edit`, `close`, `reopen`, `timeline`, `attachment`.

---

## release

```text
gt release <command>
  asset  create  delete  download  edit  list  publish  upload  view
```

`release list` aliases `release ls`; `release create` aliases `release new`.

**JSON fields:** `assets, author, body, created_at, draft, id, prerelease,
published_at, tag, target, title, url`.

### create

`gt release create [<tag>] [<file>...]` with `-t/--title`, `-n/--notes`,
`-F/--notes-file`, `-e/--editor`, `-d/--draft`, `-p/--prerelease`,
`--target branch|commit` (for a new tag), `--asset File`, `--verify-tag`.

`gt` **never pushes**: push the tag first, or let Gitea create it with
`--target`. A tag that exists only in your clone is refused without `--target`.

```bash
gt release create v1.0.0 dist/tool-linux dist/tool-darwin --notes-file CHANGELOG.md
gt release create v1.1.0-rc1 --target release/1.1 --draft --prerelease --title "1.1 RC 1"
```

### upload / download / asset / publish / edit / delete / list / view

- `upload <tag> <file>... [--name NAME] [--clobber]` — JSON per asset:
  `created_at, download_count, id, name, size, url`.
- `download <tag> [...]`, `asset <list|...>`, `publish <tag>...`,
  `edit <tag>...`, `delete <tag>...`, `list [-L n] [--page n]`,
  `view <tag>`.

```bash
gt release upload v1.0.0 dist/tool-linux --clobber
```

---

## org

```text
gt org <command>
  create  delete  edit  list  member  view
```

Organizations you belong to. Teams are in `team` (see
`commands-misc.md`).

```bash
gt org list
gt org view acme
gt org member list acme
gt org create acme --description "Acme org"
```

## label

```text
gt label <command>
  create  delete  edit  export  import  list
```

Scope with `--org organization` for organization labels; otherwise the
repository's. Addressed by name or ID. JSON: `color, description, exclusive,
id, is_archived, name`.

`create <name> [-c color] [-d description] [--exclusive] [--archived]
[-f/--force]` — random color without `-c`; `--force` updates on name clash.

```bash
gt label create bug --color d73a4a --description "Something is broken"
gt label create "priority/high" --exclusive
gt label export labels.yml
gt label import labels.yml
```

## milestone

```text
gt milestone <command>
  close  create  delete  edit  issue  list  reopen  view
```

Addressed by title or ID. `milestone issue` manages the issues/PRs in it.

```bash
gt milestone create v1.0 --deadline 2026-12-31 --description "First release"
gt milestone list --state open
gt milestone issue list v1.0
```

## branch

```text
gt branch <command>
  create  delete  list  protect  rename  unprotect  view
```

Server-side branches (use `git` for local branches).

```bash
gt branch list
gt branch create release/1.0 --source main
gt branch protect main --required-approvals 2 --status-check ci/build
gt branch view main
```
