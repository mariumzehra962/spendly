# Spec: Profile Page

## Overview

Spendly now has working registration (Step 2) and login/logout (Step 3), so a user can create an account and start a session, but `/profile` is still the placeholder text `"Profile page — coming in Step 4"` that `app.py` redirects to right after login. This step replaces that placeholder with a real, logged-in-only profile page: it shows the signed-in user's account details (name, email, member-since date) and a quick summary of their expense activity (total number of expenses and total amount spent), using the `expenses` rows already seeded in Step 1. This is the fourth step in the roadmap and gives users a landing page after login while the expense list/dashboard and expense CRUD (Steps 7–9) are built in later steps.

## Depends On

- Step 1 — Database Setup (`database/db.py`: `get_db()`, `users` and `expenses` table schemas)
- Step 3 — Login and Logout (`session["user_id"]` / `session["user_name"]` set on login; nav already reads `session.get('user_id')`)

## Routes

- `GET /profile` — replaces the current placeholder; if no `user_id` in session, redirect to `/login`; otherwise fetch the current user's row from `users` and their expense totals from `expenses`, then render `profile.html` — access level: logged-in

## Database Changes

No database changes. The existing tables (`database/db.py`) already have everything the profile page needs:

```sql
-- users
id INTEGER PRIMARY KEY AUTOINCREMENT
name TEXT NOT NULL
email TEXT UNIQUE NOT NULL
password_hash TEXT NOT NULL
created_at TEXT DEFAULT (datetime('now'))

-- expenses
id INTEGER PRIMARY KEY AUTOINCREMENT
user_id INTEGER NOT NULL REFERENCES users(id)
amount REAL NOT NULL
category TEXT NOT NULL
date TEXT NOT NULL
description TEXT
created_at TEXT DEFAULT (datetime('now'))
```

## Templates

Create:
- `templates/profile.html` — extends `base.html`; shows the user's name, email, and member-since date (formatted from `created_at`), plus a summary card with total number of expenses and total amount spent (from `SUM(amount)` / `COUNT(*)` on `expenses` for that `user_id`).

Modify: none. `templates/base.html` already renders the signed-in nav state (`session.get('user_name')` + "Log out" link) from Step 3 and needs no changes.

## Files to Change

- `app.py` —
  - Replace the placeholder `/profile` route: if `session.get("user_id")` is missing, `redirect(url_for("login"))`; otherwise look up the user by id (`SELECT id, name, email, created_at FROM users WHERE id = ?`), compute expense totals for that user (`SELECT COUNT(*), COALESCE(SUM(amount), 0) FROM expenses WHERE user_id = ?`), and render `profile.html` with the user and totals.

## Files to Create

- `templates/profile.html`

## New Dependencies

No new dependencies.

## Rules for Implementation

- No SQLAlchemy or ORMs.
- Parameterized queries only — never build SQL with string formatting or f-strings.
- Passwords hashed with `werkzeug.security` (not touched by this step, but any password field must never be read back to the template).
- Use CSS variables (`var(--...)`) for any new styling in `static/css/style.css` — never hardcode hex values.
- All templates extend `base.html`.
- `/profile` must check `session.get("user_id")` and redirect to `/login` when absent — never render user data for an unauthenticated request.
- Look up the user by the id stored in the session, not by trusting any client-supplied value.
- Use `COALESCE(SUM(amount), 0)` (or equivalent) so a user with zero expenses gets `0`/`0.00`, not `NULL`, in the template.
- Close every `sqlite3` connection opened via `get_db()` after use.

## Definition of Done

- [ ] Visiting `/profile` while logged out redirects to `/login`.
- [ ] Logging in with the seeded demo user (`demo@spendly.com` / `demo123`) and visiting `/profile` shows the name "Demo User" and email "demo@spendly.com".
- [ ] The profile page shows a member-since date derived from the user's `created_at`.
- [ ] The profile page shows a total expense count of 8 and a total amount matching the sum of the seeded demo expenses.
- [ ] A newly registered user with no expenses sees a total count of 0 and a total amount of 0 (not an error, not `None`/`NULL`).
- [ ] The navbar on `/profile` shows the signed-in user's name and a "Log out" link (unchanged behavior from Step 3).
- [ ] All SQL statements involved use `?` placeholders — no string-formatted SQL.
