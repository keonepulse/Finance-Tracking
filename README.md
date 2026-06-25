# Finance Tracking Application

A full-stack finance tracking web application that delivers real-time exchange rates and asset prices, historical trend analysis with interactive charts, and social features such as comments, profiles, and follows. The frontend is a React single-page application; the backend is a Flask REST API backed by SQLite.

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Tech Stack](#tech-stack)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [API Overview](#api-overview)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Backend Setup](#backend-setup)
  - [Frontend Setup](#frontend-setup)
  - [Development Proxy](#development-proxy)
- [Docker Deployment](#docker-deployment)
- [Testing](#testing)
- [Troubleshooting](#troubleshooting)
- [Roadmap](#roadmap)

## Overview

The project was restructured from a traditional Flask + Jinja multi-page application into a React-based single-page application (SPA). The Flask backend now serves REST-style JSON endpoints, authentication, and data access, while React handles all rendering and routing on the client. This eliminates full-page reloads, speeds up navigation, and allows live market data to be shared across pages without repeated requests.

- **Frontend** — React with Vite and React Router. A central `LiveDataContext` polls the backend every 60 seconds and feeds the live ticker bar and all pages from a single shared dataset. Polling pauses automatically when the browser tab is in the background.
- **Backend** — Flask served by Waitress, with SQLAlchemy models on SQLite. Market data is sourced from external financial APIs, news is aggregated via feed parsing, and responses are cached to reduce upstream calls.

## Features

- **Live Market Data** — up-to-date exchange rates and asset prices displayed on the homepage and in a persistent ticker-style top bar with equal-width chips, price-change highlighting, and smooth horizontal scrolling on small screens.
- **Historical Charts** — asset detail and analysis pages render daily, weekly, monthly, and yearly charts, including client-side RSI calculation.
- **Currency Converter** — client-side conversion between assets using live rates.
- **News Feed** — aggregated financial news.
- **Authentication** — registration, login, and session-based authentication.
- **Social Features** — comment threads with replies, likes and dislikes, user profiles, and follow/unfollow functionality.
- **Price Alarms** — backend endpoints for setting and clearing price alerts.
- **Admin Panel** — Flask-Admin integration for data management.
- **Fast Navigation** — SPA routing keeps the layout mounted while page content changes; no full page reloads.

## Tech Stack

### Frontend

| Technology | Purpose |
|---|---|
| React 18 | UI library |
| Vite | Build tool and dev server |
| React Router | Client-side routing |
| React Context API | Shared live-data state |
| Chart.js | Historical charts |
| CSS3 | Styling |

### Backend

| Technology | Purpose |
|---|---|
| Python 3.10+ / Flask | Web framework and REST API |
| Flask-SQLAlchemy | ORM over SQLite |
| Flask-Migrate | Database migrations |
| Flask-Caching | Response caching |
| Flask-Admin | Admin interface |
| Waitress | Production WSGI server |
| yfinance | Market data source |
| feedparser | News aggregation |
| pytest | Testing |

## Architecture

```
┌────────────────────┐      JSON over HTTP      ┌────────────────────┐        ┌──────────────┐
│   React SPA        │ ───────────────────────► │   Flask API        │ ─────► │    SQLite    │
│   Vite (5173)      │ ◄─────────────────────── │   Waitress (8000)  │ ◄───── │ (SQLAlchemy) │
│   LiveDataContext  │                          └─────────┬──────────┘        └──────────────┘
└────────────────────┘                                    │
                                                          ▼
                                               ┌────────────────────┐
                                               │ External financial │
                                               │ data and news APIs │
                                               └────────────────────┘
```

- The React frontend requests data from the backend's `/api/*` endpoints; session cookies handle authentication.
- `LiveDataContext` fetches `/api/data` once every 60 seconds and distributes it to the top bar, homepage, news, and converter pages — navigating between pages triggers no additional requests.
- The backend fetches market data from external providers, caches results, and persists users, comments, and follows in SQLite.

## Project Structure

```
.
├── app/                        # Flask backend
│   ├── __init__.py             # App factory, logging, request hooks
│   ├── config.py               # Configuration
│   ├── models.py               # SQLAlchemy models (users, comments, follows, ...)
│   ├── admin.py                # Flask-Admin setup
│   ├── utils.py                # Helpers
│   ├── routes/
│   │   ├── main.py             # Live data and asset endpoints
│   │   ├── auth.py             # Register, login, logout
│   │   ├── comments.py         # Comment threads, likes, replies
│   │   ├── profile.py          # Profiles and follows
│   │   └── json_api.py         # JSON endpoints for the SPA
│   ├── static/                 # Static assets
│   └── templates/              # Legacy Jinja templates
├── finance-react/              # React SPA frontend
│   ├── vite.config.js          # Dev server and API proxy
│   └── src/
│       ├── context/            # LiveDataContext (shared live data)
│       ├── components/         # Layout, LiveRateBar, Navbar, ...
│       └── pages/              # Home, Converter, Analysis, News, About,
│                               # Comments, Profile, Login, Register,
│                               # AssetDetail, NotFound
├── run.py                      # Backend entry point (Waitress on port 8000)
├── requirements.txt            # Python dependencies
├── test.py                     # Backend tests
├── Dockerfile                  # Backend container image
└── docker-compose.yml          # Backend + frontend orchestration
```

## API Overview

| Method | Endpoint | Description |
|---|---|---|
| GET | `/api/data` | Live market data for all tracked assets |
| GET | `/asset/<name>/chart/<timeframe>` | Historical chart data for an asset |
| GET | `/api/comments` | Full comment tree |
| GET | `/api/profile/<username>` | Profile with comments and follow lists |
| GET | `/api/me` | Currently authenticated user |
| POST | `/login`, `/register`, `/logout` | Authentication |
| POST | `/add_comment`, `/like_comment`, `/dislike_comment`, `/delete_comment` | Comment interactions |
| POST | `/follow`, `/unfollow` | Follow management |
| POST | `/set_alarm`, `/clear_alarm` | Price alerts |

## Getting Started

### Prerequisites

- Python 3.10 or later
- Node.js 18 or later and npm

### Backend Setup

From the project root:

```bash
python -m venv env
source env/bin/activate      # macOS/Linux
env\Scripts\activate         # Windows

pip install -r requirements.txt
python run.py
```

The API runs at `http://localhost:8000`. The SQLite database is created automatically on first start.

### Frontend Setup

In a separate terminal:

```bash
cd finance-react
npm install
npm run dev
```

The application is available at `http://localhost:5173`.

### Development Proxy

The Vite dev server proxies all API and session routes (`/api`, `/asset`, `/login`, `/profile`, and others) to the Flask backend, so session cookies work without manual CORS configuration. The backend address defaults to `http://localhost:8000` and can be overridden with the `VITE_FLASK_URL` environment variable.

## Docker Deployment

Start the entire application with a single command:

```bash
docker compose up --build
```

- React SPA: `http://localhost:5173`
- Flask API: `http://localhost:8000`

The frontend container forwards API requests to the backend container over the Docker network. Use `docker compose up --build -d` to run in the background and `docker compose down` to stop.

The compose file mounts `instance/database.db` and the `logs/` directory as volumes so data and logs persist across container restarts.

## Testing

Backend tests are run with pytest:

```bash
pytest test.py
```

Coverage focuses on API endpoint validation, business logic, and live/historical data responses. Manual validation should confirm that navigation triggers no full reloads, the top bar stays stable across screen sizes, and pages reuse the shared live dataset.

## Troubleshooting

- **Frontend cannot reach the backend** — confirm the backend is running on port 8000 and the Vite proxy or `VITE_FLASK_URL` points to it.
- **Slow data refresh** — check the 60-second polling interval and upstream API response times.
- **Missing chart data** — verify the asset history endpoints return data for the selected timeframe.
- **Dependency errors** — reinstall Python and npm dependencies from a clean environment.
- **Flash messages not visible** — server-side flash messages do not appear in the SPA; form errors are produced client-side.

## Roadmap

### Short-Term

- Finalize the React SPA migration
- Improve top bar ticker responsiveness
- Expand API coverage for all pages
- Add loading and skeleton states

### Mid-Term

- Smarter caching for frequently requested data
- Authentication and session improvements
- Advanced chart filtering and comparisons
- Better mobile optimization

### Long-Term

- WebSocket-based real-time updates instead of polling
- Role-based access control
- Notification and alert system
- Multi-language support
- Premium analytics features
