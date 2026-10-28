# Dawn FM

A full-stack music streaming app built with **Vue 3, Flask, and SQLite**. Listeners can browse albums, play songs, create playlists, and rate tracks. Creators manage artists, albums, and songs, while administrators view platform statistics and manage the catalog. Celery and Redis support email reminders and creator reports.

## Code structure

```text
music_streaming_app/
|-- backend/
|   |-- main.py              # Flask API entry point
|   |-- website/             # Routes, database models, email, and report tasks
|   |-- instance/            # SQLite database
|   |-- celeryconfig.py      # Background task configuration
|   `-- requirements.txt     # Python dependencies
|-- frontend/
|   |-- public/              # HTML entry point and favicon
|   |-- src/
|   |   |-- views/           # Listener, creator, and admin pages
|   |   |-- components/      # Shared music player
|   |   |-- assets/          # Images and icons
|   |   |-- router/          # Routes and navigation guards
|   |   |-- store.js         # User and playlist state
|   |   |-- App.vue          # Root component
|   |   `-- main.js          # Vue entry point
|   `-- package.json         # Frontend dependencies and scripts
`-- README.md
```

## Start the app

With Python and Node.js installed, open two terminals and run the commands in the indicated folders:

| Server | Folder | Commands |
| --- | --- | --- |
| Backend | `backend/` | `pip install -r requirements.txt`, then `python main.py` |
| Frontend | `frontend/` | `npm ci`, then `npm run serve` |

Open `http://localhost:8080`; the API runs at `http://127.0.0.1:5000`.
