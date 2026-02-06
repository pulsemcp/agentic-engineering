# AGENTS.md — Test Directory Example

> **This is not a template to copy and fill in.** It's an example of what a directory-scoped AGENTS.md *could* look like for a `test/` directory. Don't start here — start with an empty file. When you notice agents writing tests in the wrong directory, add the structure table. When they mock internals, add the mocking policy. When they run the full suite and waste 10 minutes, add the "what NOT to run" section. Every line should exist because you watched an agent get it wrong without it.

Guidance for agents working in the `test/` directory. This file is only loaded when an agent is operating on files within this directory, so it contains context that agents working elsewhere don't need.

---

## Required Reading

<!-- Point to any testing philosophy docs before the agent writes
     a single test. This is the most important line in this file. -->

- Read `docs/TESTING_PHILOSOPHY.md` before writing or modifying any tests

## Test Structure

```
test/
├── models/          — unit tests for models and domain logic
├── services/        — unit tests for service objects
├── controllers/     — request-level tests for HTTP endpoints
├── integration/     — multi-step workflow tests
├── system/          — end-to-end browser tests (Capybara, Playwright, etc.)
├── support/         — shared helpers, custom assertions, test utilities
└── fixtures/        — test data (or factories/ if using FactoryBot)
```

### Where does a new test go?

| What you're testing | Directory | Notes |
|---------------------|-----------|-------|
| Model validations, scopes, methods | `models/` | One test file per model |
| Service object logic | `services/` | One test file per service |
| Single HTTP endpoint (status, response body) | `controllers/` | One test file per controller |
| Multi-step flow (create -> update -> verify) | `integration/` | Group by workflow, not by model |
| User-visible behavior in a browser | `system/` | Group by feature, not by page |
| Shared helper or assertion | `support/` | Only if used by 3+ test files |

## Writing Tests

### File naming

- Test files mirror the source path: `app/services/foo_service.rb` -> `test/services/foo_service_test.rb`
- Name the test class to match: `class FooServiceTest < ActiveSupport::TestCase`

### Test data

<!-- Pick one of these and delete the other: -->

**FactoryBot (preferred):**
```ruby
@user = FactoryBot.create(:user, role: "admin")
```

**Fixtures:**
```ruby
@user = users(:admin)
```

- Do not mix both in the same codebase
- Keep test data minimal — only set attributes the test depends on

### Mocking policy

- Never mock internal code (models, services, controllers) — let real code run
- Only mock at external boundaries: third-party HTTP APIs, external SDKs, file system in production paths
- If something is hard to test without mocking internals, that's a design signal — refactor the code

### Assertions

- Assert on behavior and outcomes, not implementation details
- Bad: `assert_equal 3, service.send(:internal_counter)`
- Good: `assert_equal 3, Result.count`

## Running Tests

### Locally — targeted only

```bash
# Run the specific test file for what you changed
bin/test test/models/user_test.rb

# Run a single test method
bin/test test/models/user_test.rb -n test_validates_email_format

# System tests must run headless
HEADLESS=true bin/test test/system/login_test.rb
```

### What NOT to run locally

```bash
# DO NOT run these — delegate to CI:
bin/test                          # full suite
bin/test test/system/             # all system tests
bin/test test/models/             # all model tests
```

Running the full suite locally wastes time and produces false failures from environment differences. Run only what's relevant to your change, push, and let CI handle the rest.

### Timeouts

Always wrap test runs in a timeout to prevent runaway output from crashes or infinite loops:

```bash
timeout 60 bin/test test/models/user_test.rb
```

## System Tests

- Always run with `HEADLESS=true` — agents cannot see a browser window
- System tests are slower and more brittle than other test types — only write them for critical user-facing flows
- Group related assertions into a single system test rather than many small ones (each test pays the cost of browser setup)
- If a system test fails, check the failure screenshot before reading the error message — the screenshot shows what actually happened

## Flaky Tests

If you encounter a test that passes sometimes and fails other times with no code changes:

1. Do not ignore it or retry until it passes
2. File a GitHub Issue with: test name, file path, error message, CI run URL, and any pattern you noticed
3. Common causes: parallel test interference, timing/async issues, test-order dependencies, external service calls without mocks
