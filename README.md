[README.md](https://github.com/user-attachments/files/31374538/README.md)
# Alumni Influencers Platform

A microservices-based web platform built for the *Advanced Server-Side Development* module at the University of Westminster.

- **Part 1 — Alumni Influencers API**: alumni registration, profile management, and a daily "blind bidding" system for featured alumni slots.
- **Part 2 — University Analytics & Intelligence Dashboard**: a web client that visualises alumni data (skills gaps, career pathways, industry trends) for university staff, consuming the alumni data via scoped API keys.

> Coursework mark: 75/100

## Architecture

The system is split into three independent Express services plus a shared internal package, each with its own SQLite database:

| Service | Port | Responsibility |
|---|---|---|
| `authenticationservice` | `3000` | User accounts, JWT/session auth, password reset, API key issuance, scoping & usage stats, developer portal (`/dev`) |
| `alumniservice` | `3001` | Alumni profiles, degrees/certifications/licences/courses, blind bidding, monthly limit enforcement, cron-driven daily winner selection, alumni-facing UI |
| `statservice` | `3002` | Analytics dashboard UI, chart data endpoints, CSV/PDF export, CSRF protection |
| `shared-utils` | — | Shared helpers used across services (JWT/bearer token handling, password hashing, standard API response format) |

Services talk to each other over HTTP using service-to-service API keys (e.g. `statservice` calls `alumniservice`, both call `authenticationservice` to verify identity).

## Tech Stack

- **Backend:** Node.js, Express 5
- **Database:** SQLite (per-service `.db` file)
- **Frontend:** HTML, CSS, vanilla JavaScript, Chart.js (served as static pages from each service's `public/` folder)
- **Auth:** JWT (access + refresh tokens) for users, scoped API keys for service-to-service and third-party client access
- **Security:** bcrypt password hashing, Helmet, CORS, CSRF protection (`csrf-csrf`), express-rate-limit, express-validator
- **Docs:** Swagger / OpenAPI per service, served at `/api-docs`
- **Scheduling:** node-cron for monthly bid-count resets and the nightly (6PM) "Alumni of the Day" winner selection
- **Export:** `json2csv` and `pdfkit` for report generation

## Project Structure

```
Advanced Server Side Web Development/
├── authenticationservice/   # Auth, users, API keys, developer portal
│   ├── routes/               # userroute, apikeyroute, devroute
│   ├── middleware/           # jwtAuthMiddleware, apikeyvalidator
│   ├── daos/ services/ database/
│   ├── public/                # dev login/register/dashboard pages
│   ├── swagger.js
│   └── index.js
├── alumniservice/            # Alumni profiles + bidding
│   ├── routes/                # alumniroute, bidroute, uiroute, armobileclientroute
│   ├── middleware/            # jwtAuthMiddleware, apikeyMiddleware
│   ├── cronjobs/               # monthly reset + daily winner selection
│   ├── daos/ services/ database/
│   ├── public/                 # login/register/profile/dashboard/mybids pages
│   ├── swagger.js
│   └── index.js
├── statservice/               # Analytics dashboard + exports
│   ├── routes/                 # dataroute, exportroute, userroute, viewroute, uiroute
│   ├── middleware/             # jwtAuthMiddleware, csrf
│   ├── daos/ services/ database/
│   ├── public/                  # login/register/dashboard/charts/alumni pages
│   ├── swagger.js
│   └── index.js
└── shared-utils/               # bearerToken.js, hashService.js, reponse.js
```

## Getting Started

### Prerequisites
- Node.js v18+ (developed on v22)

### Setup

Each service is installed and run independently.

```bash
# 1. Authentication service (must be running first — the others verify against it)
cd authenticationservice
npm install
cp .env.example .env    # fill in secrets, see below
npm start                # listens on PORT (default 3000)

# 2. Alumni service (in a new terminal)
cd alumniservice
npm install
cp .env.example .env
npm start                # listens on PORT (default 3001)

# 3. Analytics/stats service (in a new terminal)
cd statservice
npm install
cp .env.example .env
npm start                # listens on PORT (default 3002)

# 4. Shared utils is a local dependency, not a standalone service — no need to run it separately
```

Once all three are running:
- Alumni-facing site: `http://localhost:3001/ui`
- Analytics dashboard: `http://localhost:3002/ui`
- Developer portal (API key self-service): `http://localhost:3000/dev`

### Environment Variables

Each service reads its own `.env`. Typical variables include:

```
PORT=
AUTH_SERVER_ID=
BASE_URL=                  # URL of the authentication service
AUTH_SERVICE_API_KEY=      # this service's key to call the auth service
JWT_SECRET=
REFRESH_SECRET=
COOKIE_SECRET=             # statservice only
CSRF_SECRET=               # statservice only
ALUMNI_SERVICE_URL=        # statservice only, points at alumniservice
ALUMNI_API_KEY=            # statservice only, key to call alumniservice
```

> ⚠️ **Note:** the zipped project as submitted included real `.env` files with live secrets. Before pushing to GitHub, delete the `.env` files, commit `.env.example` templates instead, and add `.env`, `*.db`, and `node_modules/` to a `.gitignore`. Rotate the JWT/refresh secrets and API keys if the original `.env` values are ever exposed publicly.

## API Documentation

Each service exposes its own interactive Swagger UI:

```
http://localhost:3000/api-docs   # authenticationservice
http://localhost:3001/api-docs   # alumniservice
http://localhost:3002/api-docs   # statservice
```

## Key Features

**Authentication service**
- User creation, retrieval, password reset (`/user`)
- API key creation, listing, validation, regeneration and revocation, with usage stats (`/apikey`, `/dev`)
- Developer self-service portal with login/register/dashboard pages

**Alumni service**
- Profile create/update/retrieve, degrees/certifications/employment history (`/alumni`)
- Blind bidding — place, check status, view today's/all bids, withdraw last bid (`/bid`)
- Monthly bid-limit tracking (`/alumni/monthly-limits`)
- Scoped public endpoints for the AR client (`/alumni/alumni-of-the-day`, `read:alumni_of_day` scope) and analytics consumers (`read:analytics` scope on certifications/degrees/employment)
- Cron jobs: monthly appearance-count reset (1st of month) and nightly (6PM) blind-bid winner selection

**Analytics service (dashboard)**
- Dashboard, charts and alumni-list pages, CSRF-protected forms
- Data endpoints for stats, certifications, alumni-by-sector, career pathways, top employers, geographic distribution, graduation trends, tech adoption (`/data`)
- CSV and PDF export (`/export/csv/:type`, `/export/pdf`)


## License

For educational/portfolio use.
