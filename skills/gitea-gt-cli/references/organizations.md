# Organizations and teams

Organization management and team access. `gt <cmd> --help` is authoritative.
Global flags are in SKILL.md.

## org

```text
gt org <command>
  create  delete  edit  list  member  view
```

Organizations you belong to.

- `list`, `view <org>`
- `create <name> [-d description] [--full-name NAME] [--location LOC]
  [--website URL] [--visibility {public|limited|private}]
  [--repo-admins-can-change-team-access[=false]]`
- `edit <org>`, `delete <org>`
- `member {list|add|remove}` — `member list <org> [--public]`; Gitea lists every
  membership as public to administrators, so for them `gt` checks each one's
  visibility (one request per member) unless the output omits it.

JSON: `description, full_name, id, location, name, repo_admin_change_team_access,
visibility, website`.

```bash
gt org list
gt org create acme --full-name "ACME Corp"
gt org create acme-internal --visibility private
gt org member list acme --public
```

## team

```text
gt team <command>
  create  delete  edit  list  member  repo  view
```

Teams grant members access to an organization's repositories. In JSON, `access`
is the effective access shown in text (e.g. `write`, `"write (code, issues,
pulls)"`); `permission` is the team-wide level Gitea reports (`none` for a team
that grants access per unit).

**JSON fields:** `access, all_repositories, can_create_repo, description, id,
name, organization, permission, units`.

- `create <org> [<name>] [--name NAME] [--permission {read|write|admin}]
  [--unit repo.code=write,repo.wiki=read] [--all-repositories]
  [--can-create-repo] [-d description]` — `--permission` grants every repository
  unit; `--unit` sets single units and overrides `--permission` per unit.
- `edit <team>`, `delete <team>`, `view <team>`, `list <org>`
- `member {add|list|remove} <team> [<username>...]` — a team is `ORG/TEAM` or a
  numeric ID; adding a user also makes them an org member.
- `repo {add|list|remove} <team> [<repository>...]` — repositories named alone
  (`service`) or `ORG/NAME`.

```bash
gt team create acme developers --permission write
gt team create acme/reviewers --unit repo.code=read,repo.pulls=write
gt team member add acme/developers bob
gt team repo add acme/developers service
```
