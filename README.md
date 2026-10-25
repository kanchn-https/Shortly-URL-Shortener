# URL Shortener

A full-stack URL shortener with a React dashboard and an Express API. Users can register, create and manage short links, and inspect recorded click events. Short links redirect through a Redis cache backed by MongoDB.

## Implemented features

- User registration and login with bcrypt-hashed passwords and JWTs stored in `httpOnly` cookies.
- Authenticated dashboard for creating links, listing owned links, editing destinations and aliases, enabling or disabling links, and deleting links.
- Public API endpoint for creating a link with or without a valid login cookie. The current frontend only exposes the creation form on the authenticated dashboard.
- Generated short codes and optional custom aliases. When an alias is set, both the alias and generated code resolve to the same link.
- HTTP 302 redirects with Redis cache-aside lookup and MongoDB fallback.
- Link click counts and per-click event records for timestamp, parsed browser, OS, device, referrer, and a SHA-256 IP hash.
- Authenticated analytics endpoint and dashboard modal displaying the click count and event records.
- Redis fixed-window rate limiting for link creation (10 requests per minute per request IP). If Redis is unavailable, the middleware logs the failure and permits the request.
- Light and dark themes, with the selected theme saved in browser local storage.


## Technology

- **Frontend:** React, TypeScript, Vite, React Router, TanStack Query, React Hook Form, Zod, Tailwind CSS
- **Backend:** Node.js, Express, Mongoose
- **Persistence:** MongoDB
- **Cache and rate limiting:** Upstash Redis REST client
- **Authentication:** JWT in an `httpOnly` cookie

## Getting started

### Requirements

- Node.js and npm
- A MongoDB connection string
- Upstash Redis REST URL and token for Redis-backed caching and rate limiting

### Backend

```bash
cd backend
cp .env.example .env
```

Set the following values in `backend/.env`:

```dotenv
MONGO_URI=your-mongodb-connection-string
UPSTASH_REDIS_REST_URL=your-upstash-rest-url
UPSTASH_REDIS_REST_TOKEN=your-upstash-rest-token
JWT_SECRET=your-long-random-secret
PORT=5000
NODE_ENV=development
FRONTEND_URL=http://localhost:5173
BASE_URL=http://localhost:5000
```

Then install dependencies and start the API:

```bash
npm install
npm run dev
```

The backend exits at startup if `JWT_SECRET` or `MONGO_URI` is missing, and it waits for MongoDB to connect before listening. Redis is used by redirect caching and link-creation rate limiting.

### Frontend

In a separate terminal:

```bash
cd frontend
npm install
npm run dev
```

The Vite development server runs on `http://localhost:5173`. The API client defaults to `http://localhost:5000`. To point a frontend build at another backend, set `VITE_API_BASE_URL` in the frontend environment, and configure the backend's `FRONTEND_URL` to allow the frontend origin. Set backend `BASE_URL` to the public backend URL so generated short URLs use the right host.

## Main API routes

| Method | Route | Access | Purpose |
|---|---|---|---|
| `POST` | `/api/auth/register` | Public | Register and set the auth cookie |
| `POST` | `/api/auth/login` | Public | Log in and set the auth cookie |
| `POST` | `/api/auth/logout` | Authenticated | Clear the auth cookie |
| `GET` | `/api/auth/me` | Authenticated | Return the current user |
| `POST` | `/api/shorten` | Public; auth optional | Create a short link |
| `GET` | `/api/links` | Authenticated | List the current user's links |
| `GET` | `/api/links/:id/analytics` | Authenticated, owner only | Retrieve click events for a link |
| `PUT` | `/api/links/:id` | Authenticated, owner only | Update a link |
| `DELETE` | `/api/links/:id` | Authenticated, owner only | Delete a link |
| `GET` | `/:shortCode` | Public | Resolve a code or alias and redirect |

## Repository layout

```text
backend/
  src/
    config/       MongoDB connection
    controllers/  Authentication, link, redirect, and analytics handlers
    middleware/   Cookie authentication and link-creation rate limit
    models/       User, link, and analytics event schemas
    routes/       Express route definitions
    utils/        Redis client, short-code and URL helpers, analytics helpers
frontend/
  src/
    api/          Typed API client
    components/   Dashboard forms, tables, and dialogs
    hooks/        Authentication query
    pages/        Login, registration, dashboard, and link-detail placeholder
```
