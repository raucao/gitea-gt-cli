# CI configuration: secrets, variables, packages

Actions secrets and variables and the package registry. Workflow files and
dispatch are in [actions.md](actions.md). `gt <cmd> --help` is authoritative.

**Scoping (secrets and variables):** repository (default), organization
(`--org`), or your account (`--account`). Names are stored **upper-cased**.

> The global `--user` flag selects the account `gt` acts as — it is **not** a
> scope. Use `--org` / `--account` for scope.

## secret

```text
gt secret <command>
  delete  list  set
```

Read by workflows as `${{ secrets.NAME }}`. Secret values are never shown.

- `set [<name>] [-b|--body value] [-F|--body-file file|-] [-d|--description]
  [-f|--env-file file|-] [-o|--org ORG] [--account]` — `--env-file` sets every
  `NAME=value` of a dotenv file. The value comes from the flag/file, else stdin
  when not a terminal (one trailing newline removed), else a prompt. Setting an
  existing secret without `--description` clears its description.
- `list [-L n] [--page n] [-o|--org ORG] [--account]` (alias `ls`)
- `delete <name> [-o|--org ORG] [--account] [-y|--yes]` (alias `rm`)

JSON: `created_at, description, name`.

```bash
gt secret set DEPLOY_TOKEN
echo -n "$TOKEN" | gt secret set NPM_TOKEN --org acme
gt secret set --env-file .env
gt secret list --org acme
```

## variable

```text
gt variable <command>
  delete  get  list  set  view
```

Read by workflows as `${{ vars.NAME }}`. Same scoping and naming rules as
secrets.

**JSON fields:** `description, name, value`.

- `set [<name>] [-b value] [-F file|-] [-d description] [-f|--env-file file|-]
  [-o|--org ORG] [--account]`
- `get <name> [...]` — print just the value (for scripts).
- `view <name>` — value plus description.
- `list [-L n] [--page n] [-o|--org ORG] [--account]` (alias `ls`)
- `delete <name> [-o|--org ORG] [--account] [-y|--yes]` (alias `rm`)

```bash
gt variable set ENVIRONMENT --body staging
gt variable set CONFIG --org acme < config.json
gt variable set --env-file .env
region=$(gt variable get REGION --org acme)
gt variable list --org acme
```

## package

```text
gt package <command>
  delete  link  list  unlink  upload  versions  view
```

Packages live in the registry of a user or organization: the owner of the
selected repository, else your account, unless `--owner` names another. A
package is addressed by **type** (e.g. `generic`, `npm`, `container`) and
**name**.

`package list` aliases `package ls`; `package delete` aliases `package rm`.

**JSON fields:** `created_at, creator, id, name, owner, repository, type, url,
version`; `view` adds `files`.

- `list [-t|--type type] [-S|--search text] [--owner OWNER] [-L n] [--page n]`
- `versions <type> <name> [--owner] [-L n] [--page n]`
- `view <type> <name> [<version>] [--owner]` — latest version without one.
- `upload <name> <version> <file>... [--owner]` — generic packages; each file
  is stored under its base name; a version cannot hold two files of the same
  name.
- `link <type> <name> <repository> [--owner]`
- `unlink <type> <name> [--owner]`
- `delete <type> <name> [<version>] [--all] [--owner] [-y|--yes]`

```bash
gt package upload tool 1.0.0 dist/tool-linux-amd64 dist/tool-darwin-arm64
gt package list --owner acme --type container
gt package versions generic tool
gt package delete generic tool 1.0.0
```
