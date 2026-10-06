# team-tools marketplace

One plugin, `erickjth`, holds all skills. Each skill is invoked as `/erickjth:<skill-name>`.

## Skills
- `/erickjth:feature-architect` - plan, build and review a feature

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
