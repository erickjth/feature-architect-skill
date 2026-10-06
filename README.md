# team-tools marketplace

## Install (local)
```
/plugin marketplace add ./feature-architect-marketplace
/plugin install architecture@team-tools
/reload-plugins
```
Then run `/architecture:feature-architect` or just ask Claude to plan a feature.

## Install (from git)
Push this folder to a repo, then:
```
/plugin marketplace add <owner>/<repo>
/plugin install architecture@team-tools
```

## Dev loop
`claude --plugin-dir ./plugins/architecture`, edit, then `/reload-plugins`.
