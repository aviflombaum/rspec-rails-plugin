# RSpec Rails Plugin for Claude Code

A Claude Code plugin providing comprehensive RSpec testing patterns and an AI-powered test writer for Ruby on Rails applications.

## What's Included

| Component | Type | Purpose |
|-----------|------|---------|
| `write-test` | Skill | Testing patterns, decision framework, code examples |
| `rspec-test-writer` | Agent | Isolated test writing with parallel execution |

## Installation

### Via Claude Code Plugin Marketplace

```bash
# Add the marketplace (one-time)
/plugin marketplace add aviflombaum/rspec-rails-plugin

# Install the plugin
/plugin install rspec-rails
```

### Manual Installation

Clone into your Claude plugins directory:

```bash
git clone https://github.com/aviflombaum/rspec-rails-plugin.git ~/.claude/plugins/rspec-rails
```

## Usage

### Skill: `/rspec-rails:write-test`

The skill provides testing patterns and guidance. Claude will automatically invoke it when you're working on Rails tests, or you can invoke it manually:

```
/rspec-rails:write-test

# With argument hint:
/rspec-rails:write-test model User
/rspec-rails:write-test request RecipesController

# Or ask naturally:
"Write tests for the User model"
"Add specs for the orders controller"
"Help me write a system spec for user registration"
```

### Agent: `@rspec-test-writer`

Use the agent when you want isolated, parallel test writing:

```
@rspec-test-writer Write comprehensive tests for the Order model

@rspec-test-writer Create request specs for the API endpoints in app/controllers/api/
```

## What It Covers

### Spec Types
- **Model specs** - Validations, scopes, instance/class methods
- **Request specs** - HTTP routing, authentication, authorization
- **System specs** - Full UI workflows with Capybara + Cuprite
- **Job specs** - ActiveJob queuing, execution, error handling
- **Mailer specs** - Email content, headers, delivery
- **Channel specs** - ActionCable subscriptions, broadcasts
- **Storage specs** - ActiveStorage uploads, attachments

### Testing Strategies
- **Fixtures** - Rails fixtures over factories
- **Isolation** - VCR for HTTP, time helpers, verifying doubles
- **DRY patterns** - Shared examples, custom matchers
- **Decision framework** - Which spec type for which scenario

## Requirements

Your Rails application should have:

```ruby
group :test do
  gem 'rspec-rails'
  gem 'cuprite'        # For JavaScript system specs
  gem 'vcr'            # For HTTP isolation
  gem 'webmock'        # VCR dependency
end
```

## Core Principles

1. **Use fixtures, not factories** - `users(:alice)` not `create(:user)`
2. **Never modify rails_helper.rb** - Assume RSpec is configured
3. **Don't test Rails internals** - Associations and validations work
4. **One outcome per example** - Focused, clear tests
5. **Test behavior, not implementation**

## Quick Reference

```
What am I testing?
├── Data & Business Logic    → Model specs
├── HTTP & Controllers       → Request specs
├── User Interface           → System specs
├── Background Processing    → Job specs
├── Email                    → Mailer specs
├── File Uploads             → Storage specs
├── Real-time Features       → Channel specs
└── External Services        → VCR + isolation
```

## Examples

### Model Spec
```ruby
RSpec.describe User, type: :model do
  it "requires an email" do
    user = User.new(email: nil)
    expect(user).to be_invalid
    expect(user.errors[:email]).to include("can't be blank")
  end
end
```

### Request Spec
```ruby
RSpec.describe "Recipes", type: :request do
  it "creates recipe for authenticated user" do
    user = users(:alice)
    expect {
      post recipes_path(as: user), params: { recipe: { name: "Pie" } }
    }.to change(user.recipes, :count).by(1)
  end
end
```

### System Spec
```ruby
RSpec.describe "User Registration", type: :system do
  before { driven_by(:rack_test) }

  it "creates account" do
    visit new_registration_path
    fill_in "Email", with: "new@example.com"
    click_button "Sign Up"
    expect(page).to have_content("Welcome")
  end
end
```

## Migration from v1/v2

If you were using the individual agent files (v1), the functionality is now consolidated:

| v1/v2 | v3 (Current) |
|-------|--------------|
| 11 separate agent files | 1 skill + 1 agent |
| ~75KB always loaded | ~8KB, loads on-demand |
| `/rspec-rails:rspec` | `/rspec-rails:write-test` (patterns) |
| `@rspec-rails-agent` | `@rspec-test-writer` (isolated work) |

## Contributing

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/improvement`)
3. Commit your changes (`git commit -m 'Add improvement'`)
4. Push to the branch (`git push origin feature/improvement`)
5. Open a Pull Request

## License

MIT License - see [LICENSE](LICENSE) for details.

## Author

**Avi Flombaum** - [avi.nyc](https://avi.nyc)

---

*Transform your AI assistant into an RSpec expert!*
