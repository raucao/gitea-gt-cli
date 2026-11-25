# Projects (`gt project`)

Issue/PR project boards of a repository, an organization or your account.
`gt <cmd> --help` is authoritative. Global/repository-selection flags are in
SKILL.md.

```text
gt project <command>
  close  create  delete  edit  list  reopen  view  workflow
```

`project list` aliases `project ls`; `project create` aliases `project new`;
`project delete` aliases `project rm`.

**Scope:** the selected repository (default), an organization with
`--org ORG`, or your account with `--account`.

**Project argument:** a title, such as `Roadmap`, or an ID. A number is an ID
first, then a title; `--by-name` matches it as a title only.

**JSON fields:** `card_type, closed_issues, created_at, description, id,
open_issues, state, title, type, updated_at, url`.

## create

`create [<title>] [-t title] [-d description] [--card-type {text_only|images_and_text}]
[--template {none|basic_kanban|bug_triage}] [-f|--force]` — without a title on
a terminal, a prompt asks for it. `--template` creates the board's columns. A
project with the same title, ignoring case, is refused unless `--force` updates
it (titles address projects). Prints the project URL.

```bash
gt project create Roadmap --template basic_kanban
gt project create --title "Bug triage" --org acme --template bug_triage -d "Incoming bugs"
gt project create Roadmap --description "Planned work" --force
```

## list / view / edit

- `list [-s|--state {open|closed|all}] [-L n] [--page n]` (default state
  `open`).
- `view <project> [--by-name] [-w|--web]`.
- `edit <project> [--by-name] [-t title] [-d description]
  [--card-type {text_only|images_and_text}]`.

```bash
gt project list --org acme --state all
gt project view Roadmap
gt project edit 3 --title "Roadmap 2026" --card-type images_and_text
```

## close / reopen / delete

Each takes one or more projects; add `--by-name` to read numbers as titles.

```bash
gt project close Roadmap
gt project reopen 3 4 --org acme
gt project delete Roadmap --yes
```

## Moving issues on a board

`gt issue move` moves an issue between a board's columns (see
[issues.md](issues.md)). Moving issues on organization projects needs a token
that can write to the organization.

```bash
gt issue move 47 to "In progress"
gt issue move 47 in-progress --project Roadmap
```

## Project workflows (Gitea 29 or newer)

```text
gt project workflow <command>
  create  delete  disable  edit  enable  list  view
```

Automations that move, label, close or reopen the issues and pull requests of a
board when something happens to them. Workflows are addressed by ID; the same
`--org`, `--account` and `--by-name` scope flags as `project` apply.

`create <project> --event EVENT [...]` takes
`--event {item_added_to_project|item_removed_from_project|item_reopened|
item_closed|item_column_changed|code_changes_requested|code_review_approved|
pull_request_merged}`, `--column` and `--from-column`/`--to-column`,
`-l/--label` (only for items with all of them), `--add-label`/`--remove-label`,
`--issue-state {close|reopen|none}`, `--issue-type {issue|pull_request|any}`,
`--disabled` and `--dry-run`.

```bash
gt project workflow list Roadmap
gt project workflow create Roadmap --event item_added_to_project --column Todo
gt project workflow disable Roadmap 3
```
