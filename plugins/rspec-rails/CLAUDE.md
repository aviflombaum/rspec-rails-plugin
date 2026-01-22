# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Plugin Structure

```
rspec-rails/
├── .claude-plugin/
│   └── plugin.json           # Plugin metadata
├── skills/
│   └── write-test/
│       └── SKILL.md          # Testing patterns and guidance
├── agents/
│   └── rspec-test-writer.md  # Subagent for parallel test writing
├── CLAUDE.md
├── README.md
└── LICENSE
```

## Components

### Skill: `write-test`

Invocation: `/rspec-rails:write-test`

Contains decision framework for spec type selection, code patterns for all spec types (model, request, system, job, mailer, channel, storage), isolation patterns (VCR, time helpers, doubles), fixture best practices, and DRY patterns.

**Argument hint:** `[model|request|system|job|mailer|channel] ClassName`

**Auto-invoked when:** User asks to "write tests", "add specs", or asks Rails testing questions.

### Agent: `rspec-test-writer`

A subagent for isolated, parallel test writing. Delegates via Task tool. Preloads the `write-test` skill.

## Core Testing Principles

- Use fixtures, not factories
- Never modify rails_helper.rb or spec_helper.rb
- Don't test Rails internals
- One outcome per example
- Test behavior, not implementation
