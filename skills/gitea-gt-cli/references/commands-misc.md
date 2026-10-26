# Command reference: misc

Remaining `gt` groups: `team`, `notification`, `search`, `status`, `time`,
`webhook`, `wiki`, `ssh-key`, `api`, `config`, `alias`, `completion`.

`gt <group> <cmd> --help` is authoritative for the installed version.

---

## team

```text
gt team <command>
  create  delete  edit  list  member  repo  view
```

Teams grant members access to an organization's repositories. In JSON, `access`
is the effective access shown in text (e.g. `write`, `"write (code, issues,
pulls)"`); `permission` is the team-wide level Gitea reports (`none` for a team
that grants access per unit).

**JSON fields:** `access, all_repositories, can_create_repo, description, id,
name, organization, permission, units`.

- `create <org> [<name>] [--name NAME] [--permission {read|write|admin}]
  [--unit repo.code=write,repo.wiki=read] [--all-repositories]
  [--can-create-repo] [-d description]`
- `edit`, `delete`, `view`, `list`
- `member {add|list|remove}` — adding a user to a team of an org also makes them
  an org member.
- `repo {add|list|remove}` — repositories named alone (`service`) or `ORG/NAME`.

```bash
gt team create acme developers --permission write
gt team create acme/reviewers --unit repo.code=read,repo.pulls=write
gt team member add acme/developers bob
gt team repo add acme/developers service
```

---

## notification

```text
gt notification <command>
  list  mark-read  mark-unread  pin  unpin  view  watch
```

Across your whole inbox unless `-R`/`--this-repo` selects a repository. **IDs
are notification thread IDs, not issue numbers.** `--type` is `issue`, `pull`,
`commit`, or `repository`.

**JSON fields:** `id, number, pinned, repository, state, title, type, unread,
updated_at, url`.

- `list [--status unread,read,pinned] [--type ...] [--since/--before when]
  [-L n] [--page n] [--this-repo]` (alias `ls`)
- `view <id> [--this-repo]` — marks unread as read.
- `mark-read [<id>...] [--all] [--status ...] [--before when] [--this-repo]`,
  `mark-unread`, `pin`, `unpin`.
- `watch [--type ...] [--status ...] [--interval 5s] [--timeout d] [--json]` —
  streams as they arrive; on a terminal runs until Ctrl-C, otherwise stops after
  1 minute unless `--timeout` is given.

```bash
gt notification list
gt notification list --type pull --since 24h
gt notification mark-read --all -R alice/project
gt notification watch --type pull --json id,repository,title --timeout 0 | ./notify.sh
```

---

## search

```text
gt search <command>
  issue  pr  repo
```

Searches the selected instance within what your account can see. Words match
titles and bodies.

- `search repo [<query>...] [--topic] [--owner OWNER] [--type {fork|source|mirror}]
  [--archived[=false]] [--private[=false]] [--sort {alpha|created|updated|size|id}]
  [--order {asc|desc}] [-L n] [--page n]`
- `search issue [<query>...] [--assigned] [--authored] [--mentioned]
  [-A author] [-l label] [-m milestone] [--owner] [--team team] [-s state]
  [--since/--until when] [-L n] [--page n]` (alias `search issues`)
- `search pr [<query>...]` — like issue plus `--review-requested`, `--reviewed`
  (instance-wide) (alias `search prs`).

`search issue`/`pr` JSON: `author, body, created_at, id, number, repository,
state, title, type, updated_at, url`.

```bash
gt search repo command line
gt search repo --owner acme --archived
gt search issue "login error"
gt search issue crash -R alice/project --label bug --milestone v1.0
gt search pr --review-requested
```

---

## status

```text
gt status [flags]
```

Open issues and PRs assigned to you, reviews requested from you, and discussions
mentioning you; in a clone it also lists failing Actions runs. `-R` scopes
everything to one repository.

Flags: `--limit n` (1..100, default 30), `--runs` (default true).

JSON: `assigned_issues, failing_runs, host, mentions, pull_requests,
repository, runs_included, runs_repository, user`.

```bash
gt status
gt status -R alice/project --runs=false
```

---

## time

```text
gt time <command>
  create  delete  list  reset
```

Log time on issues/PRs. Durations use `h`, `m`, `s` (`1h25m` or `"1h 25m"`).
`create` aliases `add`/`log`; `list` aliases `ls`; `delete` aliases `rm`.

**JSON fields:** `created_at, id, issue_number, issue_title, issue_url,
repository, seconds, user`.

- `create {issue} <duration>`
- `list [--issue N|URL] [--mine] [--username user|@me] [--since/--until when]
  [--total] [-L n] [--page n]`
- `delete {issue} <time-id> [-y]`
- `reset {issue}` — delete your logged time on an issue.

```bash
gt time create 12 1h25m
gt time list --issue 12
gt time list --mine --since 168h
gt time delete 12 8
```

---

## webhook

```text
gt webhook <command>
  create  delete  edit  list  view
```

Scope: repository (default), organization (`--org`), your account (`--account`),
or the instance (`--global`, administrators). Delivery URLs are shown without
credentials/path unless `view --reveal-url`. Signing secrets and authorization
headers are never shown.

**JSON fields:** `active, branch_filter, content_type, created_at, events, id,
type, updated_at, url`.

- `create [<url>] [--url URL] [--type {gitea|gogs|slack|discord|dingtalk|telegram|msteams|feishu|matrix|wechatwork|packagist}]
  [--events push,issues,...] [--branch-filter glob] [--active[=false]]
  [--secret-file file|-] [--authorization-header-file file|-] [--config k=v]`
- `list [-L n] [--page n]` (alias `ls`)
- `view <id> [--reveal-url]`, `edit <id>`, `delete <id>`

Events include: `create, delete, fork, push, issues, issues_only,
issue_assign, issue_label, issue_milestone, issue_comment, pull_request,
pull_request_only, pull_request_assign, pull_request_label,
pull_request_milestone, pull_request_comment, pull_request_review,
pull_request_sync, pull_request_review_request, wiki, repository, release,
package, status, workflow_run, workflow_job`.

```bash
gt webhook create https://ci.example.com/hook --events push,pull_request --secret-file secret.txt
gt webhook list --org acme
gt webhook view 3 --reveal-url
```

---

## wiki

```text
gt wiki <command>
  create  delete  edit  list  revisions  view
```

**JSON fields (page):** `author, body, commit_count, footer, sha, sidebar,
title, updated_at, url`; `list` returns `author, sha, title, updated_at, url`.

- `create [<title>] [-t title] [-b body] [-F file|-] [-e] [-m message]`
- `edit <title> [...]`, `view <title>`, `delete <title>`,
  `list [-L n] [--page n]` (alias `ls`), `revisions <title>`

```bash
gt wiki create "Getting Started" --body-file docs/start.md
gt wiki list -R alice/project --json title,url
gt wiki view Home
```

---

## ssh-key

```text
gt ssh-key <command>
  create  delete  list  view
```

Manage the SSH public keys of your account. `create` aliases `add`; `list`
aliases `ls`.

**JSON fields:** `created_at, fingerprint, id, key, read_only, title`.

- `create [<file>|-] [-t|--title TITLE]` — without a file, reads stdin.
- `list [-L n] [--page n]`, `view <id>`, `delete <id>`

```bash
gt ssh-key add ~/.ssh/id_ed25519.pub --title "Work laptop"
gt ssh-key list --json title,fingerprint
```

---

## api

```text
gt api <endpoint> [flags]
```

Authenticated request to the instance API (path under `/api/v1`, or a full API
URL). `{owner}`, `{repo}` expand to the selected repository; `{branch}` to the
checked-out branch.

| Flag | Meaning |
| --- | --- |
| `-X, --method` | HTTP method (default GET; POST when a body is supplied) |
| `-f, --raw-field k=v` | String parameter (query for GET/HEAD, JSON body otherwise) |
| `-F, --field k=v` | Typed value: `true`/`false`/`null`, numbers, arrays, `@FILE`, `@-` |
| `-H, --header name:value` | Request header |
| `--input file` | Request body from a file, or `-` |
| `--paginate`, `--max-pages n` | Follow next-page links / list paging |
| `-i, --include` | Print status/headers to stderr |
| `--output-file file` | Write the whole response to a file |
| `--jq expr` | Filter each JSON response (jq not required) |
| `--silent`, `--verbose` | No body / request+response to stderr |

Non-2xx responses print the body and fail with the HTTP status.

```bash
gt api repos/{owner}/{repo}/issues -f state=open
gt api user -F name=alice -F admin=true
gt api repos/{owner}/{repo}/branches --paginate --jq '.[].name'
```

---

## config

```text
gt config <command>
  get  list  path  set
```

Preferences live in `config.yml`, instance settings in `hosts.yml`, in the
configuration directory (`$GITEA_CLI_CONFIG_DIR`, else `gitea-cli` under the XDG
config dir). `gt config path` prints both files.

- `get <key> [--show-source]`, `list` (alias `ls`, add `--host` for instance
  settings), `set <key> <value> [--host URL] [--user NAME]`.
- Keys: `default_host`, `git_protocol`, `editor`, `prompt`, `pager`, `browser`,
  `color`, `hyperlinks`, `markdown`, `remote`, `http_timeout`; instance:
  `user`, `ssh_host`, `insecure`, `git_protocol`, `oauth_client_id`, `headers`,
  `client_cert`, `client_key`; account: `ssh_key`, `ssh_agent`,
  `ssh_certificate_principal`, `ssh_key_fingerprint`.

```bash
gt config path
gt config set editor "code --wait"
gt config set ssh_host git.example.com:2222 --host https://example.com
gt config get default_host --show-source
```

---

## alias

```text
gt alias <command>
  delete  list  set
```

An alias expands to a `gt` command with arguments, before `gt` runs it. `$1`,
`$2`, ... take positional arguments; other arguments follow the expansion. No
shell is run; built-ins always win.

- `set <name> <expansion> [--clobber]`
- `list` (alias `ls`), `delete [<name>...] [--all]` (alias `rm`)

```bash
gt alias set il "issue list --label bug"
gt alias set review 'pr review $1 --approve'
gt alias list
```

---

## completion

```text
gt completion <command>
  bash  fish  powershell  zsh
```

Generate the shell completion script. Completion offers `--json` field names.

```bash
gt completion bash > /etc/bash_completion.d/gt
gt completion zsh > "${fpath[1]}/_gt"
```
