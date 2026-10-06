# team-tools marketplace

One plugin, `erickjth`, holds all skills. Each skill is invoked as `/erickjth:<skill-name>`.

## Skills
- `/erickjth:feature-architect` - plan, build and review a feature
- `/erickjth:pr-explainer` - explain a PR as an interactive HTML page
- `/erickjth:feature-tracker` - living progress tracker for a Linear parent ticket with sub-tickets

## Install (local)
```
/plugin marketplace add ./feature-architect-marketplace
/plugin install erickjth@team-tools
/reload-plugins
```

## Install (from git)
```
/plugin marketplace add <owner>/<repo>
/plugin install erickjth@team-tools
```

## Add a new skill
Create `plugins/erickjth/skills/<new-skill>/SKILL.md` (frontmatter `name` + `description`), bump `version` in
`plugins/erickjth/.claude-plugin/plugin.json`, then `/reload-plugins`. No other file needs to change.

## Dev loop
`claude --plugin-dir ./plugins/erickjth`, edit, then `/reload-plugins`.

## Notes
- `pr-explainer` and `feature-tracker` optionally call `/mattpocock-skills:code-review` (a separate plugin); both fall back to their built-in `references/review-checklist.md` if it is not installed.
- `feature-tracker` reads Linear (and GitHub for diffs), so connect those MCP servers.
