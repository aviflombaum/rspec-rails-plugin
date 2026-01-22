# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Structure

This is a Claude Code plugin **marketplace** repository containing the `rspec-rails` plugin.

```
rspec-rails-plugin/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace definition, lists available plugins
├── plugins/
│   └── rspec-rails/              # The actual plugin
│       ├── .claude-plugin/
│       │   └── plugin.json       # Plugin metadata (name, version, etc.)
│       ├── skills/
│       │   └── rspec/
│       │       └── SKILL.md      # RSpec testing patterns and guidance
│       ├── agents/
│       │   └── rspec-test-writer.md
│       ├── CLAUDE.md
│       ├── README.md
│       └── LICENSE
└── README.md                     # Marketplace README
```

## Architecture

**Marketplace vs Plugin**: The repo root defines a marketplace (via `.claude-plugin/marketplace.json`) which can contain multiple plugins. Each plugin lives in `plugins/<plugin-name>/` with its own `.claude-plugin/plugin.json`.

**Component Auto-Discovery**: Claude Code automatically discovers `skills/`, `agents/`, and `commands/` directories within each plugin. No need to explicitly list them in plugin.json.

## Publishing Updates

1. Update version in both:
   - `plugins/rspec-rails/.claude-plugin/plugin.json`
   - `.claude-plugin/marketplace.json`
2. Commit and push to GitHub
3. Users get updates via `/plugin update rspec-rails`

## Testing Locally

```bash
claude --plugin-dir ./plugins/rspec-rails
```
