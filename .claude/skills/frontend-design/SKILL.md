---
name: frontend-design
description: Implements feature specs written in this project's "Step N" markdown format (sections like Overview, Depends On, Routes, Database Changes, Templates, Files to Change/Create, New Dependencies, Rules for Implementation, Definition of Done) against this Flask app's codebase. Trigger this any time the user pastes or references a spec document in this format, says things like "implement this spec," "build Step N," or "here's the next step," even if they don't say the word "spec." Do NOT use this to draft new specs — only to implement ones the user already wrote. Enforces this project's fixed conventions (raw sqlite3, parameterized queries, Werkzeug password hashing, CSS-variable-only styling, base.html template inheritance, session-based auth guards) on every step regardless of whether the spec repeats them, and verifies the Definition of Done checklist against the actual code before reporting done.
---

# Flask Spec-Step Implementer

Implements a single "Step N" spec against the project's Flask codebase, consistently, without drifting from house conventions or faking completion.

This skill is implementation-only. If the user wants help *writing* a new spec, that's a different task — don't reach for this skill for that.

## Fixed project conventions (always apply — even if the spec doesn't repeat them)

These are constant across every step in this project. Apply them regardless of what an individual spec says, and flag it to the user rather than silently resolving it if a spec ever seems to contradict one of these:

- **No ORMs.** No SQLAlchemy or similar. All database access is raw `sqlite3` through the existing `get_db()` helper.
- **Parameterized queries only.** Never build SQL with string formatting, f-strings, or concatenation — always `?` placeholders with a params tuple.
- **Passwords hashed with Werkzeug.** `generate_password_hash` / `check_password_hash` — never store or compare plaintext.
- **CSS variables only.** No hardcoded hex (or `rgb()`/named) color values anywhere in templates or CSS — always `var(--something)`. Before adding a new visual element, check the existing stylesheet for a variable that already fits; only introduce a new variable if nothing fits, and define it alongside the others rather than inline.
- **No inline styles.** No `style="..."` attributes in templates. Style via CSS classes.
- **All templates extend `base.html`.** `{% extends "base.html" %}` at the top of every template.
- **Session-based auth guard.** Check `session.get("user_id")`. If absent, `return redirect(url_for("login"))`. Don't invent a different auth check.
- **Badges/status indicators use CSS classes**, not inline colors — e.g. `<span class="badge badge-{{ category }}">`.
- **No new dependencies** unless the spec's own "New Dependencies" section explicitly lists one.

## Workflow

### 1. Parse the spec

Read the whole spec before touching code. Pull out:
- **Depends On** — prerequisite steps that must already exist.
- **Routes** — method, path, auth requirement, redirect behavior.
- **Database Changes** — schema changes, or explicitly "none."
- **Templates** — files to create/edit and what sections/data they must contain.
- **Files to Change / Files to Create.**
- **New Dependencies** — should almost always be "none"; treat any listed dependency as a deliberate exception worth double-checking with the user if it seems unrelated to the step.
- **Rules for Implementation** — step-specific rules layered on top of the fixed conventions above.
- **Definition of Done** — the literal checklist and any test command to run.

### 2. Verify dependencies before writing anything

For each item in "Depends On," confirm it's actually present in the codebase (e.g., a schema/table exists, a route/session mechanism from a prior step is in place, `base.html` exists). Use `view` to check, don't assume from the spec text alone.

If a dependency is missing, stop and tell the user what's missing instead of building around the gap or faking a stub for it.

### 3. Inspect current codebase state

Before editing, look at what's actually there: `app.py`, `templates/base.html`, the schema/`get_db()` definition, the stylesheet's existing CSS variables, and any relevant existing tests. Match the existing style (helper names, block names, variable names) rather than introducing parallel conventions.

### 4. Implement in this order

1. **Database changes**, if any — raw `sqlite3`, parameterized queries, via `get_db()`. If the spec says "no database changes" or "no queries in this step," do not add any — including "just in case" scaffolding for a future step.
2. **Files to Change** (typically `app.py`) — add/modify view functions per the Routes section. Apply the auth guard pattern exactly. If the spec says data should be hardcoded Python dicts/lists rather than queried, keep it that way even if wiring to real data would be easy — that's usually deliberate sequencing (UI-first, DB-wired later).
3. **Files to Create** (typically templates) — extend `base.html`, follow the CSS/class rules above, include everything the spec's section-by-section content list calls for (don't drop required fields to save time).

### 5. Reconcile spec-specific rules against the fixed conventions

Re-read the spec's own "Rules for Implementation" section against the fixed list above. If they agree — normal case — just proceed. If a spec-specific rule looks like it conflicts with a fixed convention, don't silently pick one: flag the conflict to the user before writing code that depends on the resolution.

### 6. Walk the Definition of Done, item by item

Don't eyeball this. For each checkbox, trace through the actual code you wrote (or the actual route logic) to confirm it, e.g.:
- Auth redirect: trace the guard clause, confirm the redirect target.
- HTTP 200 for authenticated case: confirm no exception paths before the render.
- Required page content: confirm each named element (card, stat, table row count, category count, badge) is actually present with the required minimum count, not just plausibly present.
- No hex colors: grep the new/edited template and CSS for `#` color patterns.
- Navbar logged-in state: confirm it's driven by the existing session-check block in `base.html`, not reimplemented.

If the spec's Definition of Done includes a test command, run it for real and report the actual pass/fail output — never state tests pass without having run them.

### 7. Report back

Give the user the Definition of Done checklist with an honest pass/fail per item (based on step 6), plus the real test output. Call out anything you flagged in step 2 or step 5 up front rather than burying it.

## Common pitfalls specific to this spec format

- Treating "hardcoded" sections as a suggestion and quietly wiring up real queries anyway — don't. UI-first steps are hardcoded on purpose.
- Skipping the dependency check and discovering mid-implementation that a prior step's route/table doesn't actually exist yet.
- Adding a "temporary" hex color or inline style "just for this one badge" — there are no exceptions to the CSS-variable/no-inline-style rules.
- Marking a Definition of Done item done because it's *likely* true rather than tracing it in the actual code or running the actual test.