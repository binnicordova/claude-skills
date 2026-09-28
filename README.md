# claude-skills

Reusable Claude skills by Binni Cordova.

| Skill | What it does |
|---|---|
| [`play-store-aso`](play-store-aso/SKILL.md) | Audits and rewrites Google Play listings (name, short/full description, category, tags, language) with the CoCap "3 steps first" formula, checks policy risks, and sends changes for review to grow installed audience. |

## Install

- **Claude Code (per project):** copy the folder into `.claude/skills/` of your app repo:
  ```bash
  mkdir -p .claude/skills && cp -r play-store-aso .claude/skills/
  ```
- **Claude Code (all projects):** copy it into `~/.claude/skills/`.
- **Claude.ai / Cowork:** Settings → Capabilities → Skills → upload a zip of the `play-store-aso` folder.

Then ask: *"Improve the Play Store listing of com.example.app"*.
