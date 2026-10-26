# auth (`gt auth`)

Authentication, accounts, git credential setup, and environment variables.
`gt <cmd> --help` is authoritative.

`gt` keeps accounts in `hosts.yml` and tokens in the system keyring (or a
private file), under `~/.config/gitea-cli/` (`GITEA_CLI_CONFIG_DIR` overrides).
It never reads `tea`'s config. One account per instance is active; `--user` or
`GITEA_CLI_USER` selects another for a single command.

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

Create a token on the instance under **Settings > Applications**. The first
instance logged in to becomes `default_host` unless one is set.

### status / token / switch / setup-git / logout / refresh

- `status [--verify] [--json ...]` — accounts per instance, active one, method.
  Fails with exit **4** when there is no account or the credential is
  missing/rejected; with `--json` it exits 0 and you read
  `credential_available`, `verified`, `error`. JSON fields: `active,
  credential_available, credential_store, error, expires_at,
  git_credential_helper, host, method, scopes, user, verified`.
- `token` — print the active token (refreshing an expiring OAuth token). SSH-key
  accounts have no token.
- `switch` — change the active account of an instance.
- `setup-git [--keep-helpers] [--local]` — configure git to ask `gt` for
  credentials. Other credential helpers are turned off for the instance unless
  `--keep-helpers`; `--local` configures only the current repository.
- `logout` — log out of an account.
- `refresh` — refresh an OAuth token.

```bash
gt auth status --verify
gt auth status --host gitea.com --json
curl -H "Authorization: token $(gt auth token)" https://gitea.com/api/v1/user
```

## Environment variables

- `GITEA_TOKEN` + `GITEA_INSTANCE_URL` (or `GITEA_CLI_HOST`): use a token
  instead of stored credentials. `GITEA_TOKEN` replaces the active account of
  its instance for the active account too; it must belong to the account
  `--user`/`GITEA_CLI_USER` names.
- `GITEA_CLI_*` prefixed variables mirror global flags and settings; see
  `gt help environment` for the full list (config dir, user, repo, editor,
  pager, color, http timeout, ...).
- `GITEA_INSTANCE_INSECURE`, `GITEA_INSTANCE_SSH_HOST`: pair with `GITEA_TOKEN`.

## doctor

`gt doctor [-R ...] [--json ...]` checks the setup commands in this directory
use: settings files, the instance and its headers, whether it answers, the
account and credential, the repository, and git. Each problem names the command
that fixes it; `gt doctor` fails when a check fails. JSON fields: `checks,
host, host_source, repo, repo_source, user, user_source`.

```bash
gt doctor
gt doctor -R gitea.com/gitea/cli
gt doctor --jq '.checks[] | select(.status == "failed")'
```
