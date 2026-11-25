# Extensions and upgrades (`gt extension`, `gt upgrade`)

Install, run and remove extensions — executables named `gt-NAME` that run as
`gt NAME` — and keep `gt` itself up to date. `gt <cmd> --help` and
`gt help extensions` are authoritative. Global flags are in SKILL.md.

```text
gt extension <command>        (alias: gt ext)
  browse  create  exec  install  list  remove  search  upgrade
gt upgrade
```

Extensions come from repositories on Gitea instances, or as GitHub CLI
extensions named `gh-NAME` from github.com. Built-in commands always win over
an extension of the same name; `gt extension exec` runs it anyway. An extension
can declare aliases in its `gt-extension.yml`, so its commands run as top-level
`gt` commands without every user setting them up with `gt alias set`.

## install

`install <repository> [--pin tag] [--allow-credentials] [--compat-gh]
[--force] [-y|--yes]` — the repository is `OWNER/REPO`, `HOST/OWNER/REPO`, a
URL on a Gitea instance, or `https://github.com/OWNER/gh-NAME` for a GitHub CLI
extension. `.` installs the `gt-NAME` executable of the current directory,
which runs in place while you develop it, as long as it (or what it links to)
is a regular file owned by you or root that other users cannot write to or
replace.

`gt` installs the release asset `NAME-OS-ARCH` or `REPO-OS-ARCH` (`.exe` on
Windows), else one whose name ends in `OS-ARCH`, of the latest release or of
the release `--pin` names. Without such an asset it clones the repository at
the release, the `--pin` commit or the default branch, which must hold an
executable `gt-NAME` at its root. The commit an install resolves is checked to
be a full commit ID before it is shown or passed to git.

`gt` checks a downloaded asset against the SHA-256 GitHub recorded for it, else
against the release's `checksums.txt`. When the release publishes neither, `gt`
asks with a default of no, or installs with `--yes`. It records the SHA-256 and
refuses to run the extension once the executable changes.

- `--allow-credentials` gives the extension your Gitea token (see security).
- `--compat-gh` lets the extension call `gh`, which `gt` answers for `gh api`,
  `gh auth token` and `gh repo view` (with `gt`'s fields, such as `full_name`).
- `--pin` installs a release tag or commit and never upgrades it.

```bash
gt extension install alice/gt-hello
gt extension install https://gitea.com/alice/gt-hello --pin v1.2.0
gt extension install https://github.com/owner/gh-hello
gt extension install .
```

## list / search / browse / remove / upgrade

- `list` (alias `ls`) — JSON: `commit, compat_gh, credentials, description,
  digest_source, host, kind, missing, name, path, pinned, repository, sha256,
  shadowed, source, tag`. `missing` marks a local extension whose executable
  no longer exists.
- `search [<query>...] [--index repository]` — searches an index repository's
  `extensions.yml` by name and description; the default index is
  `https://gitea.com/gitea/cli-extensions`. JSON: `description, installed,
  name, platforms, repository`.
- `browse [<name>] [--index repository]` — open the extension's repository, or
  the index, in the browser.
- `remove <name>... [--yes]` (alias `rm`).
- `upgrade {<name>... | --all} [--dry-run] [--force] [--yes]` — shows the old
  and new version and digest first, refuses a repository that was transferred
  or recreated and a release tag whose asset changed, and never upgrades in the
  background. Extensions installed with `--pin`, and local ones, are not
  upgraded; reinstall them with `install --force`.

```bash
gt extension list --json name,tag,credentials
gt extension search review
gt extension remove hello --yes
gt extension upgrade --all --dry-run
```

## create / exec

- `create <name> [--precompiled=go]` — creates the directory `gt-NAME` with a
  script extension, or with `--precompiled=go` a Go program and a workflow that
  publishes it as release assets with a `checksums.txt`.
- `exec <name> [<args>...]` — runs an extension even when a built-in command
  has its name. Global flags go before `exec`.

```bash
gt extension create hello
gt extension create hello --precompiled=go
gt --repo alice/project extension exec hello --greeting hi
```

## Security model

> This is hygiene, not a sandbox. An extension runs with your privileges, can
> read `gt`'s configuration directory, and can change its own files and
> manifest. Install extensions only from sources you trust.

- Global flags work before the extension's name: `--host`, `--repo`, `--user`
  become `GITEA_CLI_HOST`, `GITEA_CLI_REPO`, `GITEA_CLI_USER`; `--cwd`,
  `--no-input` and `--yes` are accepted. Other global flags, such as `--json`,
  are refused. Everything after the extension's name is passed on unchanged.
- The extension inherits `gt`'s terminal and environment except for
  `GITEA_TOKEN`, `GITEA_INSTANCE_*` and the GitHub tokens/host, which are
  withheld unless you allowed credentials at install. It also gets
  `GITEA_CLI_EXTENSION` (its name) and `GITEA_CLI_EXECUTABLE` (the path of `gt`).
- `gt` refuses stored credentials to an extension that was not given them, and
  checks the executable against the SHA-256 recorded at install.
- A local extension runs only when its executable, or what it links to, is a
  regular file owned by you or root that not every user can write to or
  replace. A missing one points to `gt extension remove`.
- `GITEA_CLI_NO_EXTENSIONS` turns extensions off entirely.

## Upgrade `gt` itself

`gt upgrade [--dry-run] [--force] [--nightly] [--version release] [-y|--yes]`
replaces `gt` with its latest release, a named release, or the latest build of
the main branch (`--nightly`), after verifying the download's SHA-256 checksum.
`GITEA_CLI_DOWNLOAD_URL` selects another download location. When Homebrew, a
Linux package or Nix installed `gt`, it names the command that upgrades it
instead. A `gt` built with `go install` cannot upgrade itself; install a release
instead.

After a command, `gt` tells you, at most once a day, when a newer release is
out. It stays quiet in CI and when stdout or stderr is not a terminal;
`GITEA_CLI_NO_UPDATE_NOTIFIER` turns the check off.

```bash
gt upgrade
gt upgrade --dry-run
gt upgrade --version 1.0.0 --yes
gt upgrade --nightly
```

## Environment and configuration

- `GITEA_CLI_DATA_DIR` — data directory; installed extensions live in its
  `extensions/` (default `gitea-cli` under the XDG data directory).
- `GITEA_CLI_NO_EXTENSIONS`, `GITEA_CLI_NO_UPDATE_NOTIFIER`,
  `GITEA_CLI_DOWNLOAD_URL` — described above.
- `config.yml` preferences: `extensions` (`enabled`/`disabled`),
  `extension_hosts`, `extension_owners`. An administrator can set
  `extension_credentials: denied` and the same policy in
  `/etc/gitea-cli/policy.yml` (`%ProgramData%\gitea-cli\policy.yml` on
  Windows), which you cannot loosen.
