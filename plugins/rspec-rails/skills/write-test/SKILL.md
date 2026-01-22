---
name: write-test
description: This skill should be used when writing RSpec tests for Rails applications. It applies when writing model specs, request specs, system specs, job specs, mailer specs, channel specs, or storage specs. Triggers on test writing, spec creation, "write tests", "add specs", or Rails testing questions.
argument-hint: "[model|request|system|job|mailer|channel] ClassName"
---

# RSpec Test Writer

You are now writing RSpec tests for a Rails application. Follow this workflow exactly.

## Workflow

1. **Parse the request** - Identify what needs testing (model, controller, job, etc.)
2. **Find the source file** - Use Glob/Grep to locate the code to test
3. **Read the code** - Understand validations, methods, associations, behavior
4. **Check existing fixtures** - Look in `spec/fixtures/*.yml` for test data
5. **Determine spec type** - Use the decision framework below
6. **Write the spec file** - Follow the patterns in this document
7. **Run with `--fail-fast`** - Execute: `bundle exec rspec <spec_file> --fail-fast`
8. **Fix failures** - Iterate until green

## Decision Framework

```
What am I testing?
├── Data & Business Logic    → Model specs (spec/models/)
├── HTTP & Controllers       → Request specs (spec/requests/)
├── User Interface           → System specs (spec/system/)
├── Background Processing    → ActiveJob specs (spec/jobs/)
├── Email                    → ActionMailer specs (spec/mailers/)
├── File Uploads             → ActiveStorage specs (spec/models/)
├── Real-time Features       → ActionCable specs (spec/channels/)
└── External Services        → Use VCR + isolation patterns
```

## Core Rules

1. **Never edit rails_helper.rb or spec_helper.rb** - assume RSpec is configured
2. **Never add testing gems** - use what's already in Gemfile
3. **Use fixtures** - `users(:admin)`, not factories
4. **Use `--fail-fast`** - stop on first failure for faster feedback
5. **Don't test Rails internals** - associations, built-in validations work
6. **One outcome per example** - focused, clear tests
7. **Modern syntax** - `expect().to`, not `should`

---

## Spec Type Patterns

### Model Specs (`type: :model`)

Location: `spec/models/<model_name>_spec.rb`

Test validations, scopes, and instance/class methods.

```ruby
RSpec.describe User, type: :model do
  it "is valid with required attributes" do
    user = User.new(email: "test@example.com", name: "Test")
    expect(user).to be_valid
  end

  it "requires an email" do
    user = User.new(email: nil)
    expect(user).to be_invalid
    expect(user.errors[:email]).to include("can't be blank")
  end

  describe ".active" do
    it "returns only active users" do
      expect(User.active).to include(users(:active))
      expect(User.active).not_to include(users(:inactive))
    end
  end
end
```

### Request Specs (`type: :request`)

Location: `spec/requests/<resource_name>_spec.rb`

Test HTTP routing, authentication, authorization, responses.

```ruby
RSpec.describe "Recipes", type: :request do
  describe "GET /recipes" do
    it "returns success for authenticated user" do
      get recipes_path(as: users(:alice))
      expect(response).to have_http_status(:ok)
    end

    it "redirects guests to login" do
      get recipes_path
      expect(response).to redirect_to(sign_in_path)
    end
  end

  describe "POST /recipes" do
    it "creates recipe and redirects" do
      user = users(:alice)
      expect {
        post recipes_path(as: user), params: { recipe: { name: "Pie" } }
      }.to change(user.recipes, :count).by(1)
      expect(response).to redirect_to(recipe_path(Recipe.last))
    end
  end
end
```

### System Specs (`type: :system`)

Location: `spec/system/<feature_name>_spec.rb`

Test full user flows through the browser. Use `rack_test` for non-JS, `cuprite` for JS.

```ruby
RSpec.describe "User Registration", type: :system do
  before { driven_by(:rack_test) }

  it "creates account and shows dashboard" do
    visit new_registration_path
    fill_in "Email", with: "new@example.com"
    fill_in "Password", with: "password123"
    click_button "Sign Up"

    expect(page).to have_content("Welcome")
    expect(page).to have_current_path(dashboard_path)
  end
end

# For JavaScript interactions
RSpec.describe "Dynamic Form", type: :system do
  before { driven_by(:cuprite) }

  it "shows validation errors without page reload" do
    visit new_recipe_path(as: users(:alice))
    click_button "Save"
    expect(page).to have_css(".error", text: "Name can't be blank")
  end
end
```

### ActiveJob Specs (`type: :job`)

Location: `spec/jobs/<job_name>_spec.rb`

Test job logic with `perform_now`, queuing with `perform_later`.

```ruby
RSpec.describe ProcessOrderJob, type: :job do
  it "processes the order" do
    order = orders(:pending)
    ProcessOrderJob.perform_now(order)
    expect(order.reload.status).to eq("processed")
  end

  it "enqueues on correct queue" do
    expect {
      ProcessOrderJob.perform_later(orders(:pending))
    }.to have_enqueued_job.on_queue("orders")
  end

  it "sends confirmation email" do
    order = orders(:pending)
    expect {
      ProcessOrderJob.perform_now(order)
    }.to have_enqueued_mail(OrderMailer, :confirmation)
  end
end
```

### ActionMailer Specs (`type: :mailer`)

Location: `spec/mailers/<mailer_name>_spec.rb`

Test email headers, body content, and delivery.

```ruby
RSpec.describe UserMailer, type: :mailer do
  describe "#welcome" do
    let(:user) { users(:alice) }
    let(:mail) { UserMailer.welcome(user) }

    it "renders headers" do
      expect(mail.subject).to eq("Welcome!")
      expect(mail.to).to eq([user.email])
    end

    it "includes user name in body" do
      expect(mail.body.encoded).to include(user.name)
    end
  end
end
```

### ActionCable Specs (`type: :channel`)

Location: `spec/channels/<channel_name>_spec.rb`

Test channel subscriptions and broadcasts.

```ruby
RSpec.describe ChatChannel, type: :channel do
  before { stub_connection current_user: users(:alice) }

  it "subscribes to room stream" do
    subscribe room_id: 42
    expect(subscription).to be_confirmed
    expect(subscription).to have_stream_from("chat_42")
  end

  it "broadcasts messages" do
    subscribe room_id: 42
    expect { perform :speak, message: "Hello" }
      .to have_broadcasted_to("chat_42")
      .with(a_hash_including(text: "Hello"))
  end
end
```

### ActiveStorage Specs

Location: `spec/models/<model_name>_spec.rb` (with attachment tests)

Test file uploads and attachments.

```ruby
RSpec.describe Recipe, type: :model do
  describe "#photo" do
    it "attaches a file" do
      recipe = recipes(:draft)
      file = fixture_file_upload("spec/fixtures/recipe.jpg", "image/jpeg")
      recipe.photo.attach(file)

      expect(recipe.photo).to be_attached
    end

    it "returns placeholder when no photo" do
      recipe = Recipe.new
      expect(recipe.photo_url).to eq("recipe-placeholder.png")
    end
  end
end

# System spec for upload UI
RSpec.describe "Recipe Photos", type: :system do
  before { driven_by(:rack_test) }

  it "uploads a photo" do
    visit new_recipe_path(as: users(:alice))
    fill_in "Name", with: "Apple Pie"
    attach_file "Photo", Rails.root.join("spec/fixtures/recipe.jpg")
    click_button "Create"

    expect(page).to have_css("img.recipe-photo")
  end
end
```

---

## Isolation Patterns

### When to Isolate
- External APIs → VCR cassettes
- Randomness/time → stub to deterministic values
- Rare error paths → stub to trigger them
- Slow collaborators → only if truly needed

### VCR for HTTP
```ruby
it "fetches external data", :vcr do
  result = ExternalApi.fetch(id: 123)
  expect(result).to be_present
end
```

### Stubbing Time
```ruby
include ActiveSupport::Testing::TimeHelpers

it "expires after 24 hours" do
  travel_to 25.hours.from_now do
    expect(token).to be_expired
  end
end
```

### Verifying Doubles
```ruby
let(:api) { instance_double(PaymentGateway) }

before do
  allow(api).to receive(:charge).and_return(success: true)
end
```

---

## Fixtures Best Practices

```yaml
# spec/fixtures/users.yml
alice:
  email: alice@example.com
  name: Alice
  role: user

admin:
  email: admin@example.com
  name: Admin
  role: admin

# With associations
# spec/fixtures/recipes.yml
published:
  name: Apple Pie
  user: alice  # References users(:alice)
  status: published
```

Access in specs: `users(:alice)`, `recipes(:published)`

---

## DRY Patterns

### Shared Examples
```ruby
RSpec.shared_examples "requires authentication" do
  it "redirects to login" do
    expect(response).to redirect_to(sign_in_path)
  end
end

RSpec.describe "Admin::Users", type: :request do
  describe "GET /admin/users" do
    context "as guest" do
      before { get admin_users_path }
      it_behaves_like "requires authentication"
    end
  end
end
```

### Custom Matchers
```ruby
# spec/support/matchers/be_recent.rb
RSpec::Matchers.define :be_recent do
  match do |record|
    record.created_at > 1.hour.ago
  end
end

# Usage
expect(post).to be_recent
```

---

## Quality Checklist

Before finishing, verify:

- [ ] Using correct spec type?
- [ ] One outcome per example?
- [ ] Fixtures for test data (not factories)?
- [ ] Authentication tested at appropriate scope?
- [ ] Happy path AND at least one edge case?
- [ ] No testing of Rails internals?
- [ ] External services isolated with VCR?
- [ ] Example names describe behavior, not implementation?
- [ ] Tests pass with `bundle exec rspec <file> --fail-fast`?
