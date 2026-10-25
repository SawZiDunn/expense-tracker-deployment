# Expense Tracker - Deployment

Runs the full expense tracker - React frontend, ASP.NET Core backend and MongoDB - with a
single command.

This repository contains **no application code**. It holds the Compose file and
documentation; the services run from images published on Docker Hub.

## Quick start

Requires Docker with Compose v2 - check with `docker compose version`. Works on Linux,
Windows and macOS, both Intel and Apple Silicon.

```bash
git clone https://github.com/SawZiDunn/expense-tracker-deployment
cd expense-tracker-deployment
docker compose up -d
```

Open **http://localhost:3000**.

Nothing else is needed: no `.env` file, no MongoDB account, no local source code. Compose
pulls these images on the first run:

| Service  | Image                                       |
| -------- | ------------------------------------------- |
| frontend | `sawzidunn/expense-tracker-frontend:latest` |
| backend  | `sawzidunn/expense-tracker-backend:latest`  |
| mongo    | `mongo:7`                                   |

To compile the images from the two source repositories instead, see
[Build from source](#build-from-source-optional). A bare `up --build` does not build
anything - the base file has no `build:` sections.

## Application code

| Repository                                                                        | Stack                                   |
| --------------------------------------------------------------------------------- | --------------------------------------- |
| [expense-tracker-backend](https://github.com/SawZiDunn/expense-tracker-backend)   | ASP.NET Core 8 Minimal API              |
| [expense-tracker-frontend](https://github.com/SawZiDunn/expense-tracker-frontend) | React 18 + TypeScript + Vite + Tailwind |

You only need these to **build the images yourself**. The quick start uses the published
ones.

## Build from source (optional)

```bash
git clone https://github.com/SawZiDunn/expense-tracker-backend
git clone https://github.com/SawZiDunn/expense-tracker-frontend

docker compose -f docker-compose.yml -f docker-compose.build.yml up --build
```

`docker-compose.yml` on its own only pulls. The second file adds the `build:` sections, and
keeping them separate is what lets a plain `docker compose up` work on a checkout that has
no source code.

The built images are tagged `expense-tracker-backend:local` and
`expense-tracker-frontend:local`, separate from the published `sawzidunn/...:latest` tags.
That way `docker compose ps` shows which stack is running, and pulling the published images
never overwrites a local build.

## Ports

| Service     | URL                   |
| ----------- | --------------------- |
| Frontend    | http://localhost:3000 |
| Backend API | http://localhost:8080 |

If either port is taken, change the left-hand number in `docker-compose.yml`. The backend
also only accepts browser calls from `http://localhost:3000`; if you move the frontend to a
different port, update `FRONTEND_URL` in the same file.

## Database: local MongoDB or Atlas

The project works with either one. The local container is the default and needs no setup.

| Mode                          | Setup                            | Data lives in                           |
| ----------------------------- | -------------------------------- | --------------------------------------- |
| **Local container** (default) | nothing - `docker compose up -d` | the `mongo-data` volume on this machine |
| **MongoDB Atlas** (cloud)     | set `MONGODB_CONNECTION_STRING`  | your Atlas cluster                      |

Both use the same database and collection, `ExpenseTrackerDB.expenses`, because those names
come from the backend's `appsettings.json` - not from the connection string.

To use Atlas, create a `.env` file next to `docker-compose.yml`:

```
MONGODB_CONNECTION_STRING=mongodb+srv://user:password@cluster.mongodb.net/
```

then run `docker compose up -d`. Compose passes the value to the backend, which takes
precedence over the local container. Comment the line out (or delete `.env`) to go back to
the local database. Your Atlas cluster's network access list must allow your IP address.

## Updating and data

```bash
docker compose pull      # fetch newer images
docker compose up -d
docker compose down      # stop, keep the data
docker compose down -v   # stop and delete the database volume
```

MongoDB data lives in the `mongo-data` volume when you use the local container. It survives
`restart`, `down` and `up -d`; only `down -v` deletes it. The images are tagged `latest`, so
an existing copy is not refreshed by `up` alone - run `docker compose pull` to update.

## Publishing updated images (maintainers)

The images are multi-architecture (`linux/amd64` + `linux/arm64`) and must be pushed with
`buildx`:

```bash
docker login

docker buildx build --platform linux/amd64,linux/arm64 \
  -t sawzidunn/expense-tracker-backend:latest --push ./expense-tracker-backend

docker buildx build --platform linux/amd64,linux/arm64 \
  --build-arg VITE_API_URL=http://localhost:8080 \
  -t sawzidunn/expense-tracker-frontend:latest --push ./expense-tracker-frontend
```

Use `buildx`, not `docker compose build && docker compose push` - the latter replaces the
manifest with an amd64-only image and breaks Apple Silicon Macs. `VITE_API_URL` is baked
into the frontend at build time, so the published image is for `localhost` use; a real
deployment domain needs its own frontend build.
