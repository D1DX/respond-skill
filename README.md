# Respond.io Skill

[![Author](https://img.shields.io/badge/Author-Daniel_Rudaev-000000?style=flat)](https://github.com/daniel-rudaev)
[![Studio](https://img.shields.io/badge/Studio-D1DX-000000?style=flat)](https://d1dx.com)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat)](./LICENSE)

Claude Code skill for working with the Respond.io unofficial internal API (`app.respond.io`). Covers operations missing from the public API: snippets, custom fields, closing note categories, and AI settings.

## What's Inside

- **Snippets** — list, create, update, delete message templates
- **Custom Fields** — update and delete custom contact fields
- **Closing Notes** — list categories, set required fields
- **Respond AI** — get/update AI Assist and AI Agent settings
- **Authentication** — session cookie + CSRF token extraction

## Usage

1. Copy `SKILL.md` to your Claude Code skills directory (`~/.claude/skills/`)
2. The skill auto-triggers on Respond.io tasks

## License

MIT License — Copyright (c) 2026 Daniel Rudaev @ D1DX
