# Actions: runs and workflows

Gitea Actions are first-class in `gt`. This file covers `run` (workflow runs)
and `workflow` (definitions and dispatch). Secrets, variables and packages are
in [ci-config.md](ci-config.md). `gt <cmd> --help` is authoritative.

**Run argument:** a run ID (`34`, `#34`) or a run URL
(`https://HOST/OWNER/REPO/actions/runs/34`). Without one, a terminal asks you to
pick a recent run.

**JSON fields (run):** `actor, attempt, branch, commit, completed_at, conclusion,
created_at, event, id, jobs, run_number, started_at, status, title, updated_at,
url, workflow, workflow_path`.

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

Prints each job's logs, job names to stderr. `--follow` waits for jobs to start
and prints new lines. `--output`/`--dir` download the whole run's logs as a ZIP.

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
branch if it is on the server, else the default branch. With `--follow`, fails
like `gt run watch` when the run completes without success.

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
