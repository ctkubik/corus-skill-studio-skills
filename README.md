# Skill Studio skill catalog (Corus Orthodontists)

The skill catalog Skill Studio installs from (Settings → Skills, or the
first-run wizard's "Install skills" step).

## Layout

One folder per skill at the top level. Each folder holds a `SKILL.md` with
YAML frontmatter, plus any files the skill needs:

```
my-skill/
  SKILL.md
  references/...
```

`SKILL.md` starts with:

```markdown
---
name: my-skill
description: One line on what the skill does and when to use it.
---
```

## Adding a skill

Copy the skill folder into the root of this repo, commit, and push. The app
picks it up the next time someone installs or refreshes the catalog.

This repo is public so the app can read it without a GitHub login. Don't put
API keys, passwords, or patient data in a skill.
