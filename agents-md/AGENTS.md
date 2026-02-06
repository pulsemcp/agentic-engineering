# AGENTS.md — Root Template

A template for the root-level `AGENTS.md` (or `CLAUDE.md`) in a project. Copy this file to your repo root and fill in the sections that apply. Delete sections that don't.

---

<!-- ============================================================
     STRUCTURE GUIDE

     This template is organized from most-critical to least-critical.
     The first thing an agent reads should be the thing most likely
     to prevent wasted work.

     Sections:
     1. Common pitfalls        — things that will confuse the agent immediately
     2. Project structure       — where things live
     3. Development workflow    — how to make changes safely
     4. Commands                — how to build, test, lint, deploy
     5. Architecture            — how the code is organized
     6. Testing                 — testing philosophy and practices
     7. CI/CD                   — how CI works and what to expect
     8. Code standards          — style, naming, comments
     9. Working with the user   — communication norms
     10. Learnings              — accumulated non-obvious knowledge

     For directory-scoped AGENTS.md files (e.g. test/, api/),
     see agents-md/test/AGENTS.md for an example.
     ============================================================ -->

## Common Pitfalls

<!-- Put the things that WILL trip the agent up front and center.
     These are the "I just wasted 10 minutes because I didn't know X"
     items. Be specific and direct. Examples:

     - "This is a monorepo. Always cd into web-app/ before running
       Rails commands. Running from root will fail with confusing errors."
     - "Never create app/ or config/ directories in the repo root —
       all application code lives in src/."
     - "Always use full absolute paths with Edit/Write tools.
       Relative paths will create files in the wrong location."
-->

## Project Structure

<!-- A brief map of where things live. Agents need this to avoid
     creating files in the wrong place or missing existing code.
     For monorepos, list each project with a pointer to its own
     AGENTS.md. -->

```
repo-root/
├── src/              — application source code
├── test/             — test suite (see test/AGENTS.md)
├── infra/            — CDK / IaC definitions
├── scripts/          — utility scripts
├── .github/          — CI workflows
└── docs/             — documentation
```

## Development Workflow

### Branching

<!-- How branches work in this repo. -->

- Create feature branches from `main`
- <!-- Branch naming convention if any -->
- Do not commit unless explicitly asked — show the user your work first

### Pre-commit hooks

<!-- What happens on commit, and what to do when hooks fail.
     The key message: fix the issue, never --no-verify. -->

- If hooks fail, fix the underlying issue — do not use `--no-verify`
- <!-- e.g. "Lint failures: run `npm run lint:fix`" -->
- <!-- e.g. "Type errors: run `tsc --noEmit` and fix" -->

### Pull requests

- After pushing, perform a self-review of the diff before asking for human review
- <!-- PR template or description conventions -->

## Commands

<!-- Only include commands that an agent would actually run.
     Group by activity. Keep it short — agents can read READMEs
     for full docs, this is the quick reference. -->

### Setup
```bash
# e.g. npm install, bundle install, pip install -r requirements.txt
```

### Build
```bash
# e.g. npm run build, cargo build
```

### Test
```bash
# Run targeted tests (PREFERRED locally)
# e.g. npm test -- path/to/test.ts
# e.g. bin/test test/models/specific_test.rb

# DO NOT run the full test suite locally — delegate to CI
```

### Lint
```bash
# e.g. npm run lint:fix
# e.g. bundle exec standardrb --fix
```

### Run locally
```bash
# e.g. npm run dev, bin/dev, cargo run
```

## Architecture

<!-- High-level architecture. Enough for an agent to understand where
     to make changes, not a full design doc. -->

### Tech stack
- **Language**: <!-- e.g. TypeScript, Ruby, Rust -->
- **Framework**: <!-- e.g. Next.js, Rails, Axum -->
- **Database**: <!-- e.g. PostgreSQL, DynamoDB -->
- **Infrastructure**: <!-- e.g. AWS ECS via CDK, Vercel, Fly.io -->

### Key patterns
<!-- The 3-5 patterns that govern how code is organized in this repo.
     An agent that understands these will write code that fits in. -->

- <!-- e.g. "Service objects for external API calls" -->
- <!-- e.g. "All background work goes through a job queue (GoodJob / BullMQ)" -->
- <!-- e.g. "Feature flags via config, not code branches" -->

### Important files
<!-- Files that an agent will need surprisingly often. -->

- <!-- e.g. "src/config.ts — all runtime configuration" -->
- <!-- e.g. "infra/lib/stack.ts — CDK stack definition" -->

## Testing

<!-- Testing philosophy and practical guidance. If you have a
     separate testing philosophy doc, point to it here and keep
     this section as a summary. -->

### Philosophy
- <!-- e.g. "Test behavior, not implementation" -->
- <!-- e.g. "Never mock internal code — only mock at external service boundaries" -->
- <!-- e.g. "Prefer integration tests over unit tests for controllers" -->

### What to run locally
- Run only the tests directly related to files you changed
- Delegate the full suite to CI after pushing

### What to do when tests fail in CI
- Read the failure output carefully before making changes
- <!-- e.g. "For system test failures, check screenshots first — they show what actually happened" -->
- Fix the issue and push — do not skip or disable tests

## CI/CD

<!-- Set expectations about CI so the agent doesn't get confused
     by wait times or unexpected behavior. -->

- CI runs on every push to a PR
- <!-- e.g. "Expected CI time: ~6 minutes" -->
- After pushing and completing self-review, use the `wait-for-ci` skill (or equivalent) to block until CI completes
- Never present work as done without confirming CI is green
- If CI is not triggering, check for merge conflicts before trying anything else

## Code Standards

### Style
- <!-- e.g. "Follow existing patterns in the codebase" -->
- <!-- e.g. "Use the project's configured linter/formatter — don't override rules" -->

### Comments
- Do not use temporal language in comments ("now handles", "was refactored to", "currently")
- Write comments as if describing the permanent state of the code
- Only add comments where the logic isn't self-evident

### Backwards compatibility

<!-- Pick one of these stances (or write your own) and delete the other: -->

<!-- STRICT (no compat shims): -->
- Do not add backwards-compatibility shims during refactors — update all code to the new approach
- Exception: public API contracts. If a change would break external consumers, flag it to the user explicitly and document the decision in the PR description.

<!-- GRADUAL (deprecation cycle): -->
<!-- - Deprecate old interfaces with warnings before removing them -->
<!-- - Maintain backwards compatibility for N releases -->

## Working with the User

<!-- Norms for how the agent should communicate. These are easy to
     overlook but high-impact when they're wrong. -->

- When handing back control, be clear about what the next step is and what you need from the user
- <!-- e.g. "If the next step is to review a PR, include the PR link" -->
- <!-- e.g. "If you hit a decision point with multiple valid approaches, ask — don't guess" -->
- Do not over-explain or pad responses — be direct

## Learnings

<!-- A living section for accumulated non-obvious knowledge.
     This is NOT a changelog. Only add entries that meet ALL criteria:

     1. Non-obvious — would take significant time to rediscover
     2. Reusable — likely to come up again in future work
     3. Not documented elsewhere — check existing docs first

     Bad entries: "fixed a typo", "ran bundle install", "tests pass now"
     Good entries: "schema.rb must be regenerated after migrations or
                    CI will fail because it builds from schema.rb,
                    not by running migrations"
-->
