# Campus Lost & Found Portal

A web application where college students and staff report lost items, report found items, search listings, and recover belongings through a private ownership-claim workflow. Administrators moderate content through Django Admin.

*BCA final-year academic project. Built with Django, PostgreSQL and Bootstrap 5.*

## Problem statement
Lost-and-found on most campuses relies on notice boards, WhatsApp groups and word of mouth. Posts get buried, there is no way to search, and nobody can verify who an item really belongs to.

## Objectives
- Give the campus one searchable place for lost and found reports.
- Let finders hand items back only after reviewing private ownership details.
- Protect personal contact information.
- Give administrators simple moderation tools.

## Features
- Registration and e-mail login (custom user model, strong server-side validation, password reset)
- Profile page with live activity statistics (name and phone editable; email and College ID read-only)
- Report lost / found items with photo upload (JPEG, PNG, WebP, max 5 MB, validated with Pillow)
- Browse with keyword search (title, description, location), type/category/location filters, sorting and pagination (9 per page, filters preserved)
- Item detail pages; only the owner or an administrator can edit or remove a listing (removal is a soft-delete)
- Ownership claims: submit, approve, reject, confirm physical return (all POST-only, permission-checked and wrapped in database transactions with row locks)
- In-app notifications with an unread bell counter (computed from PostgreSQL) and mark-as-read
- Personal dashboard (all numbers come from database queries)
- Django Admin for users, items, claims and notifications, plus a small `/admin-dashboard/` statistics page for staff
- Fully offline front end: Bootstrap, Bootstrap Icons and the Inter font are bundled in `static/vendor/` (no CDN needed)
- Demo-data command and a 91-test automated suite

### Privacy decisions worth knowing
- Phone numbers and email addresses are never shown publicly. People connect only through claims.
- Claim details (explanation, identifying details) are visible only to the claimant, the listing owner and administrators.
- On **found** listings the optional "additional information" field is **private notes** visible only to the owner/admin. On **lost** listings it is public identifying information.
- While one claim is **approved** (waiting for handover), new claims on that item are blocked. When the finder confirms the return, any still-pending claims are closed automatically and those claimants are notified.
- "Successful recoveries" on the profile = items the user claimed, had approved, and that were marked returned. The dashboard "Returned items" card = the user's own listings marked returned.
- The dashboard "Pending claims" card = claims waiting for *your* review (your own pending claims are shown as a small extra line).

## Technology stack
| Layer | Technology |
|---|---|
| Frontend | HTML5, CSS3, vanilla JavaScript, Bootstrap 5.3, Bootstrap Icons |
| Backend | Python 3.10+, Django 5.2 LTS |
| Database | PostgreSQL (via psycopg 3) |
| Other | Pillow (image validation), python-dotenv (configuration) |

## Folder structure
```text
campus_lost_found/
├── manage.py
├── requirements.txt
├── .env.example
├── README.md
├── docs/ACADEMIC_OUTLINE.md
├── campus_lost_found/        settings.py, urls.py, wsgi.py, asgi.py
├── accounts/                 custom User, registration, login, profile, dashboard, admin stats
├── items/                    Item model, forms, browse/search, seed_demo command
├── claims/                   Claim model, forms, services.py (transactional review logic)
├── notifications/            Notification model, context processor, views
├── templates/                base.html, home.html, accounts/, items/, claims/, notifications/, includes/
├── static/                   css/style.css, js/main.js, images/, vendor/ (bootstrap, icons, Inter)
└── media/items/              uploaded photos (created at runtime)
```

## Prerequisites (Windows)
1. **Python 3.10 or newer** from python.org (tick "Add python.exe to PATH").
2. **PostgreSQL 14+** from postgresql.org (remember the password you set for the `postgres` user).
3. **VS Code** (optional) with the Python extension.

## PostgreSQL setup
Open **SQL Shell (psql)** from the Start menu, log in as `postgres`, then run:

```sql
CREATE DATABASE campus_lost_found;
```

(Or in a terminal: `psql -U postgres -c "CREATE DATABASE campus_lost_found;"`.)

## Virtual environment, packages and configuration
Open a terminal in the project folder (in VS Code: Terminal → New Terminal):

```powershell
python -m venv venv
venv\Scripts\activate
python -m pip install --upgrade pip
pip install -r requirements.txt
copy .env.example .env
```

Edit `.env` and set `DJANGO_SECRET_KEY` and `DB_PASSWORD` (your PostgreSQL password). Generate a secret key with:

```powershell
python -c "from django.core.management.utils import get_random_secret_key as g; print(g())"
```

## Migrations, admin user, demo data, run
```powershell
python manage.py check
python manage.py makemigrations --check --dry-run
python manage.py migrate
python manage.py createsuperuser
python manage.py seed_demo
python manage.py runserver
```

- `makemigrations --check --dry-run` only *verifies* that migrations match the models. It does not create anything. The migration files are already included.
- `createsuperuser` asks for email, full name, College ID, phone number (10 digits starting 6-9) and a password. This account can log in to the site and to `/admin/`.
- `seed_demo` creates 8 fictional users, 10 lost items, 10 found items, and claims in every state. It is safe to run again. Demo accounts such as `aarav.mehta@example.com` use the password `Demo@1234`.

Open <http://127.0.0.1:8000/>. Admin: <http://127.0.0.1:8000/admin/>. Staff statistics: <http://127.0.0.1:8000/admin-dashboard/>.

Password-reset e-mails are printed in the terminal running `runserver` (console e-mail backend), so no mail server is needed.

## Running the tests
```powershell
python manage.py test
```
The test runner creates and removes a temporary `test_campus_lost_found` database, so the PostgreSQL user needs permission to create databases (the default `postgres` user has it). Tests cover registration, validation, login/logout, access control, item creation, image validation, search/filters, ownership permissions, claims, review authorization, return confirmation, notifications, privacy and dashboard statistics.

## Troubleshooting
| Problem | Fix |
|---|---|
| `connection refused` / `could not connect` | PostgreSQL service is not running. Open *Services* → start `postgresql-x64-NN`. Check `DB_HOST`/`DB_PORT` in `.env`. |
| `password authentication failed for user "postgres"` | `DB_PASSWORD` in `.env` is wrong. |
| `database "campus_lost_found" does not exist` | Run the `CREATE DATABASE` command above. |
| `permission denied to create database` when testing | Use a PostgreSQL role with `CREATEDB`, e.g. `postgres`. |
| `ImproperlyConfigured: Set DJANGO_SECRET_KEY` | Create `.env` from `.env.example`, or set `DJANGO_DEBUG=True`. |
| `running scripts is disabled on this system` (PowerShell) | Run `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned`, or use Command Prompt (`cmd`). |
| `No module named 'django'` | Activate the venv (`venv\Scripts\activate`) and run `pip install -r requirements.txt`. |
| Images upload but do not show | Keep `DJANGO_DEBUG=True` locally; Django serves `/media/` only in debug mode. |
| CSS/JS not loading | Use `python manage.py runserver` (not another server) with `DEBUG=True`. |
| Login says "Incorrect email or password" for an old test account | Accounts are stored with lower-case emails; use the email you registered with. |

## Known limitations
- Local-development project. For real deployment you would add HTTPS, a production web server, static/media hosting and a real e-mail backend. With `DJANGO_DEBUG=False`, cookies are marked secure-only (needs HTTPS).
- Claim review is manual by design: no automatic verification, chat, OTP or e-mail notifications.
- Phone numbers are collected but only used on the profile page.
- The test suite runs against PostgreSQL; the UI was checked by automated HTTP tests and a live server smoke test (pages, static files, login, upload), but not in a real browser.
