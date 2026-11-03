# gitea-gt-cli

An [Agent Skill](https://agentskills.io) for **`gt`**, the official Gitea CLI
(the successor to `tea`), covering repositories, branches, issues, pull
requests, reviews, projects, releases, Actions runs/workflows/runners, secrets,
variables, packages, organizations, extensions, and authentication.

## Install

Listed on [skills.sh](https://skills.sh/raucao/gitea-gt-cli/gitea-gt-cli):

```bash
# Default (GitHub, tracks skills.sh)
npx skills add raucao/gitea-gt-cli

# Global / a specific agent
npx skills add raucao/gitea-gt-cli -a opencode -g

# From the upstream Gitea instance
npx skills add https://gitea.kosmos.org/raucao/gitea-gt-cli.git
```

Without the `skills` CLI, copy `SKILL.md` and the `references/` directory into
your agent's skills directory, for example `~/.agents/skills/gitea-gt-cli/` or
`.agents/skills/gitea-gt-cli/`.

## What's inside

The skill is the repository root:

```text
.
├── SKILL.md                  # when to use gt, auth, core concepts, command map, gotchas
├── references/
│   ├── workflows.md          # end-to-end recipes (issues, PRs, backports, Actions, projects, releases)
│   ├── auth.md               # auth, doctor, setup-git, env vars
│   ├── repos.md              # repo, branch, browse
│   ├── issues.md             # issue, label, milestone
│   ├── pull-requests.md      # pr (reviews, checks, diff, backport, merge, status)
│   ├── projects.md           # project
│   ├── releases.md           # release
│   ├── organizations.md      # org, team
│   ├── actions.md            # run, workflow
│   ├── runners.md            # runner
│   ├── ci-config.md          # secret, variable, package
│   ├── discovery.md          # search, status, notification, time
│   ├── instance.md           # config, api, alias, completion, webhook, wiki, ssh-key, admin
│   └── extensions.md         # extension, upgrade
├── README.md
└── LICENSE
```

`SKILL.md` stays small and maps each command group to a focused reference file,
so the agent loads only the detail it needs.

## Requirements

- `gt` 1.0.0 or newer (`gt version`)
- A Gitea instance running **28 or newer**

Install `gt` itself from the official source:
[gitea.com/gitea/cli](https://gitea.com/gitea/cli) (releases, install script and
source) or [dl.gitea.com/gitea-cli](https://dl.gitea.com/gitea-cli/) (prebuilt
binaries); `gt upgrade` keeps it current.

## License

[Unlicense](LICENSE) — public domain.
