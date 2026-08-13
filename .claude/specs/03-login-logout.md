# Spec: Login and Logout

## Overview
This step implements session-based authentication for Spendly. A registered user can submit their email and password via a login form; the server verifies the credentials against the hashed password in the database, writes the user's id and name into `flask.session`, and redirects to the dashboard (or a suitable placeholder). A logout route clears the session and redirects to the landing page. This step is the gateway to all authenticated features: every subsequent step will rely on `session['user_id']` being present to identify the current user.

## Depends on
- Step 1 — Database setup (`get_db()`, `users` table must exist)
- Step 2 — Registration (`create_user()`, hashed passwords stored)

## Routes
- `GET /login` — Render the login form — public (already exists as stub, promote to full handler)
- `POST /login` — Process login credentials, set session, redirect on success — public
- `GET /logout` — Clear the session, redirect to landing — logged-in (currently a stub returning a plain string)

## Database changes
No new tables or columns. A new helper `get_user_by_email(email)` must be added to `database/db.py` to look up a user row by email for credential verification.

## Templates
- **Create:** No new templates required for this step.
- **Modify:** `templates/login.html` — add a `<form method="POST">` with fields for email and password; display flash error messages; link to register page with `url_for()`

## Files to change
- `app.py` — convert `GET /login` stub to a full two-method route; add `POST /login` handler with credential check and session write; convert `GET /logout` stub to a full handler that clears the session; import `session` from Flask (already imported `flash`, `redirect`, `render_template`, `request`, `url_for`)
- `database/db.py` — add `get_user_by_email(email)` helper that returns a `sqlite3.Row` or `None`
- `templates/login.html` — add form markup, error display, and link to `/register`

## Files to create
No new files.

## New dependencies
No new dependencies. `werkzeug.security.check_password_hash` is already available via the existing `werkzeug` install.

## Rules for implementation
- No SQLAlchemy or ORMs — raw `sqlite3` only
- Parameterised queries only — never f-strings in SQL
- Password verification with `werkzeug.security.check_password_hash`
- Use CSS variables — never hardcode hex values
- All templates extend `base.html`
- Use `flask.flash()` for user-facing error messages — render them in the template
- On successful login write `session['user_id']` and `session['user_name']` — nothing else
- On failed login show a single generic message ("Invalid email or password") — never reveal which field was wrong
- `get_user_by_email()` lives in `database/db.py`, not inline in the route
- Logout must call `session.clear()` then redirect to `url_for('landing')`
- Do not implement access control / `@login_required` in this step — that belongs to Step 4
- `app.secret_key` is already set; do not change it

## Definition of done
- [ ] `GET /login` renders the login form with no errors
- [ ] Submitting valid credentials sets the session and redirects (to `/profile` stub or landing — whichever exists)
- [ ] Submitting an unknown email re-renders the form with a generic error — no session written
- [ ] Submitting a wrong password re-renders the form with the same generic error — no session written
- [ ] Submitting an empty email or empty password re-renders the form with an error
- [ ] `GET /logout` clears the session and redirects to `/`
- [ ] After logout, `session['user_id']` is no longer present
- [ ] All internal links use `url_for()`, no hardcoded URLs
- [ ] App starts and all existing routes still work after the change
