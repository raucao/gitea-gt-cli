# auth (`gt auth`)

Authentication, accounts, git credential setup, and environment variables.
`gt <cmd> --help` is authoritative.

`gt` keeps accounts in `hosts.yml` and tokens in the system keyring (or a
private file), under `~/.config/gitea-cli/` (`GITEA_CLI_CONFIG_DIR` overrides).
Keyring entries use the service `gitea-cli:HOST[/PATH]` with the account name;
tokens stored by 1.x are moved the first time 2.0 uses them, after which 1.x can
no longer read them. It never reads `tea`'s config. One account per instance is
active; `--user` or `GITEA_CLI_USER` selects another for a single command.

## Commands

```text
gt auth <command>
  login  logout  refresh  setup-git  status  switch  token
```

### login

Without a method flag on a terminal it asks for the instance, login method,
token storage and whether git should use `gt`.

| Flag | Meaning |
| --- | --- |
| `--with-token` | Read an access token from stdin (or ask on a terminal) |
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

Create a token on the instance under **Settings > Applications**. A token works
only for what its scopes allow: `read:repository`/`write:repository` for
repositories, pull requests and releases, `read:issue`/`write:issue` for issues,
labels and milestones, and so on. Logging in needs `read:user` to learn whose
token it is. The first instance logged in to becomes `default_host` unless one
is set.

### status / token / switch / setup-git / logout / refresh

- `status [--verify] [--json fields]` — accounts per instance, active one,
  method. Fails with exit **4** when there is no account, the credential is
  missing/rejected, or the token lacks a needed scope, and **1** when `--verify`
  gets no answer; with `--json` it exits 0 either way, so read
  `credential_available`, `verified`, `error`. JSON fields: `active,
  credential_available, credential_store, error, expires_at,
  git_credential_helper, host, method, scopes, user, verified`.
- `token` — print the active token (refreshing an expiring OAuth token). SSH-key
  accounts have no token.
- `switch` — change the active account of an instance.
- `setup-git [--keep-helpers] [--local]` — configure git to ask `gt` for
  credentials. Other credential helpers are turned off for the instance unless
  `--keep-helpers`; `--local` configures only the current repository.
- `logout`, `refresh`.

```bash
gt auth status --verify
gt auth status --host gitea.com --json host,user,credential_available,verified,error
curl -H "Authorization: token $(gt auth token)" https://gitea.com/api/v1/user
```

## Environment variables

- `GITEA_TOKEN` + `GITEA_INSTANCE_URL` (or `GITEA_CLI_HOST`): use a token
  instead of stored credentials. `GITEA_TOKEN` replaces the active account of
  its instance for the active account too; it must belong to the account
  `--user`/`GITEA_CLI_USER` names.
- In Gitea Actions, `GITEA_TOKEN` is enough: `gt` picks the instance from the
  job (`GITEA_SERVER_URL`), like `gh` on GitHub.
- **Narrowly scoped tokens:** without `read:user`, every command that does not
  need to know whose token it is runs (including the git credential helper),
  while `@me`, `--user`, a repository named without its owner and commands about
  your own account fail with exit **4**. A 403 for a missing scope also exits
  **4**.
- `GITEA_CLI_ERROR_FORMAT=json`: print failures as one line of JSON on stderr
  instead of text; `kind` is `failure`, `usage`, `canceled`, `authentication`,
  `unsuccessful`, `head_mismatch` or `pending`.
- `GITEA_CLI_*` prefixed variables mirror global flags and settings; see
  `gt help environment` for the full list (config dir, user, repo, editor,
  pager, color, http timeout, data dir, extensions, ...).
- `GITEA_INSTANCE_INSECURE`, `GITEA_INSTANCE_SSH_HOST`: pair with `GITEA_TOKEN`.

## doctor

`gt doctor [-R ...] [--json fields]` checks the setup commands in this directory
use: settings files, the instance and its headers, whether it answers, the
account and credential, the repository, and git. Each problem names the command
that fixes it; `gt doctor` fails when a check fails. JSON fields: `checks,
host, host_source, repo, repo_source, user, user_source`.

```bash
gt doctor
gt doctor -R gitea.com/gitea/cli
gt doctor --jq '.checks[] | select(.status == "failed")'
```
