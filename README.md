# gitea-gt-cli

An [Agent Skill](https://agentskills.io) for **`gt`**, the official Gitea CLI
(the rewrite of `tea`), covering repositories, issues, pull requests, reviews,
releases, Actions, packages, organizations, and authentication.

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
│   ├── workflows.md          # end-to-end recipes (issues, PRs, Actions, releases, fork/sync)
│   ├── auth.md               # auth, doctor, setup-git, env vars
│   ├── repos.md              # repo, branch
│   ├── issues.md             # issue, label, milestone
│   ├── pull-requests.md      # pr (reviews, checks, diff, merge, status)
│   ├── releases.md           # release
│   ├── organizations.md      # org, team
│   ├── actions.md            # run, workflow
│   ├── ci-config.md          # secret, variable, package
│   ├── discovery.md          # search, status, notification, time
│   └── instance.md           # config, api, alias, completion, webhook, wiki, ssh-key, admin
├── README.md
└── LICENSE
```

`SKILL.md` stays small and maps each command group to a focused reference file,
so the agent loads only the detail it needs.

## Requirements

- `gt` 0.1.0 or newer (`gt version`)
- A Gitea instance running **28 or newer**

Install `gt` itself from the official source:
[gitea.com/gitea/cli](https://gitea.com/gitea/cli) (releases and source) or
[dl.gitea.com/gitea-cli](https://dl.gitea.com/gitea-cli/) (prebuilt binaries).

## License

[Unlicense](LICENSE) — public domain.
