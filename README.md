# claude-skills

Claude Code plugins by Luca Kohlhepp, listed in the Mayflower marketplace
(`mayflower/agent-plugins`).

| Plugin | Skills |
|---|---|
| `plugins/impl-skills` | `impl-plan`, `implement`, `impl-debrief` — plan-to-code workflow with reviewed commits |
| `plugins/reasoning-skills` | `debate`, `distill`, `fermi-estimate` — reasoning primitives |

Install via the marketplace:

```
claude plugin marketplace add mayflower/agent-plugins
claude plugin install impl-skills@agent-plugins
claude plugin install reasoning-skills@agent-plugins
```

Each plugin is a directory with `.claude-plugin/plugin.json` and its skills under
`skills/<name>/SKILL.md`.
