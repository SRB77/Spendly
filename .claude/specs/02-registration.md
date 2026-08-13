# Spec: Registration

## Overview
This step implements user registration for Spendly. A new visitor can submit their name, email, and password via a form. The server validates the input, hashes the password, and inserts a new row into the `users` table. On success the user is redirected to the login page; on failure the form is re-rendered with a descriptive error message. This step is the first point at which real user data enters the system.

## Depends on
- Step 1 — Database setup (`get_db()`, `init_db()`, `users` table must exist)

## Routes
- `GET /register` — Render the registration form — public (already exists as stub, promote to full handler)
- `POST /register` — Process the registration form — public

## Database changes
No new tables or columns. The `users` table created in Step 1 is sufficient:
- `name TEXT NOT NULL`
- `email TEXT NOT NULL UNIQUE`
- `password_hash TEXT NOT NULL`

A new helper function `create_user(name, email, password)` must be added to `database/db.py`.

## Templates
- **Modify:** `templates/register.html` — add a `<form method="POST">` with fields for name, email, password, and confirm-password; display flash error messages; link to login page with `url_for()`

## Files to change
- `app.py` — convert `GET /register` stub to a full two-method route; add `POST /register` handler; import `flash`, `redirect`, `request`, `url_for`, `session` from Flask
- `database/db.py` — add `create_user(name, email, password)` helper
- `templates/register.html` — add form markup and error display

## Files to create
No new files.

## New dependencies
No new dependencies. `werkzeug.security.generate_password_hash` is already available.

## Rules for implementation
- No SQLAlchemy or ORMs — raw `sqlite3` only
- Parameterised queries only — never f-strings in SQL
- Passwords hashed with `werkzeug.security.generate_password_hash` using `pbkdf2:sha256`
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- Use `flask.flash()` for user-facing error messages — render them in the template
- Duplicate email must be caught by the UNIQUE constraint; catch `sqlite3.IntegrityError` and flash a friendly message
- Validate server-side: name non-empty, valid email format (basic check), password ≥ 8 chars, passwords match
- After successful registration redirect to `url_for('login')` — do not auto-login
- `create_user()` lives in `database/db.py`, not inline in the route
- `app.secret_key` must be set for `flash()` to work; use a hard-coded dev key for now (e.g. `"dev-secret-key"`)

## Definition of done
- [ ] `GET /register` renders the form page with no errors
- [ ] Submitting valid data creates a new row in `users` with a hashed password and redirects to `/login`
- [ ] Submitting a duplicate email re-renders the form with an error message — no new row inserted
- [ ] Submitting mismatched passwords re-renders the form with an error — no DB write
- [ ] Submitting a password shorter than 8 characters re-renders the form with an error
- [ ] Submitting an empty name re-renders the form with an error
- [ ] The password is stored as a hash, never plaintext
- [ ] All internal links use `url_for()`, no hardcoded URLs
- [ ] App starts and all existing routes still work after the change
