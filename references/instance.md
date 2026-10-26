# Instance: webhooks, wiki, keys, API, config, admin

Instance- and account-level commands. `gt <cmd> --help` is authoritative.
Global flags are in SKILL.md.

## webhook

```text
gt webhook <command>
  create  delete  edit  list  view
```

Scope: repository (default), organization (`--org`), your account (`--account`),
or the instance (`--global`, administrators). Delivery URLs often carry
credentials, so `gt` shows them without user info and query (and without the path
for chat services); `view --reveal-url` shows a URL whole. Signing secrets and
authorization headers are never shown.

**JSON fields:** `active, branch_filter, content_type, created_at, events, id,
type, updated_at, url`.

- `create [<url>] [--url URL] [--type {gitea|gogs|slack|discord|dingtalk|telegram|msteams|feishu|matrix|wechatwork|packagist}]
  [--events push,issues,...] [--branch-filter glob] [--active[=false]]
  [--secret-file file|-] [--authorization-header-file file|-] [--config k=v]`
- `list [-L n] [--page n]` (alias `ls`)
- `view <id> [--reveal-url]`, `edit <id>`, `delete <id>`

Events include: `create, delete, fork, push, issues, issues_only, issue_assign,
issue_label, issue_milestone, issue_comment, pull_request, pull_request_only,
pull_request_assign, pull_request_label, pull_request_milestone,
pull_request_comment, pull_request_review, pull_request_sync,
pull_request_review_request, wiki, repository, release, package, status,
workflow_run, workflow_job`.

```bash
gt webhook create https://ci.example.com/hook --events push,pull_request --secret-file secret.txt
gt webhook list --org acme
gt webhook view 3 --reveal-url
```

## wiki

```text
gt wiki <command>
  create  delete  edit  list  revisions  view
```

**JSON fields (page):** `author, body, commit_count, footer, sha, sidebar,
title, updated_at, url`; `list` returns `author, sha, title, updated_at, url`.

- `create [<title>] [-t title] [-b body] [-F file|-] [-e] [-m message]`
- `edit <title> [...]`, `view <title>`, `delete <title>`,
  `list [-L n] [--page n]` (alias `ls`), `revisions <title> [--page n]`

```bash
gt wiki create "Getting Started" --body-file docs/start.md
gt wiki list -R alice/project --json title,url
gt wiki view Home
```

## ssh-key

```text
gt ssh-key <command>
  create  delete  list  view
```

Manage the SSH public keys of your account. `create` aliases `add`; `list`
aliases `ls`.

**JSON fields:** `created_at, fingerprint, id, key, read_only, title`.

- `create [<file>|-] [-t|--title TITLE]` — without a file, reads stdin; the
  title defaults to the key comment, then the computer's name.
- `list [-L n] [--page n]`, `view <id>`, `delete <id>`

```bash
gt ssh-key add ~/.ssh/id_ed25519.pub --title "Work laptop"
gt ssh-key list --json title,fingerprint
```

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

## alias

```text
gt alias <command>
  delete  list  set
```

An alias expands to a `gt` command with arguments, before `gt` runs it. `$1`,
`$2`, ... take positional arguments; other arguments follow the expansion. No
shell is run; built-in commands always win over an alias of the same name.

- `set <name> <expansion> [--clobber]`
- `list` (alias `ls`), `delete [<name>...] [--all]` (alias `rm`)

```bash
gt alias set il "issue list --label bug"
gt alias set review 'pr review $1 --approve'
gt alias list
```

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

## admin

```text
gt admin <command>
  user  (list, view, create, edit, delete)
```

Instance administration; requires an administrator account. (`--sudo user` on
other commands acts as another user.) JSON: `active, admin, created_at,
description, email, full_name, id, location, prohibit_login, restricted, url,
username, visibility, website`.

```bash
gt admin user list
gt admin user list --query bob
gt admin user create alice --email alice@example.com
```
