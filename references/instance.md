# Instance: webhooks, wiki, keys, API, config, admin

Instance- and account-level commands. `gt <cmd> --help` is authoritative.
Global flags are in SKILL.md.

## webhook

```text
gt webhook <command>
  create  delete  edit  list  test  view
```

Scope: repository (default), organization (`--org`), your account (`--account`),
or the instance's system webhooks (`--global`, administrators). Delivery URLs
often carry credentials, so `gt` shows them without user info and query (and
without the path for chat services); `view --reveal-url` shows a URL whole.
Signing secrets and authorization headers are never shown.

**JSON fields:** `active, branch_filter, content_type, created_at, events, id,
type, updated_at, url`.

- `create [<url>] [--type {gitea|gogs|slack|discord|dingtalk|telegram|msteams|feishu|matrix|wechatwork|packagist}]
  [--events push,issues,...] [--branch-filter glob] [--active[=false]]
  [--secret-file file|-] [--authorization-header-file file|-] [--config k=v]`
- `edit <id>` — only the flags you pass change; Gitea drops the stored
  authorization header on every edit, so pass it again (or `--clear-authorization`).
  Signing secrets cannot be changed: recreate the webhook.
- `list [-L n] [--page n]` (alias `ls`), `view <id> [--reveal-url]`,
  `delete <id>`, `test <id>` — send a test push delivery (repository webhooks
  only).

Events include: `create, delete, fork, push, issues, issues_only, issue_assign,
issue_label, issue_milestone, issue_comment, pull_request, pull_request_only,
pull_request_assign, pull_request_label, pull_request_milestone,
pull_request_comment, pull_request_review, pull_request_sync,
pull_request_review_request, wiki, repository, release, package, status,
workflow_run, workflow_job`.

```bash
gt webhook create https://ci.example.com/hook --events push,pull_request --secret-file secret.txt
gt webhook list --org acme
gt webhook test 3
gt webhook view 3 --reveal-url
```

## wiki

```text
gt wiki <command>
  create  delete  edit  list  revisions  view
```

**JSON fields (page):** `author, body, commit_count, footer, sha, sidebar,
title, updated_at, url`; `list` returns `author, sha, title, updated_at, url`;
`revisions` returns `author, created_at, message, sha`.

- `create [<title>] [-t title] [-b body] [-F file|-] [-e] [-m message]`
- `edit <page> [...]` (rename with `--title` alone), `view <page> [-w]`,
  `delete <page>`, `list [-L n] [--page n]` (alias `ls`),
  `revisions <page> [--page n]`

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
aliases `ls`; `delete` aliases `rm`.

**JSON fields:** `created_at, fingerprint, id, key, read_only, title`.

- `create [<file>|-] [-t|--title TITLE]` — without a file, reads stdin; the
  title defaults to the key comment, then the computer's name.
- `list [-L n] [--page n]`, `view <id>`, `delete <id> [-y]`

```bash
gt ssh-key add ~/.ssh/id_ed25519.pub --title "Work laptop"
gt ssh-key list --json title,fingerprint
```

## gpg-key

```text
gt gpg-key <command>
  create  delete  list  token  verify  view
```

Manage the GPG public keys of your account. `create` aliases `add`; `list`
aliases `ls`; `delete` aliases `rm`.

**JSON fields:** `can_certify, can_encrypt_comms, can_encrypt_storage,
can_sign, created_at, emails, expires_at, id, key_id, primary_key_id,
public_key, subkeys, verified`.

- `create [<file>|-] [--signature-file file|-]` — add an ASCII-armored public
  key (`gpg --armor --export KEYID`); without a file it reads stdin. A key that
  names no activated email address is refused unless `--signature-file` signs
  what `gt gpg-key token` prints.
- `list [-L n] [--page n]`, `view <id>`, `delete <id>` — keys are addressed by
  the ID that `gpg-key list` shows, not by the GPG key ID.
- `token` — print the token to sign; it changes every minute.
- `verify <key-id> [<file>]` — verify a key Gitea holds as unverified, for all
  your activated email addresses; `<file>` is an ASCII-armored detached
  signature, read from stdin without one.

```bash
gpg --armor --export alice@example.com | gt gpg-key add -
gt gpg-key list --json key_id,emails,verified
gt gpg-key token | gpg --armor --detach-sign > token.asc
gt gpg-key verify 3 token.asc
```

## schema

`gt schema [<command>...]` prints the JSON fields of every command (or the named
ones) as JSON: `schema_version` and `commands[]`, each with `command` and
`fields[]` of `name`, `type`, `nullable`, `format` and `items`. `format:
date-time` marks timestamps. `--jq` filters the report; there is no `--json`.

```bash
gt schema issue list
gt schema --jq '.commands[] | select(.command == "gt pr view") | .fields[].name'
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
| `--paginate`, `--max-pages n` | Follow next-page links / list paging (default max 1000 pages) |
| `--slurp` | With `--paginate`, print the pages as one JSON array (not with `--jq`) |
| `-i, --include` | Print status/headers to stderr |
| `--output-file file` | Write the whole response to a file |
| `--jq expr` | Filter each JSON response (jq not required) |
| `--silent`, `--verbose` | No body / request+response to stderr |
| `--dry-run` | Show the write requests instead of sending them |

Non-2xx responses print the body and fail with the HTTP status.

```bash
gt api repos/{owner}/{repo}/issues -f state=open
gt api user -F name=alice -F admin=true
gt api repos/{owner}/{repo}/branches --paginate --jq '.[].name'
gt api repos/{owner}/{repo}/issues --paginate --slurp > issues.json
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
- Preference keys: `default_host`, `git_protocol`, `editor`, `prompt`, `pager`,
  `browser`, `color`, `hyperlinks`, `markdown`, `remote`, `http_timeout`,
  `extensions`, `extension_hosts`, `extension_owners`, `aliases`; instance
  (`--host`): `user`, `ssh_host`, `insecure`, `git_protocol`,
  `oauth_client_id`, `headers`, `client_cert`, `client_key`; account (`--host`
  + `--user`): `ssh_key`, `ssh_agent`, `ssh_certificate_principal`,
  `ssh_key_fingerprint`.

`gt` reads settings files written by a later version or holding settings it
does not know, and keeps those settings when it rewrites the file, so several
`gt` versions can share one configuration. `gt config set` stores an unknown key
with a warning instead of failing; `gt` keeps it but does not use it.

```bash
gt config path
gt config set editor "code --wait"
gt config set ssh_host git.example.com:2222 --host https://example.com
gt config set extension_hosts https://gitea.com/gitea
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
- `list` (alias `ls`), `delete {<name>... | --all} [-y]` (alias `rm`)

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

Generate the shell completion script. Completion offers `--json`/`--csv` field
names.

```bash
gt completion bash > /etc/bash_completion.d/gt
gt completion zsh > "${fpath[1]}/_gt"
```

## admin

```text
gt admin <command>
  cron  email  org  user
```

Instance administration; requires an administrator account. (`--sudo user` on
other commands acts as another user.) JSON for accounts: `active, admin,
created_at, description, email, full_name, id, location, prohibit_login,
restricted, url, username, visibility, website`.

- `user {list|view|create|edit|delete|rename|key|badge}` — `user list [--query
  text] [-L n] [--page n]`, `user view <username>`,
  `user create [<username>] [--email addr] [--password-file file]
  [--full-name NAME] [--admin] [--must-change-password[=false]]
  [--prohibit-login] [--restricted] [--visibility ...]`,
  `user edit <username> [...]` (as above plus `--active[=false]`,
  `--allow-*-`/`--max-repo-creation`, `--location`, `--website`),
  `user delete <username> [--purge] [-y]`, `user rename <username>
  <new-username>`, `user key {create|delete}`, `user badge
  {add|delete|list}`.
- `org {list|create}` — `org create <name> --owner USER [...]` creates an
  organization for a user; `org list` shows every organization.
- `cron {list|run <task>}` — list scheduled tasks or start one now.
- `email list [--query text]` — email addresses of all accounts.

```bash
gt admin user list --query bob
gt admin user create alice --email alice@example.com
gt admin cron run cleanup-todos
gt admin user delete spammer --purge
```
