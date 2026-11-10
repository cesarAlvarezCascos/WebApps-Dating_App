# Dating App - Flask + MariaDB

A full-stack dating web application: create a profile, discover people who fit your preferences, like or block them, and, once there's a mutual match, book a dinner date at a restaurant with live table availability.

> Built as a team project for the **Web Applications** course at UC3M.

**Stack:** Python · Flask · SQLAlchemy · MariaDB/MySQL · Jinja2 · HTML/CSS · JavaScript (AJAX + jQuery)

---

## ✨ Features

- **Two-step sign-up** (credentials → profile with photo, bio and matching preferences), hashed passwords and session login.
- **Discovery homepage** that only shows people who match *your* preferences **and** whose preferences match *you*. A "Show all" toggle drops the filter.
- **Like / block** from any profile, updated via AJAX with live button states and on-screen notifications.
- **Matches** appear when two users like each other. Likes and blocks are mutually exclusive, and blocking someone removes their like.
- **Date scheduling** (matches only): pick a day on a calendar, an AJAX call returns the restaurants with free slots, and submitting creates a proposal.
- **Date management**: sent/received proposals with dynamic status filters (`proposed`, `accepted`, `rejected`, `ignored`, `reschedule`).
- **Photo gallery**: upload with preview, full-size viewer (`Esc` to close), delete in edit mode.
- **Privacy-minded rules**: blocked users can't see you or open your profile, and an ignored proposal still looks "proposed" to its sender.

## 🧱 How it works

```
dating/
├── __init__.py      # App factory: config, SQLAlchemy, Flask-Login, blueprints
├── model.py         # ORM models (see below)
├── auth.py          # Blueprint: signup, signup2, login, logout, change_password
├── main.py          # Blueprint: homepage, profiles, like/block, dates, photos
├── templates/       # Jinja2 views (auth/, main/)
└── static/          # CSS, JS (AJAX behaviour), photo uploads in img/
```

- **Backend** — Flask with two blueprints (`auth`, `main`). Flask-Login handles sessions; Werkzeug hashes passwords.
- **Data layer** — SQLAlchemy 2.0 (typed `Mapped` models) over PyMySQL, targeting MariaDB.
- **Frontend** — server-rendered Jinja2 templates; vanilla JS + jQuery for the interactive bits (like/block, photo upload/delete, date availability, proposal filters, dropdowns).

### Data model

| Model | Purpose |
|---|---|
| `User`, `Profile`, `Photo` | Account, public profile and gallery (`is_profile_photo` flags the main picture) |
| `MatchingPreferences` | Genders of interest (JSON) and min/max age difference |
| `LikingAssociation`, `BlockingAssociation` | Self-referential many-to-many between users |
| `DateProposal` | Proposer, recipient, date, status and messages |
| `Restaurant`, `Availability`, `DateInRestaurant` | Multiple restaurants, free slots per night, and the venue of each proposal |

## 🚀 Getting started

### Requirements

- Python 3.11+
- A **MariaDB / MySQL** database you can connect to (see [What you need to provide](#-what-you-need-to-provide))

### Run it

```bash
git clone https://github.com/cesarAlvarezCascos/WebApps-Dating_App.git
cd WebApps-Dating_App
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

Point the app at your database (see below), then:

```bash
flask --app dating run --debug
```

Open <http://127.0.0.1:5000>, sign up, complete your profile and log in.

## 🔧 What you need to provide

The original project ran against a database hosted by the university, so **a fresh clone won't work out of the box**. You need to adapt a few things:

1. **A database and connection string.** In [`dating/__init__.py`](dating/__init__.py), `SQLALCHEMY_DATABASE_URI` and `SECRET_KEY` are hardcoded. Replace them with your own, ideally read from environment variables:
   ```python
   app.config["SQLALCHEMY_DATABASE_URI"] = os.environ["DATABASE_URL"]  # mysql+pymysql://user:pass@host/dbname
   app.config["SECRET_KEY"] = os.environ["SECRET_KEY"]
   ```
   For a local setup, MariaDB or MySQL in Docker is enough:
   ```bash
   docker run -d --name dating-db -e MARIADB_ROOT_PASSWORD=secret -e MARIADB_DATABASE=dating -p 3306:3306 mariadb:11
   ```
   Then use `mysql+pymysql://root:secret@127.0.0.1:3306/dating`.

2. **Create the tables.** The repo has no migrations or schema script, so create them from the models once:
   ```bash
   flask --app dating shell
   >>> from dating import db
   >>> db.create_all()
   ```

3. **Seed restaurants and availability.** Date scheduling needs at least one `Restaurant` and some `Availability` rows (date + `slots`), otherwise the calendar shows nothing. Insert a few from the same shell session.

4. **Sample users (optional).** Profile photos are stored as `static/img/photo-<id>.<ext>`, matching the `Photo` row ids. The bundled images belong to the original database, so with a new one you'll just create your own users and uploads.

> ⚠️ **Security note:** never commit real credentials or a production `SECRET_KEY`. Rotate any that were previously published, and consider adding a `.gitignore` for `__pycache__/` and your `.env`.

## 🗺️ Ideas for next steps

- Move configuration to environment variables / `.env` and add a `schema.sql` + seed script.
- Add Flask-Migrate for schema evolution and automated tests for the match/block rules.
- Real-time chat between matches.
