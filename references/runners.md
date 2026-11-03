# Actions runners (`gt runner`)

Manage the Actions runners that execute workflow jobs: list them, see what they
are doing, enable or disable them, delete them, and print the token that
registers new ones. `gt <cmd> --help` is authoritative. Global/repository
selection flags are in SKILL.md.

```text
gt runner <command>
  delete  disable  enable  list  token  view
```

`runner list` aliases `runner ls`; `runner delete` aliases `runner rm`.

**Scope:** the selected repository (default), an organization with
`--org ORG`, your account with `--account`, or the whole instance with
`--global` (administrators). A list shows only the runners of its scope, not
the organization and instance runners that also take a repository's jobs;
`--global` shows every runner.

**JSON fields:** `busy, disabled, ephemeral, id, labels, name, status`.

> A runner's registration token is a secret: anyone with it can register a
> runner that then receives the jobs and secrets of that scope. Treat the text
> `gt runner token` prints as sensitive.

## list / view

- `list [--disabled | --enabled] [-L n] [--page n]`.
- `view <id>`.

```bash
gt runner list
gt runner list --org acme --disabled
gt runner list --json name,status,labels --jq '.[] | select(.status == "online")'
gt runner view 3 --org acme --json labels,status
```

## enable / disable / delete

- `enable <id>` — let the runner take jobs again.
- `disable <id>` — stop it from taking new jobs; a job it already runs
  finishes.
- `delete <id> [--yes]` — remove it from Gitea; the runner program keeps
  running but can no longer take jobs until it registers again.

```bash
gt runner enable 3
gt runner disable 3 --org acme
gt runner delete 4 --org acme --yes
```

## token

`token` prints the registration token of the repository, organization, account
or instance. Gitea creates it on first use and returns the same one until it is
reset in the web interface. JSON field: `token`.

```bash
gt runner token
gitea-runner register --no-interactive --instance https://gitea.example.com \
  --token "$(gt runner token --org acme)"
```

Runners interact with runs covered in [actions.md](actions.md).
