# Spec: Registration

## Overview

Spendly currently has a working data layer (Step 1) and a static `/register` page that renders a form but does not process submissions. This step wires that form up to real account creation: validating input, hashing the password, inserting a new row into `users`, and rejecting duplicate emails. This is the second step in the roadmap and is a prerequisite for Login/Logout (Step 3), which will need real user rows to authenticate against.

## Depends On

- Step 1 — Database Setup (`database/db.py`: `get_db()`, `init_db()`, `seed_db()`, and the `users` table schema)

## Routes

- `POST /register` — process the registration form: validate input, hash password, insert user, redirect to `/login` on success or re-render the form with an error — access level: public
- `GET /register` — already exists (`app.py`, renders `register.html`); unchanged

## Database Changes

None. The existing `users` table (`database/db.py`) already has every column registration needs:

```sql
id INTEGER PRIMARY KEY AUTOINCREMENT
name TEXT NOT NULL
email TEXT UNIQUE NOT NULL
password_hash TEXT NOT NULL
created_at TEXT DEFAULT (datetime('now'))
```

The existing `UNIQUE` constraint on `email` is what backs the duplicate-email check.

## Templates

Create: none

Modify:
- `templates/register.html` — re-populate the `name` and `email` inputs with the previously submitted values (via `value="{{ name or '' }}"` etc.) when validation fails, so the user doesn't have to retype them. The existing `{% if error %}` block already handles displaying the error message; no new markup or CSS classes needed (`auth-error`, `form-group`, `form-input`, `btn-submit` are all already styled in `static/css/style.css`).

## Files to Change

- `app.py` — extend the `/register` route to accept `POST` (`methods=["GET", "POST"]`), and on `POST`: validate input, check for duplicate email, hash the password, insert the user, redirect to `/login` on success, or re-render `register.html` with an `error` (and the submitted `name`/`email`) on failure.
- `templates/register.html` — re-populate `name`/`email` field values on validation error.

## Files to Create

None.

## New Dependencies

No new dependencies.

## Rules for Implementation

- No SQLAlchemy or ORMs.
- Parameterized queries only — never build SQL with string formatting or f-strings.
- Passwords hashed with `werkzeug.security.generate_password_hash` before insertion; never store or compare plaintext passwords.
- Use CSS variables (`var(--...)`) for any new styling — never hardcode hex values. (In practice this step needs no new CSS since all auth classes already exist.)
- All templates extend `base.html`.
- Validate on the server even though the form has HTML5 `required`/`type` attributes: name and email must be non-empty, email must contain `@`, password must be at least 8 characters (matching the field's placeholder text).
- Check for an existing email (`SELECT id FROM users WHERE email = ?`) before inserting, so the error message is friendly ("An account with this email already exists") rather than surfacing a raw `sqlite3.IntegrityError` — but still rely on the `UNIQUE` constraint as the source of truth in case of a race.
- On any validation failure, re-render `register.html` with `error` set and HTTP status 400; on success, issue a redirect (302) to `/login` — do not create a session here, since session/login handling is Step 3's responsibility.

## Definition of Done

- [ ] Submitting the registration form with valid name, email, and an 8+ character password creates a new row in `users` and redirects to `/login`.
- [ ] The stored `password_hash` for that row is a werkzeug hash (e.g. starts with `scrypt:` or `pbkdf2:`), never the plaintext password.
- [ ] Submitting the form again with the same email shows an error on `register.html` and does not insert a second row (`SELECT COUNT(*) FROM users WHERE email = ?` stays at 1).
- [ ] Submitting the form with an empty name, empty email, or a password under 8 characters shows a validation error and inserts no row.
- [ ] After a failed submission, the previously entered name and email are still visible in the form fields (not cleared).
- [ ] `GET /register` still renders the empty form as before (unchanged behavior).
- [ ] All SQL statements involved use `?` placeholders — no string-formatted SQL.
