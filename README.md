# Competitive Programming Trainer
#### Video Demo: https://youtu.be/SQ7oSmnTOso?si=2mdZ378JFN5_CtbR
#### Description:

Competitive Programming Trainer is a full-stack web application for finding competitive programming practice problems and keeping track of progress. I built it for my CS50x final project because I wanted a simple way to organize practice without constantly jumping between platforms, spreadsheets, bookmarks, and random problem lists.

The app lets a user register, log in, choose topics, choose a confidence level, optionally filter by platform, and receive problem recommendations. A user can save a problem to a waitlist or mark it as completed. Completed problems are stored and are not recommended again. There is also a sidebar section that lists solved problems, 10 at a time, with links back to the original problem pages.

The project currently supports Codeforces and CSES. Codeforces problems are fetched from the Codeforces problemset API. CSES does not provide the same kind of API for this use case, so the backend reads the public CSES problemset page and extracts problem data from it. The backend then converts both sources into the same internal shape: platform, external id, name, topic, difficulty, and link.

## Why I Made It

When practicing competitive programming, the hardest part is not always solving the next problem. Sometimes it is deciding what the next problem should be. I often found that practice could become scattered: one problem from Codeforces, another from CSES, some solved problems forgotten, and no clear record of what was saved for later.

This project is meant to make that process calmer. Instead of being a large learning platform, it is a focused practice dashboard. The user chooses what they want to work on, gets a short recommendation list, and tracks what happens next.

## Main Features

- User registration and login
- Signed token authentication
- Problem recommendations based on topic, platform, and confidence level
- Support for Codeforces and CSES
- Waitlist for problems to solve later
- Completed-problem tracking
- Sidebar list of solved problems
- Direct links to the original problem pages
- Minimal, Notion-inspired interface
- Fixed dashboard layout with independent scrolling for problem lists

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

## Files I Wrote

`backend/app.py` is the entry point for the Flask backend. It creates the Flask application, enables CORS, registers the authentication and problem routes, initializes the database, and starts the development server on port 5000.

`backend/routes/auth.py` contains the login and registration routes. It handles username and password validation, stores password hashes, creates signed tokens, and includes the helper function used by protected routes to identify the current user.

`backend/routes/problems.py` contains the main recommendation logic. It fetches problems from Codeforces and CSES, maps platform data into a shared format, normalizes selected topics and platforms, excludes completed or waitlisted problems, and returns a balanced recommendation list. It also contains the route for updating problem progress.

`backend/utils/db.py` sets up the SQLite database connection and creates the needed tables. The database has a `users` table and a `progress` table. The `progress` table stores the problem status for each user, including whether a problem is waitlisted or completed.

`frontend/src/app/page.tsx` is the main dashboard page. It loads the current user’s saved data, handles topic/platform/confidence selection, requests recommendations, and updates progress when the user waitlists or completes a problem.

`frontend/src/components/Sidebar.tsx` renders the left sidebar. It shows the current page, a count of solved problems, and a scrollable list of completed problems. The completed list is paginated in groups of 10 so the sidebar does not become too crowded.

`frontend/src/components/ProblemsList.tsx` renders recommendation, waitlist, and completed sections. It displays each problem’s name, platform, topic, difficulty, source link, and any available actions.

`frontend/src/components/AuthPage.tsx` and `frontend/src/components/AuthForm.tsx` handle the login and registration screens. I separated the page logic from the form itself to keep the components easier to read.

`frontend/src/lib/api.ts` contains frontend helper functions for calling the Flask backend. It centralizes requests for authentication, dashboard data, recommendations, and progress updates.

`frontend/src/lib/types.ts` contains TypeScript types shared by frontend components. These types help keep the problem, user, auth, and recommendation data consistent.

`frontend/src/app/globals.css` contains the global styling and layout rules. I used Tailwind CSS for most component-level styling, but this file defines the app shell, fixed layout, scroll regions, colors, typography, and Notion-inspired design tokens.

## How Recommendations Work

The recommendation flow starts when the user clicks the Recommend button. The frontend sends selected topics, selected platforms, and confidence level to the backend.

If the user selects no topics, the backend treats that as all topics. If the user selects no platforms, it treats that as all supported platforms. This felt more natural than forcing the user to manually select every option.

The backend fetches available problems from each platform and removes anything already waitlisted or completed by the current user. It then sorts candidates by how close their difficulty is to the user’s confidence level. Low confidence prefers easy problems, medium confidence prefers medium problems, and high confidence prefers hard problems.

One design choice I made was to balance recommendations across platforms instead of taking every problem from the first source. This makes the output more varied when multiple platforms are selected.

## Design Choices

I wanted the interface to feel clean and quiet. Competitive programming problems are already mentally demanding, so I did not want the app itself to feel noisy. The UI is inspired by Notion: muted colors, light borders, simple cards, and minimal buttons.

I also chose a fixed dashboard layout. The whole page does not scroll. Instead, only the recommendation list scrolls, and the solved-problem list in the sidebar scrolls if it becomes long. This makes the filters and navigation stay in place while practicing.

For the backend, I chose Flask because it is simple and fits the size of the project. I chose SQLite because the app only needs local persistence for users and progress. A larger deployed version would probably use PostgreSQL or another hosted database, but SQLite is enough for this project.

I originally experimented with adding more platforms, but I kept the final version to Codeforces and CSES because they were the most reliable for this project. I preferred having fewer supported platforms that worked clearly over having extra filters that behaved unpredictably.

## Running the Project

You need two terminals: one for the backend and one for the frontend.

### Backend

```bash
cd backend
.venv/bin/python app.py
```

The backend runs at:

```text
http://127.0.0.1:5000
```

If the virtual environment does not exist, create it and install the backend dependencies:

```bash
cd backend
python3 -m venv .venv
.venv/bin/pip install flask flask-cors itsdangerous werkzeug
.venv/bin/python app.py
```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

The frontend usually runs at:

```text
http://localhost:3000
```

## API Overview

The backend exposes a few simple endpoints:

```text
POST /auth/register
POST /auth/login
GET  /problems
POST /recommendations
POST /progress
```

`/auth/register` and `/auth/login` return a signed token. The frontend stores this token in `localStorage` and sends it in the `Authorization` header for protected requests.

`/problems` returns the current user, waitlisted problems, and completed problems.

`/recommendations` receives selected topics, selected platforms, and confidence level, then returns recommended problems.

`/progress` updates a problem’s status to either waitlisted or completed.

## Database

The active database file is:

```text
backend/database.db
```

The database stores users and progress. I decided to store progress separately from fetched problem data because the problem sources are external and dynamic. The app does not need to permanently store every Codeforces or CSES problem. It only needs to remember what a user has saved or completed.

## Limitations

This is not meant to be a production-scale app. Authentication is simple, the database is local, and external problem fetching depends on Codeforces and CSES being available. CSES also requires HTML parsing, which is more fragile than using a formal API.

Topic mapping is also approximate. Codeforces has tags, which are useful. CSES sections are broader, so the app infers topic and difficulty from section names. This works well enough for a practice helper, but it is not perfect.

## AI and Assistance Disclosure

Some non-trivial parts of the project include citation comments in the source code. These comments mark places where outside help was used for logic such as topic mapping, CSES parsing, recommendation ranking, provider caching, and fixed scroll-region layout.

I kept those comments close to the relevant code so the assistance is visible and easy to understand.
