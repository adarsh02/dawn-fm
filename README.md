# Dawn FM

A full-stack music streaming application built with **Vue 3** and **Flask**. Listeners can explore albums, play music, and build playlists; creators manage their catalog; administrators get a view of platform activity.

The backend stores accounts and music metadata in SQLite. Celery handles scheduled email reminders and monthly creator reports, with Redis as the task broker and result backend.

## Features

| Role | Capabilities |
| --- | --- |
| Listener | Sign up and log in, browse and search albums and songs, play tracks, view lyrics, manage playlists, rate songs, and flag tracks. |
| Creator | Register as an artist, manage albums and song metadata, and receive monthly PDF reports with listening and rating statistics. |
| Administrator | View platform statistics, inspect popular and highly rated tracks, add artists, and manage albums and songs. |

## Tech stack

| Layer | Technologies |
| --- | --- |
| Frontend | Vue 3, Vue Router 4, Vuex 4, Vue CLI 5, Bootstrap, Font Awesome |
| API | Flask, Flask-RESTful, Flask-Security-Too, PyJWT |
| Data | SQLite, Flask-SQLAlchemy, SQLAlchemy |
| Background jobs | Celery, Redis |
| Reports and email | Matplotlib, ReportLab, SMTP / MailHog |

## Repository layout

```text
music_streaming_app/
|-- backend/
|   |-- main.py                 # Flask entry point, routes, and task schedules
|   |-- celeryconfig.py         # Redis connections and scheduler timezone
|   |-- requirements.txt        # Pinned Python dependencies
|   |-- instance/               # Local SQLite database (ignored by Git)
|   `-- website/
|       |-- __init__.py         # Application factory and extension setup
|       |-- auth.py             # Authentication, artist creation, and statistics
|       |-- albums.py           # Album and song management
|       |-- tracks.py           # Browsing, search, and playlist tracks
|       |-- user.py             # Playlists, ratings, flags, and creator registration
|       |-- models.py           # Database models and relationships
|       |-- worker.py           # Celery integration with Flask
|       |-- tasks.py            # Reminders and PDF report generation
|       `-- mail_service.py     # SMTP email and attachments
|-- frontend/
|   |-- public/                 # HTML entry point and favicon
|   |-- src/
|   |   |-- assets/             # Images and player icons
|   |   |-- components/         # Shared music player
|   |   |-- views/              # Routed listener, creator, and admin pages
|   |   |-- router/routes.js    # Routes and navigation guards
|   |   |-- store.js            # Vuex user and playlist state
|   |   |-- App.vue             # Root component
|   |   `-- main.js             # Frontend entry point
|   `-- package.json           # Frontend dependencies and scripts
|-- .editorconfig
|-- .gitattributes
|-- .gitignore
`-- README.md
```

## Run locally

### Prerequisites

- Python 3.10 or 3.11 for the pinned backend dependencies.
- Node.js and npm for the Vue CLI frontend.
- Redis and a local SMTP server such as MailHog if you want to run background jobs.

On Windows, run the Flask API and frontend with PowerShell. Use WSL or Linux for the Celery worker and scheduler.

### 1. Clone the repository

```sh
git clone https://github.com/adarsh02/music_streaming_app.git
cd music_streaming_app
```

### 2. Set up the backend

From the repository root:

```sh
python -m venv .venv
```

Activate the environment:

```powershell
# Windows PowerShell
.\.venv\Scripts\Activate.ps1
```

```sh
# macOS / Linux / WSL
source .venv/bin/activate
```

Install dependencies and open a Flask shell:

```sh
pip install -r backend/requirements.txt
cd backend
flask --app main shell
```

The app creates the database tables on startup. For a fresh database, create the roles expected by signup and creator registration. Run this in the Flask shell:

```python
from website.models import db, Role

missing_roles = [Role(name=name) for name in ("user", "creator", "admin") if not Role.query.filter_by(name=name).first()]
db.session.add_all(missing_roles)
db.session.commit()
exit()
```

Start the API from `backend/`:

```sh
python main.py
```

The API listens at `http://127.0.0.1:5000`. The database lives at `backend/instance/database_1.db` and is kept out of version control.

### 3. Start the frontend

In another terminal, from the repository root:

```sh
cd frontend
npm ci
npm run serve
```

Open the local URL printed by Vue CLI, normally `http://localhost:8080`. Sign up for a listener account, then log in. Creator registration is available from the listener interface. Administrator accounts must be provisioned separately in the local database; no default admin credentials are supplied.

### 4. Optional: run reminders and creator reports

Start Redis on port `6379` and MailHog (or another local SMTP capture server) on port `1025`. With the backend environment active, run each command in its own terminal from `backend/`:

```sh
celery -A main.celery_app worker --loglevel=info
```

```sh
celery -A main.celery_app beat --loglevel=info
```

The scheduler uses the `Asia/Kolkata` timezone. Daily reminders run at **02:20**, and monthly creator reports run at **02:20 on the 15th**. Reports include top-rated songs, top-streamed songs, streams by genre, and top-streamed albums. Generated PDFs are ignored by Git.

## Local configuration

| Setting | Location | Default |
| --- | --- | --- |
| API URL used by the frontend | `frontend/src/views/`, `components/Player.vue`, and `store.js` | `http://127.0.0.1:5000` |
| Database and Flask settings | `backend/website/__init__.py` | Local SQLite database; development keys |
| Task broker | `backend/celeryconfig.py` | `redis://localhost:6379/1` |
| Task results | `backend/celeryconfig.py` | `redis://localhost:6379/2` |
| SMTP host, port, and sender | `backend/website/mail_service.py` | `localhost:1025`, `noreply@dawnfm.local` |

## Development commands

Run these from `frontend/`:

```sh
npm run serve       # Development server
npm run build       # Production bundle in dist/
npm run lint        # ESLint checks with automatic fixes
npm run lint -- --no-fix  # Check without changing files
```

To check Python syntax from the repository root:

```sh
python -m compileall backend
```

## Project notes

- This is a local development project. Authentication uses development keys, administrator login uses a plaintext password comparison, and some API endpoints lack authorization checks. Review these before a public deployment.
- The repository includes source code, not a music library or a populated database. Playback needs browser-accessible audio URLs in `Track.file_path`. The current add-song endpoint reuses an existing track's audio path, so a fresh database needs an initial catalog entry before that flow can work. There is no audio upload pipeline.
- The frontend currently calls a fixed local API URL. A hosted setup needs that URL and the backend configuration adjusted.
- Background email expects a local SMTP server without authentication. PDF reports are written to the worker's current directory.
- Backend dependencies are pinned to the original project versions. Dependency upgrades and broader application fixes are separate from this repository cleanup.
