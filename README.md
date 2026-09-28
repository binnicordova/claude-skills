# claude-skills

Reusable Claude skills by Binni Cordova.

| Skill | What it does |
|---|---|
| [`play-store-aso`](play-store-aso/SKILL.md) | Audits and rewrites Google Play listings (name, short/full description, category, tags, language) with the CoCap "3 steps first" formula, checks policy risks, and sends changes for review to grow installed audience. |
| [`expo-go-module`](expo-go-module/SKILL.md) | Builds Expo modules with no native code (pure TypeScript) that run in Expo Go and ship over expo-updates on iOS, Android and Web: scaffolding with `bun create expo-module`, patterns for web, testing and optional dependencies, an Expo Go example app, verification, the README and npm publishing. Based on [`expo-logs`](https://github.com/binnicordova/expo-logs). |

## Install

- **Claude Code (per project):** copy the skill folder into `.claude/skills/` of your repo:
  ```bash
  mkdir -p .claude/skills && cp -r play-store-aso expo-go-module .claude/skills/
  ```
- **Claude Code (all projects):** copy it into `~/.claude/skills/`.
- **Claude.ai / Cowork:** Settings → Capabilities → Skills → upload a zip of the skill folder.

Then ask, for example:

- *"Improve the Play Store listing of com.example.app"*
- *"Create an Expo Go–compatible module called expo-haptic-patterns, based on react-native-haptic-feedback"*
