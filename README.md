# gitea-gt-cli

An [Agent Skill](https://agentskills.io) for **`gt`**, the official Gitea CLI
(the rewrite of `tea`), covering repositories, issues, pull requests, reviews,
releases, Actions, packages, organizations, and authentication.

## Install

```bash
npx skills add https://gitea.kosmos.org/raucao/gitea-gt-cli.git

# Specific agent / global
npx skills add https://gitea.kosmos.org/raucao/gitea-gt-cli.git -a opencode -g
```

Without the `skills` CLI, copy `skills/gitea-gt-cli/` into your agent's skills
directory, for example `~/.agents/skills/gitea-gt-cli/` or
`.agents/skills/gitea-gt-cli/`.

## What's inside

```text
skills/gitea-gt-cli/
├── SKILL.md                  # when to use gt, auth, core concepts, command map, gotchas
└── references/
    ├── workflows.md          # end-to-end recipes (issues, PRs, Actions, releases, fork/sync)
    ├── auth.md               # auth, doctor, setup-git, env vars
    ├── repos.md              # repo, branch
    ├── issues.md             # issue, label, milestone
    ├── pull-requests.md      # pr (reviews, checks, diff, merge, status)
    ├── releases.md           # release
    ├── organizations.md      # org, team
    ├── actions.md            # run, workflow
    ├── ci-config.md          # secret, variable, package
    ├── discovery.md          # search, status, notification, time
    └── instance.md           # config, api, alias, completion, webhook, wiki, ssh-key, admin
```

`SKILL.md` stays small and maps each command group to a focused reference file,
so the agent loads only the detail it needs.

## Requirements

- `gt` 0.1.0 or newer (`gt version`)
- A Gitea instance running **28 or newer**

Install `gt` itself with:

```bash
curl -fsSL https://gitea.com/gitea/cli/raw/branch/main/scripts/install.sh | sh
```

## License

[Unlicense](LICENSE) — public domain.
