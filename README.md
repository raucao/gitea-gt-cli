# gitea-gt-cli

An [Agent Skill](https://agentskills.io) for **`gt`**, the official Gitea CLI
(the rewrite of `tea`), covering repositories, issues, pull requests, reviews,
releases, Actions, packages, organizations, and authentication.

## Install

```bash
# HTTPS
npx skills add https://gitea.kosmos.org/raucao/gitea-gt-cli.git

# SSH
npx skills add ssh://git@gitea.kosmos.org/raucao/gitea-gt-cli.git

# Specific agent / global
npx skills add https://gitea.kosmos.org/raucao/gitea-gt-cli.git -a claude-code -g
```

The `skills` CLI supports any git host, so this works from Gitea. It will not be
listed on skills.sh, which only indexes GitHub repositories.

Without the `skills` CLI, copy `skills/gitea-gt-cli/` into your agent's skills
directory, for example `~/.agents/skills/gitea-gt-cli/` or
`.agents/skills/gitea-gt-cli/`.

## What's inside

```text
skills/gitea-gt-cli/
├── SKILL.md                     # when to use gt, auth, core concepts, workflows, gotchas
└── references/
    ├── commands-core.md         # auth, repo, issue, pr, release, org, label, milestone, branch
    ├── commands-actions.md      # run, workflow, secret, variable, package, admin
    └── commands-misc.md         # team, notification, search, status, time, webhook, wiki, ...
```

`SKILL.md` stays small and points at the reference file for each command group,
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
