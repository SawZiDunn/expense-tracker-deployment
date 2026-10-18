# Expense Tracker

Full-stack expense tracker: a **React + TypeScript** frontend and an **ASP.NET Core 8 + MongoDB**
backend, wired together with Docker Compose so both run with one command.

The application code lives in two separate repositories, cloned into this folder:

- [expense-tracker-backend](https://github.com/SawZiDunn/expense-tracker-backend) - ASP.NET Core 8 Minimal API + MongoDB Atlas
- [expense-tracker-frontend](https://github.com/SawZiDunn/expense-tracker-frontend) - React 18 + TypeScript + Vite + Tailwind CSS

**This repository only holds the Docker Compose setup and documentation - it contains no
application code.**

## What's here

| File                 | Purpose                                          |
| -------------------- | ------------------------------------------------ |
| `docker-compose.yml` | Builds and runs both services                    |
| `.env.example`       | Template for the environment variables           |
| `.gitignore`         | Keeps `.env` and the two cloned repos out of git |

## Prerequisites

- Docker Engine with Compose v2 - check with `docker compose version`
- A MongoDB Atlas cluster and database user
- Git

---

# Run it with Docker

## 1. Clone the two projects into this folder

```bash
git clone https://github.com/SawZiDunn/expense-tracker-backend
git clone https://github.com/SawZiDunn/expense-tracker-frontend
```

You should end up with:

```
expense-tracker/
├── docker-compose.yml
├── .env
├── expense-tracker-backend/
└── expense-tracker-frontend/
```

## 2. Create the environment file

```bash
cp .env.example .env
```

Then edit `.env` and fill in your connection string:

```
MONGODB_CONNECTION_STRING=mongodb+srv://user:password@cluster.mongodb.net/?retryWrites=true&w=majority
VITE_API_URL=http://localhost:8080
```

> In MongoDB Atlas, make sure **Network Access** allows `0.0.0.0/0` (or your own IP).
> If your password contains `$` or `#`, wrap the whole value in single quotes.

## 3. Start both services

```bash
docker compose up --build
```

The first build takes a few minutes (NuGet restore + `npm ci`). Later runs are cached.

## 4. Open the app

| Service     | URL                   |
| ----------- | --------------------- |
| Frontend    | http://localhost:3000 |
| Backend API | http://localhost:8080 |

Stop everything with `Ctrl+C`, or `docker compose down` if running detached.

## What Compose builds

|                | frontend                                       | backend                                                    |
| -------------- | ---------------------------------------------- | ---------------------------------------------------------- |
| Built from     | `./expense-tracker-frontend`                   | `./expense-tracker-backend`                                |
| Container      | Node build → nginx serving static files        | .NET 8 runtime running the API                             |
| Ports          | `80` inside, published as **3000**             | `8080` inside and out                                      |
| Key settings   | `VITE_API_URL` → `http://localhost:8080`       | `MONGODB_CONNECTION_STRING`, `FRONTEND_URL` → `http://localhost:3000` |

`VITE_API_URL` is baked into the frontend bundle at build time, and `FRONTEND_URL` is what
the backend allows through CORS. Both must be addresses the **browser** can reach - never
Docker-internal hostnames like `backend:8080`.

## Common commands

```bash
docker compose up --build -d      # start in the background
docker compose logs -f            # follow both services' logs
docker compose logs -f backend    # follow one service
docker compose ps                 # what's running
docker compose down               # stop and remove containers
docker compose build --no-cache   # rebuild from scratch
```

## Troubleshooting

| Symptom                                                        | Cause / fix                                                          |
| -------------------------------------------------------------- | -------------------------------------------------------------------- |
| `The "MONGODB_CONNECTION_STRING" variable is not set`          | No `.env` file - do step 2                                            |
| Backend returns 500 with `bad auth : authentication failed`    | Wrong MongoDB username/password. Fix `.env`, then `docker compose up` |
| Browser reports a CORS error                                   | `FRONTEND_URL` must match the address bar origin exactly, no trailing slash |
| `port is already allocated`                                    | Something already uses 3000 or 8080 - change the left side of the mapping |
| Frontend loads but shows no data                               | `VITE_API_URL` is wrong; it's baked in, so rebuild the frontend image |

Two things that catch people out:

- **`.env` is read by Compose**, not by the containers directly. After editing it, run
  `docker compose up -d` so containers are recreated with the new values.
- **`.env` is gitignored.** Never commit a real connection string.

---

# Running without Docker (optional)

```bash
# backend - http://localhost:5000
cd expense-tracker-backend
dotnet run

# frontend - http://localhost:5173 (in a separate terminal)
cd expense-tracker-frontend
npm install
npm run dev
```

In this mode the backend reads its own `expense-tracker-backend/.env`
(`MONGODB_CONNECTION_STRING`, `FRONTEND_URL=http://localhost:5173`).

> On this machine `dotnet` is not on `PATH` - it lives at
> `/snap/dotnet-sdk-80/current/usr/bin/dotnet`.

Each project repository also has its own `README.md`, `DEPLOYMENT.md` and `DOCKER_SETUP.md`.
