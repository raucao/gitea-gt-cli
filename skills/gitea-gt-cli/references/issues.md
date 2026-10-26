# Issues, labels, milestones

Issue management plus repository/organization labels and milestones.
`gt <cmd> --help` is authoritative. Global/repository-selection flags are in
SKILL.md.

## issue

```text
gt issue <command>
  attachment  close  comment  create  edit  list  reopen  timeline  view
```

`issue list` aliases `issue ls`; `issue create` aliases `issue new`.

**Issue argument:** `123`, `#123`, a URL, or `OWNER/REPO#123`.

**JSON fields (issue):** `assignees, author, body, comments, created_at,
due_date, id, kind, labels, milestone, number, ref, state, title, updated_at,
url`. `view`/`edit` add `blocked_by, blocks` (extra requests, only when named).

Gitea lists pull requests among issues, so most issue commands also accept a PR
number.

### list

| Flag | Meaning |
| --- | --- |
| `-s, --state {open\|closed\|all}` | default `open` |
| `-a, --assignee`, `-A, --author`, `--mentions` | `@me` for yourself |
| `--no-assignee` | only issues without an assignee |
| `-l, --label names`, `-m, --milestone titles` | filters |
| `--kind {issues\|pulls\|all}` | include PRs |
| `-S, --search text` | search titles and bodies |
| `--since`, `--until` | updated window (date, timestamp, or duration ago) |
| `-L, --limit n` (1..1000, default 30), `--page n` | pagination |
| `-w, --web` | open in browser |

```bash
gt issue list --label bug --state all
gt issue list --assignee @me --json number,title
gt issue list --kind all --milestone v1.0 | cut -f1,2
```

### create

`-t/--title`, `-b/--body`, `-F/--body-file file|-`, `-e/--editor`,
`-a/--assignee login`, `-l/--label name` (prefix `repo:`/`org:` to disambiguate),
`-m/--milestone`, `-p/--project`, `--ref`, `--deadline DATE`,
`-T/--template name`, `--attachment File`, `-w/--web`.

Without flags on a terminal, a form asks for title and body, then optionally
assignees, a milestone, labels and a due date. Gitea applies
labels/milestone/assignees/deadline only to users with issue write access;
otherwise `gt` reports the left-out fields and fails.

```bash
gt issue create --title "Crash on start" --body "Steps..." \
  --label bug --milestone v1.0
gt issue create --template "Bug report"
```

### edit

Adds/removes relative to current values, or replaces with `--set-*`:
`--add-label`, `--remove-label`, `--set-labels`, `--add-assignee`,
`--remove-assignee`, `--set-assignees`, `-m/--milestone`,
`--remove-milestone`, `--add-blocked-by`, `--remove-blocked-by`,
`--add-project`, `--remove-project`, `-t/--title`, `-b/--body`,
`-F/--body-file`, `-e/--editor`, `--ref`, `--deadline`. Accepts multiple issues.

```bash
gt issue edit 12 --title "Crash on start" --add-label bug
gt issue edit 12 13 --add-assignee @me --milestone v1.0
gt issue edit 12 --add-blocked-by 9
```

### close / reopen / comment / timeline / view / attachment

- `close {N}... [-c comment]`, `reopen {N}...`
- `comment {N} [-b text] [-F file] [-e] [--edit-last] [--delete-last]
  [--attachment File]`; subcommands `list|view|edit|delete`. JSON: `author,
  body, created_at, id, issue_number, updated_at, url`.
- `timeline {N} [-L n] [--page n]` — comments and changes, oldest first.
- `view {N} [-c|--comments] [-w]`
- `attachment {list|view|upload|download|edit|delete}`

```bash
gt issue view 12 --comments
gt issue comment 12 --body "Fixed in #14"
gt issue close 12 13 --comment "Done in #14"
```

## label

```text
gt label <command>
  create  delete  edit  export  import  list
```

Scope with `--org organization` for organization labels; otherwise the
repository's. Addressed by name or ID. JSON: `color, description, exclusive,
id, is_archived, name`.

- `create <name> [-c color] [-d description] [--exclusive] [--archived]
  [-f/--force]` — random color without `-c`; `--force` updates on name clash.
- `list`, `edit`, `delete`
- `export [-O|--output file]` and `import <file> [-f|--force]` — a label file,
  one label per line: `#color name ;description ;flags`. `gt label export`
  writes this format; Gitea label templates use it. `-` reads/writes stdin.

```bash
gt label create bug --color d73a4a --description "Something is broken"
gt label create "priority/high" --exclusive
gt label export --org acme --output labels.txt
gt label export -R alice/old | gt label import - -R alice/new
```

## milestone

```text
gt milestone <command>
  close  create  delete  edit  issue  list  reopen  view
```

`milestone list` aliases `milestone ls`; `milestone create` aliases
`milestone new`. Addressed by title or ID. `milestone issue` manages the
issues/PRs in it. JSON: `closed_issues, description, due_on, id, open_issues,
state, title`.

- `create [<title>] [-t title] [-d description] [-F desc-file] [-e]
  [--deadline date] [--state {open|closed}] [-f/--force]` — `--force` updates an
  existing milestone of the same title.
- `list [-s {open|closed|all}] [-L n] [--page n]`, `view`, `edit`, `close`,
  `reopen`, `delete`
- `issue list <milestone>` etc.

```bash
gt milestone create v1.0 --deadline 2026-12-31 --description "First release"
gt milestone list --state open
gt milestone issue list v1.0
```
