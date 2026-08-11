# Spendly

Spendly is a personal expense-tracking web application built with **Flask** and **SQLite**. It helps users log day-to-day expenses, understand spending patterns by category, and stay on top of their monthly budget — without a spreadsheet.

This repository is the server-rendered web app: a Python/Flask backend, Jinja2 templates, and vanilla CSS/JS on the frontend. No frontend framework, no build step, no bundler — just Flask serving HTML.

> **Project status:** Active development. Marketing pages (landing, legal, auth screens) are complete. The expense-tracking core — database layer, session-based auth, and CRUD routes for expenses — is scaffolded but not yet implemented. See [Roadmap](#roadmap) for the current build order.

---

## Table of Contents

- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Application Flow](#application-flow)
- [Project Structure](#project-structure)
- [Getting Started](#getting-started)
- [Routes](#routes)
- [Roadmap](#roadmap)
- [Contributing](#contributing)

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend framework | [Flask 3](https://flask.palletsprojects.com/) (Python) |
| WSGI toolkit | Werkzeug |
| Database | SQLite (via Python's built-in `sqlite3`) |
| Templating | Jinja2 (Flask's default template engine) |
| Styling | Hand-written CSS (`static/css/style.css`) — no framework, uses CSS custom properties for theming |
| Fonts | Google Fonts — DM Serif Display (headings), DM Sans (body) |
| Client-side scripting | Vanilla JavaScript (`static/js/main.js`), no dependencies |
| Testing | pytest + pytest-flask |
| Package management | pip, `requirements.txt` |

Deliberately **no** frontend build tooling (no React/Vue, no Webpack/Vite, no Tailwind). The app is designed to run with nothing beyond a Python virtual environment.

---

## Architecture

Spendly follows a classic **server-rendered MVC-ish structure** on top of Flask:

```
┌─────────────────────────────────────────────────────────────┐
│                          Browser                              │
│   HTML (Jinja2-rendered) + CSS + vanilla JS, no client router │
└───────────────────────────┬────────────────────────────────┘
                             │ HTTP (GET/POST, cookies for session)
┌───────────────────────────▼────────────────────────────────┐
│                        Flask app (app.py)                     │
│   Route handlers → validate input → call database layer      │
│                  → render_template(...) or redirect           │
└───────────────────────────┬────────────────────────────────┘
                             │ SQL (sqlite3)
┌───────────────────────────▼────────────────────────────────┐
│                    database/db.py                             │
│   get_db()  — connection with row_factory + foreign keys      │
│   init_db() — creates tables (idempotent)                    │
│   seed_db() — sample data for local development                │
└───────────────────────────┬────────────────────────────────┘
                             │
┌───────────────────────────▼────────────────────────────────┐
│                  expense_tracker.db (SQLite file)              │
└─────────────────────────────────────────────────────────────┘
```

**Key design decisions:**

- **No API layer / no JSON endpoints (yet).** Every route returns fully rendered HTML. This keeps the app simple and dependency-free, at the cost of full-page reloads on most interactions.
- **Template inheritance.** `templates/base.html` defines the page shell (nav, footer, font/CSS includes) and exposes three overridable blocks — `title`, `head`, and `content` (plus `scripts`) — that every other template extends.
- **One CSS file, theme-driven.** `static/css/style.css` defines the visual system once, at the top, as CSS custom properties (`--ink`, `--accent`, `--paper`, `--radius-md`, etc.), and every component (`.hero`, `.feature-card`, `.auth-card`, `.footer`, ...) is built from those tokens. Page-specific styling (e.g. the landing hero) is added in a page's own `{% block head %}` rather than growing the shared stylesheet indefinitely.
- **SQLite over a heavier RDBMS.** Appropriate for a single-user-per-deployment personal finance tool; `database/db.py` is the single seam where that decision could later be swapped for Postgres/MySQL without touching route logic.

---

## Application Flow

### Current (implemented) flow

```
 Visitor
   │
   ▼
 GET /  ──────────────► Landing page (hero, features, CTA, demo-video modal)
   │
   ├── "Create free account" ──► GET /register ──► fill form ──► POST /register (not yet wired to DB)
   │
   ├── "Sign in" ───────────────► GET /login ────► fill form ──► POST /login    (not yet wired to DB)
   │
   └── Footer ── "Terms and Conditions" ──► GET /terms
              └─ "Privacy Policy"        ──► GET /privacy
```

- The **navbar** (in `base.html`) is present on every page and links back to the landing page, login, and register.
- The **landing page hero** includes a "See how it works" button that opens a modal with an embedded video player (vanilla JS, no libraries — video stops when the modal closes).
- **Auth forms** (`register.html`, `login.html`) render and `POST` to `/register` and `/login` respectively, and support displaying a server-supplied `error` message — but the handlers on the Python side don't yet touch the database or create a session (see Roadmap).

### Planned flow (once the core is built)

```
 Visitor ──► Register/Login ──► Session established
                                     │
                                     ▼
                         Authenticated dashboard
                                     │
                ┌────────────────────┼────────────────────┐
                ▼                    ▼                     ▼
        GET /expenses/add    GET /expenses/<id>/edit   GET /expenses/<id>/delete
        (create expense)     (update expense)          (remove expense)
                                     │
                                     ▼
                          GET /profile · GET /logout
```

The route stubs for this flow already exist in `app.py` (see [Routes](#routes) below) and each is annotated with the build step it belongs to.

---

## Project Structure

```
expense-tracker/
├── app.py                  # Flask app instance + all route definitions
├── requirements.txt        # Python dependencies (Flask, Werkzeug, pytest)
├── database/
│   ├── __init__.py         # Marks database/ as a Python package
│   └── db.py                # get_db() / init_db() / seed_db() — SQLite access layer (to be implemented)
├── templates/
│   ├── base.html            # Shared page shell: <head>, navbar, footer, block outlets
│   ├── landing.html         # Marketing homepage — hero, features, CTA, demo modal
│   ├── register.html        # Sign-up form
│   ├── login.html           # Sign-in form
│   ├── terms.html           # Terms and Conditions
│   └── privacy.html         # Privacy Policy
├── static/
│   ├── css/
│   │   └── style.css        # Entire design system: tokens, layout, components, responsive rules
│   └── js/
│       └── main.js          # Vanilla JS behavior (currently minimal / placeholder)
└── expense_tracker.db       # SQLite database file (gitignored, created at runtime)
```

---

## Getting Started

### Prerequisites

- Python 3.9+
- `pip`

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/SRB77/Spendly.git
cd Spendly

# 2. Create and activate a virtual environment
python3 -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Run the development server
python3 app.py
```

The app starts on **http://127.0.0.1:5001** in debug mode (auto-reloads on file changes).

### Running tests

```bash
pytest
```

---

## Routes

| Method | Route | Status | Description |
|---|---|---|---|
| `GET` | `/` | ✅ Implemented | Landing page |
| `GET` | `/register` | ✅ Implemented | Registration form |
| `GET` | `/login` | ✅ Implemented | Login form |
| `GET` | `/terms` | ✅ Implemented | Terms and Conditions |
| `GET` | `/privacy` | ✅ Implemented | Privacy Policy |
| `GET`/`POST` | `/logout` | 🚧 Placeholder | Ends the user session |
| `GET`/`POST` | `/profile` | 🚧 Placeholder | View/edit account profile |
| `GET`/`POST` | `/expenses/add` | 🚧 Placeholder | Create a new expense |
| `GET`/`POST` | `/expenses/<id>/edit` | 🚧 Placeholder | Update an existing expense |
| `POST` | `/expenses/<id>/delete` | 🚧 Placeholder | Delete an expense |

---

## Roadmap

The backend is being built incrementally, in the following order:

1. **Database setup** — implement `get_db()`, `init_db()`, `seed_db()` in `database/db.py`; define the schema (users, expenses, categories).
2. **User registration** — wire `POST /register` to create a user record with a hashed password.
3. **Login / logout** — session-based authentication.
4. **Profile** — view and edit account details.
5. **Dashboard** — authenticated home view summarizing spend.
6. *(reserved)*
7. **Add expense** — `POST /expenses/add`.
8. **Edit expense** — `POST /expenses/<id>/edit`.
9. **Delete expense** — `POST /expenses/<id>/delete`.

---

## Contributing

This project doesn't use a frontend framework or build step by design — please keep changes to `static/js/main.js` framework-free, and keep shared styling in `static/css/style.css` using the existing CSS custom properties rather than introducing new hardcoded colors/spacing.
