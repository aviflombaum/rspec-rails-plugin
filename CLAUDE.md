# CLAUDE.md

This is a Claude Code plugin for RSpec Rails testing.

## Repository Structure

```
rspec-rails-plugin/
├── .claude-plugin/
│   ├── marketplace.json     # Marketplace registry
│   └── plugin.json          # Plugin metadata
├── skills/
│   └── rspec/
│       └── SKILL.md         # Testing patterns and guidance
├── agents/
│   └── rspec-test-writer.md # Subagent for isolated test writing
├── README.md
├── CLAUDE.md
└── LICENSE
```

## Components

### Skill: `rspec-rails`

The main skill containing:
- Decision framework for spec type selection
- Code patterns for all spec types (model, request, system, job, mailer, channel, storage)
- Isolation patterns (VCR, time helpers, doubles)
- Fixture best practices
- DRY patterns (shared examples, custom matchers)

**Invocation**: `/rspec-rails:rspec` or auto-detected when working on Rails tests

### Agent: `rspec-test-writer`

A subagent for isolated, parallel test writing work. References the `rspec-rails` skill.

**Invocation**: `@rspec-test-writer` or via Task tool delegation

## Development

### Testing the Plugin Locally

```bash
# Run Claude with this plugin
claude --plugin-dir /path/to/rspec-rails-plugin

# Or symlink to plugins directory
ln -s /path/to/rspec-rails-plugin ~/.claude/plugins/rspec-rails
```

### Publishing Updates

1. Update version in `.claude-plugin/plugin.json`
2. Update CHANGELOG if present
3. Commit and push to GitHub
4. Users will get updates on next `/plugin update`

## Core Testing Principles

- Use fixtures, not factories
- Never modify rails_helper.rb or spec_helper.rb
- Don't test Rails internals
- One outcome per example
- Test behavior, not implementation
