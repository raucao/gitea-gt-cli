# Command reference: Actions

`gt` treats Gitea Actions as first-class. Groups: `run` (workflow runs), `workflow`
(definitions and dispatch), `secret`, `variable`, `package` (registry), and `admin`.

`gt <group> <cmd> --help` is authoritative for the installed version.

**Run argument:** a run ID (`34`, `#34`) or a run URL
(`https://HOST/OWNER/REPO/actions/runs/34`). Without one, a terminal asks you to
pick a recent run.

**JSON fields (run):** `actor, attempt, branch, commit, completed_at, conclusion,
created_at, event, id, jobs, run_number, started_at, status, title, updated_at,
url, workflow, workflow_path`.

---

## run

```text
gt run <command>
  artifact  cancel  delete  download  jobs  list  logs  rerun  view  watch
```

`run list` aliases `run ls`.

### list

`-u/--actor @me`, `-b/--branch`, `-c/--commit SHA|rev`, `-e/--event`,
`-w/--workflow file|name`, `-s/--status` (status or conclusion, repeatable),
`--since`/`--until` (date, RFC3339, or duration ago; scans up to 5000 runs),
`-L/--limit`, `--page`.

```bash
gt run list
gt run list --workflow CI --branch main --status failure
gt run list --since 24h --json id,status,conclusion,workflow
```

### view / watch

- `view [<run-id>] [--log] [--no-jobs] [-w]` — run, jobs and steps.
- `watch [<run-id>] [-i|--interval 3s] [--timeout 1h]` — refresh until done;
  **fails when the run completes without success**; tolerates a 5-minute outage.

```bash
gt run view 34
gt run view 34 --json jobs --jq '.jobs[] | {name, conclusion}'
gt run watch 34 --timeout 10m && echo deployed
```

### logs

`logs [<run-id>] [-j|--job name|ID] [-f|--follow] [--failed]
[-i|--interval 5s] [-O|--output file|-] [-D|--dir dir] [--timeout 1h]
[--clobber] [--skip-existing]`

Prints each job's logs, job names to stderr. `--follow` waits for jobs and new
lines. `--output`/`--dir` download the whole run's logs as a ZIP.

```bash
gt run logs 34
gt run logs 34 --job build --follow
gt run logs 34 --output logs.zip
```

### download

`download [<run-id>] [-n|--name artifact] [-D|--dir dir] [-O|--output path|-]
[--zip] [--clobber] [--skip-existing]` — extracts each artifact into a
directory named after it; a single `--name` extracts into the destination
itself; `--zip` keeps archives.

```bash
gt run download 34
gt run download 34 --name build --dir dist
gt run download 34 --zip
```

### jobs / artifact list / cancel / rerun / delete

- `jobs [<run-id>] [-L n] [--page n]` — JSON: `completed_at, conclusion,
  created_at, id, labels, name, run_attempt, started_at, status, steps, url`.
- `artifact list [<run-id>]` — list uploaded artifacts.
- `cancel [<run-id>] [--force]` — keeps the run and logs.
- `rerun [<run-id>] [--failed] [-j|--job name|ID]` — all jobs, failed only, or one.
- `delete [<run-id>]`.

```bash
gt run jobs 34 --json id,conclusion --jq '.[] | select(.conclusion == "failure") | .id'
gt run rerun 34 --failed
```

---

## workflow

```text
gt workflow <command>
  disable  enable  list  run  view
```

`workflow list` aliases `workflow ls`.

**Workflow argument:** file name (`ci.yml`), name without extension (`ci`),
path (`.gitea/workflows/ci.yml`), or workflow name (`CI`).

**JSON fields:** `id, name, path, state, url`.

### run (dispatch)

`gt workflow run <workflow>` with `-r/--ref branch|tag`,
`-f/--field key=value` (repeatable), `--follow`, `--interval 2s`,
`--timeout 30m`. Requires a `workflow_dispatch` trigger. Inputs are validated
against the declared inputs: unknown, missing-required, or invalid choice/boolean
values are refused. The workflow must exist on the default branch (like `gh`);
`--ref` selects the version that runs. Without `--ref`, runs on the checked-out
branch if it is on the server.

JSON fields: `id, url`.

```bash
gt workflow run CI
gt workflow run ci.yml --ref main -f target=production -f dry_run=false
gt workflow run nightly --follow
```

### view / list / enable / disable

- `view <workflow> [-r ref] [--yaml] [-w]` — workflow and recent runs.
- `list`
- `enable <workflow>`, `disable <workflow>`

```bash
gt workflow view CI
gt workflow view ci.yml --yaml --ref feature/login
gt workflow disable Nightly
```

---

## secret

```text
gt secret <command>
  delete  list  set
```

Secrets are read by workflows as `${{ secrets.NAME }}`. Scope: repository
(default), organization (`--org`), or your account (`--account`). Names are
stored **upper-cased**.

> The global `--user` flag selects the account `gt` acts as — it is **not** a
> scope. Use `--org` / `--account` for scope.

### set

`gt secret set [<name>]` with `-b/--body value`, `-F/--body-file file|-`,
`-d/--description`, `-f/--env-file file|-` (sets every `NAME=value`).
The value comes from the flag/file, else stdin when not a terminal (one
trailing line break removed), else a prompt.

JSON: `created_at, description, name`.

```bash
gt secret set DEPLOY_TOKEN
echo -n "$TOKEN" | gt secret set NPM_TOKEN --org acme
gt secret set --env-file .env
```

### list / delete

- `list [-L n] [--page n] [-o|--org ORG]` (alias `ls`)
- `delete <name> [-o|--org ORG] [--account] [-y|--yes]` (alias `rm`)

```bash
gt secret list --org acme
gt secret delete DEPLOY_TOKEN --org acme --yes
```

---

## variable

```text
gt variable <command>
  delete  get  list  set  view
```

Variables are read by workflows as `${{ vars.NAME }}`. Same scoping and naming
rules as secrets (`--org`, `--account`; names upper-cased).

**JSON fields:** `description, name, value`.

### set / get / view / list / delete

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

---

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
  is stored under its base name.
- `link <type> <name> <repository> [--owner]`
- `unlink <type> <name> [--owner]`
- `delete <type> <name> [<version>] [--all] [--owner] [-y|--yes]`

```bash
gt package upload tool 1.0.0 dist/tool-linux-amd64 dist/tool-darwin-arm64
gt package list --owner acme --type container
gt package versions generic tool
gt package delete generic tool 1.0.0
```

---

## admin

```text
gt admin <command>
  user  (list, view, create, edit, delete)
```

Instance administration; requires an administrator account. (`--sudo user` on
other commands acts as another user.)

```bash
gt admin user list
gt admin user create alice --email alice@example.com
```
