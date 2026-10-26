# Pull requests (`gt pr`)

Pull requests, including reviews, checks, diff, checkout and merge.
`gt <cmd> --help` is authoritative. Global/repository-selection flags are in
SKILL.md.

```text
gt pr <command>
  attachment  checkout  checks  clean  close  comment  create  diff
  edit  list  merge  ready  reopen  review  status  timeline  view
```

`pr list` aliases `pr ls`; `pr create` aliases `pr new`; `pr checkout` aliases
`pr co`.

**PR argument:** `13`, `#13`, a URL, or a head branch name (e.g. `feature/login`
or `bob:feature/login`). Without one, the PR of the current branch.

**JSON fields (PR):** `assignees, author, base, body, comments, created_at,
draft, due_date, head, head_remote_deleted, head_repo, head_sha, id, labels,
merge_commit_sha, mergeable, merged, merged_at, merged_by, milestone, number,
review_decision, reviewer_teams, reviewers, reviews, state, title, updated_at,
url`. On `list`, `reviewers`, `reviewer_teams`, `reviews`, `review_decision` and
`merged_by` cost extra requests and are fetched only when named.

## list

`-s, --state {open|closed|merged|all}` (default `open`), `-a/--assignee`,
`-A/--author`, `--no-assignee`, `-l/--label`, `-m/--milestone`,
`-B/--base branch`, `-H/--head branch|OWNER:BRANCH`, `-d/--draft`,
`-S/--search`, `--sort`, `-L/--limit`, `--page`, `-w/--web`.

`review_decision` is `changes_requested`, `approved`, `review_required` or
empty.

```bash
gt pr list --author @me --state all
gt pr list --json number,title,head --jq '.[] | select(.head == "feature/login")'
```

## create

`--fill` (title/body from commits), `-t/--title`, `-b/--body`,
`-F/--body-file`, `-e/--editor`, `-B/--base`, `-H/--head`, `-d/--draft`,
`-a/--assignee`, `-r/--reviewer`, `--reviewer-team`, `-l/--label`,
`-m/--milestone`, `-p/--project`, `--deadline`, `--attachment`,
`--allow-maintainer-edits` (default true), `--agit` (+ `--topic`), `-w/--web`,
`-y/--yes`.

Creates from the current branch, or `--head`. Pushes a not-yet-remote branch
after a confirmation (`--yes` in CI). A branch without a same-named upstream
goes to the fork when origin is a fork, or to the only remote of your own fork;
otherwise to the repository's remote. Any flag skips the interactive questions;
the title is then `--title`, else the commit message of a single-commit branch,
else the branch name.

```bash
gt pr create --fill --yes
gt pr create -t "Add login" -b "Closes #1" -B develop -r carol -l enhancement
```

## checkout / diff / checks

- `checkout {N|url|branch} [-b branch] [--detach] [--update]` — local branch
  tracking the head, so `git push` updates it. `--update` fast-forwards an
  existing branch and rejects divergence. JSON: `branch, commit, detached,
  remote, tracking`.
- `diff [{N|url|branch}] [--name-only] [--color auto|always|never] [-w]` — clean
  diff when piped (`gt pr diff 13 | git apply`); colored and paged on a
  terminal.
- `checks [{N|url|branch}] [--watch] [--interval d] [--timeout d] [-w]` — latest
  status of each check. Exits **8** while pending, **1** on failure. JSON:
  `created_at, description, id, name, sha, state, updated_at, url`.

```bash
gt pr checkout 13 --update
gt pr diff 13 | git apply
gt pr checks 13 --watch && gt pr merge 13
```

## merge

`-s/--method {merge|rebase|rebase-merge|squash|fast-forward-only}` (default:
repo's style), shortcuts `--merge`, `--rebase`, `--squash`, `--auto`
(merge once required checks succeed), `-d/--delete-branch` (also deletes the
local head branch when checked out, switching to the base), `-t/--title`,
`-b/--message`, `-F/--message-file`, `-e/--editor`.

```bash
gt pr merge 13 --squash --delete-branch
gt pr merge 13 --auto --yes
```

## review

Choose the type with `-a/--approve`, `-r/--request-changes` (needs a body), or
`-c/--comment`. Flags: `-b/--body`, `-F/--body-file`, `-e/--editor`,
`--pending` (save for later), `--commit sha`,
`--comments-file file` (JSON array of `path`, `body`, `new_position` or
`old_position`), `-y/--yes`. Subcommands: `create, list, view, delete, submit,
request, cancel-request, comments, reply, resolve, unresolve, dismiss,
undismiss`. JSON: `author, body, comments_count, commit, dismissed, id,
official, stale, state, submitted_at, team, url`.

On a terminal, no type opens an interactive review that can annotate the diff in
your editor (lines you start with `#` become inline comments).

```bash
gt pr review --approve
gt pr review 13 -r -b "Please hash the password"
gt pr review 13 --pending --comments-file inline.json
gt pr review submit 13 --approve
```

## status / clean / ready / comment / view / edit

- `status [--limit n]` — current branch's PR with checks/review, then your
  authored/assigned/review-requested PRs. JSON: `assigned, authored,
  current_branch, review_requested`.
- `clean [{...}] [--ignore-sha]` — delete the branch of a merged/closed PR
  locally and on Gitea. JSON: `default_branch, local_branch, remote,
  remote_branch, remote_deleted, remote_skip_reason, remote_skipped`.
- `ready [{...}] [--undo]` — mark ready/back to draft (toggles `WIP:`/`[WIP]`).
- `comment [{...}]` — like `issue comment` (JSON: `author, body, created_at, id,
  issue_number, updated_at, url`); subcommands `list|view|edit|delete`.
- `view [{...}] [-c|--comments] [-w]`, `edit`, `close`, `reopen`, `timeline`,
  `attachment`.

```bash
gt pr status
gt pr ready
gt pr clean 13
```
