# Recon backend

FastAPI service for user profiles, Google OAuth, refresh tokens, and role-gated user administration. It is part of the [Recon event platform](https://github.com/AbhiramVSA/Recon).

## API surface

The app mounts version 1 routes under `/api/v1`.

| Route | Purpose |
|---|---|
| `/api/v1/auth/google/login` | Start Google OAuth. |
| `/api/v1/auth/me` | Read the authenticated profile. |
| `/api/v1/auth/refresh` | Rotate the refresh token. |
| `/api/v1/auth/logout` | Revoke the refresh token. |
| `/api/v1/users` | Create or list user profiles. |
| `/api/v1/users/{user_id}` | Read, update, or delete a profile. |
| `/api/v1/users/me/apply` | Submit an applicant profile. |
| `/api/v1/users/{user_id}/approve` | Approve a user. |
| `/api/v1/users/{user_id}/reject` | Reject a user. |

List, update, delete, approve, and reject actions require the admin role. The current user's profile and applicant routes require authentication.

## Run locally

You need Python 3.12 or newer, PostgreSQL, and Redis. Google OAuth also needs a client ID and client secret.

Install the locked dependencies from the repository root:

```sh
uv sync --locked --no-install-project
```

Create a `.env` file in the repository root or `backend/`. Git ignores this file. Set the database fields, `REDIS_URL`, and a stable `SECRET_KEY`. Set `MODE=development` for a local database without SSL. Set `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, and `GOOGLE_REDIRECT_URI` for OAuth.

The service can build its PostgreSQL URL from `DATABASE_USER`, `DATABASE_PASSWORD`, `DATABASE_HOST`, `DATABASE_PORT`, and `DATABASE_NAME`. Set `ASYNC_DATABASE_URI` to provide the full connection string.

Apply the database migrations, then start the API from `backend/`:

```sh
cd backend
uv run --project .. alembic upgrade head
uv run --project .. uvicorn app.main:app --reload
```

Open `http://127.0.0.1:8000/docs` for the API schema. The root path returns a welcome message. `/db_check` checks the database connection.

## Startup requirements

At startup, the app connects to PostgreSQL and creates default roles. It assigns the admin role to users listed in `BOOTSTRAP_ADMIN_EMAILS` if those accounts exist. It also creates a Redis client, so set `REDIS_URL` before you start the service.

The app does not run Alembic migrations at startup. Apply the migrations before you start the API.
