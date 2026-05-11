# Competitive Programming Trainer

Competitive Programming Trainer is a small full-stack web app for finding practice problems and keeping track of progress. It was built as a CS50x final project.

The idea is simple: pick the topics you want to practice, choose how confident you feel, optionally select a platform, and the app recommends problems that match. Problems can be saved for later or marked as completed, and completed problems are not recommended again.

## Features

- User registration and login
- Problem recommendations based on topic, platform, and confidence level
- Support for Codeforces and CSES
- Waitlist for problems to solve later
- Completed-problem tracking
- Sidebar list of solved problems, 10 at a time
- Direct links back to the original problem pages
- Minimal Notion-inspired UI
- Fixed dashboard layout with independent scrolling for problem lists

## Tech Stack

### Frontend

- Next.js
- React
- TypeScript
- Tailwind CSS

### Backend

- Python
- Flask
- SQLite
- Flask-CORS
- itsdangerous token signing
- Werkzeug password hashing

## Project Structure

```text
cp-trainer/
├── backend/
│   ├── app.py
│   ├── database.db
│   ├── routes/
│   │   ├── auth.py
│   │   └── problems.py
│   └── utils/
│       └── db.py
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   ├── components/
│   │   └── lib/
│   ├── package.json
│   └── next.config.ts
└── README.md
```

## How It Works

The backend fetches problem data dynamically from supported platforms:

- Codeforces problems are fetched from the Codeforces problemset API.
- CSES problems are fetched from the public CSES problemset page.

The app maps platform-specific metadata into a smaller shared format:

```text
platform
external_id
name
topic
difficulty
link
```

Recommendations are filtered by selected topics and selected platforms. If no topic is selected, all topics are allowed. If no platform is selected, all supported platforms are allowed.

The confidence setting maps roughly to difficulty:

- Low → easier problems
- Medium → medium problems
- High → harder problems

Saved progress is stored locally in SQLite. The `progress` table keeps waitlisted and completed problems per user.

## Running Locally

You need two terminals: one for the backend and one for the frontend.

### 1. Start the backend

From the project root:

```bash
cd backend
.venv/bin/python app.py
```

The backend runs on:

```text
http://127.0.0.1:5000
```

If you do not already have the backend virtual environment, create one and install the needed packages:

```bash
cd backend
python3 -m venv .venv
.venv/bin/pip install flask flask-cors itsdangerous werkzeug
.venv/bin/python app.py
```

### 2. Start the frontend

In a second terminal:

```bash
cd frontend
npm install
npm run dev
```

The frontend usually runs on:

```text
http://localhost:3000
```

The frontend automatically talks to the backend at port `5000`.

## Useful Commands

### Frontend

```bash
cd frontend
npm run dev
npm run build
npm run lint
```

### Backend

```bash
cd backend
.venv/bin/python app.py
```

Quick syntax check:

```bash
python3 -m py_compile backend/app.py backend/routes/auth.py backend/routes/problems.py backend/utils/db.py
```

## API Overview

### Auth

```text
POST /auth/register
POST /auth/login
```

Both endpoints return a signed token. The frontend stores it in `localStorage` and sends it as a bearer token.

### Dashboard

```text
GET /problems
```

Returns the current user, waitlisted problems, and completed problems.

### Recommendations

```text
POST /recommendations
```

Example request body:

```json
{
  "topics": ["dp", "graphs"],
  "platforms": ["codeforces"],
  "confidence": "medium"
}
```

If `topics` or `platforms` is empty, the backend treats that as “use all available options.”

### Progress

```text
POST /progress
```

Used to waitlist a problem or mark it as completed.

## Database

The active database file is:

```text
backend/database.db
```

It contains:

- users
- progress

Do not delete `backend/database.db` unless you intentionally want to reset all local users and saved progress.

There may also be a root-level `database.db`. That file is not used by the app.

## Notes and Limitations

- Codeforces and CSES are the only supported platforms right now.
- CSES does not provide rich tags like Codeforces, so topic and difficulty are inferred from problem sections.
- Recommendations depend on external platform availability. If a provider is down or blocked, that platform may temporarily return no problems.
- The app uses simple signed tokens, not a full production authentication system.
- SQLite is enough for this project, but a deployed multi-user version would probably use a hosted database.

## AI and Assistance Disclosure

Some parts of the project include citation comments where outside assistance was used for non-trivial logic, especially:

- topic mapping
- CSES HTML parsing
- recommendation ranking and platform balancing
- fixed-layout scrolling behavior
- provider caching

Those comments are kept close to the code they describe so the source of help is easy to see.

## Why I Built This

Competitive programming practice can get messy quickly. It is easy to solve random problems without a plan, forget what was already completed, or keep seeing the same problem suggestions again and again.

This app is meant to make practice feel a little more organized: choose what you want to work on, get a short list, save what matters, and keep going.
