# Spec: Login and Logout

## Overview

Spendly now has a working data layer (Step 1) and account creation (Step 2), but there is still no way to sign in — `GET /login` renders a form that posts to `/login` with no handler, and `/logout` is a placeholder that just returns text. This step wires up real authentication: verifying email/password against the `users` table, starting a Flask session on success, and clearing that session on logout. This is the third step in the roadmap and is what future steps (Profile in Step 4, expense management in Steps 7–9) will depend on to know who the current user is.

## Depends On

- Step 1 — Database Setup (`database/db.py`: `get_db()`, `users` table schema)
- Step 2 — Registration (`users` rows with `password_hash` values to authenticate against)

## Routes

- `POST /login` — validate email/password against the `users` table, and on success store the user's id in the session and redirect to `/profile`; on failure re-render `login.html` with an error — access level: public
- `GET /login` — already exists (`app.py`, renders `login.html`); unchanged — access level: public
- `GET /logout` — clear the session and redirect to `/login` — access level: logged-in (replaces the current placeholder that returns `"Logout — coming in Step 3"`)

## Database Changes

No database changes. The existing `users` table (`database/db.py`) already has everything login needs:

```sql
id INTEGER PRIMARY KEY AUTOINCREMENT
name TEXT NOT NULL
email TEXT UNIQUE NOT NULL
password_hash TEXT NOT NULL
created_at TEXT DEFAULT (datetime('now'))
```

## Templates

Create: none

Modify:
- `templates/login.html` — no structural changes needed; the `{% if error %}` block and `form-input`/`btn-submit` classes already exist and already point `POST` to `/login`. Only re-populate the `email` field's `value` on a failed submit, matching the pattern used in `register.html`.
- `templates/base.html` — conditionally render the nav links: when `session.get('user_id')` is set, show the signed-in user's name and a "Log out" link (`{{ url_for('logout') }}`) instead of the current "Sign in" / "Get started" links.

## Files to Change

- `app.py` —
  - Set `app.secret_key` (required for Flask sessions to work; no new package, this is built into Flask).
  - Extend `/login` to accept `POST` (`methods=["GET", "POST"]`): on `POST`, look up the user by email, verify the password with `werkzeug.security.check_password_hash`, and on success set `session["user_id"]` (and `session["user_name"]`) and redirect to `/profile`; on failure re-render `login.html` with a generic error (e.g. "Invalid email or password") and HTTP status 401 — do not reveal whether the email or the password was wrong.
  - Replace the placeholder `/logout` route: clear the session (`session.clear()`) and redirect to `/login`.
- `templates/login.html` — re-populate the `email` field value on a failed login attempt.
- `templates/base.html` — show session-aware nav links (signed-in state vs. signed-out state).

## Files to Create

None.

## New Dependencies

No new dependencies. Uses Flask's built-in `session` (cookie-based) and the already-installed `werkzeug.security.check_password_hash`.

## Rules for Implementation

- No SQLAlchemy or ORMs.
- Parameterized queries only — never build SQL with string formatting or f-strings.
- Passwords hashed with `werkzeug.security`; verify with `check_password_hash(password_hash, password)` — never compare plaintext passwords.
- Use CSS variables (`var(--...)`) for any new styling — never hardcode hex values. (This step needs no new CSS; existing auth and nav classes are reused.)
- All templates extend `base.html`.
- Look up the user by email first (`SELECT id, name, password_hash FROM users WHERE email = ?`); if no row is found, or `check_password_hash` fails, show the same generic "Invalid email or password" error either way — don't leak which field was wrong.
- On any login failure, re-render `login.html` with `error` set and HTTP status 401; on success, issue a redirect (302) to `/profile` and store only the user's `id` (and optionally `name`) in the session — never store the password or password hash in the session.
- `/logout` must work even if no session exists (i.e. `session.clear()` is safe to call unconditionally) and must always redirect to `/login`.

## Definition of Done

- [ ] Submitting `/login` with the seeded demo user's credentials (`demo@spendly.com` / `demo123`) redirects to `/profile` and sets a session cookie.
- [ ] Submitting `/login` with a correct email but wrong password shows "Invalid email or password" on `login.html` and does not set a session.
- [ ] Submitting `/login` with an email that doesn't exist shows the same "Invalid email or password" error (no distinct message) and does not set a session.
- [ ] After a failed login, the previously entered email is still visible in the form field.
- [ ] Visiting `/logout` while logged in clears the session and redirects to `/login`.
- [ ] Visiting `/logout` while not logged in does not error and still redirects to `/login`.
- [ ] After logging in, the navbar (visible on every page via `base.html`) shows a "Log out" link instead of "Sign in" / "Get started".
- [ ] After logging out, the navbar reverts to showing "Sign in" / "Get started".
- [ ] All SQL statements involved use `?` placeholders — no string-formatted SQL.
