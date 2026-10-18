# Expense Tracker

Workspace for the two Expense Tracker repositories. They are **separate git repos
cloned into this folder**

## Setup

### 1. Clone the two repos here

```bash
git clone https://github.com/SawZiDunn/expense-tracker-backend
git clone https://github.com/SawZiDunn/expense-tracker-frontend
```

```
expense-tracker-deployment/
├── expense-tracker-backend/
└── expense-tracker-frontend/
```

### 2. Create the env files

**`expense-tracker-backend/.env`** - required, the API returns 500 without it:

```
MONGODB_CONNECTION_STRING=mongodb+srv://user:pass@cluster.mongodb.net/
FRONTEND_URL=http://localhost:5173
```

**`.env` in this folder** - only needed for Docker Compose, start from the example:

```bash
cp .env.example .env      # then fill in MONGODB_CONNECTION_STRING
```

### 3. Run it

**Docker (both services):**

```bash
docker compose up --build
```

**Or locally, in two terminals:**

```bash
cd expense-tracker-backend  && dotnet run                        # http://localhost:5000
cd expense-tracker-frontend && npm install && npm run dev        # http://localhost:5173
```

## Ports

| Service  | Local dev | Docker |
| -------- | --------- | ------ |
| Frontend | 5173      | 3000   |
| Backend  | 5000      | 8080   |

## Notes

- `FRONTEND_URL` in the backend must match the origin the **browser** uses, or CORS
  blocks every request.
- `.env` files are read **once at startup** - restart the service after editing them.
- Each repo has its own `README.md`, `DEPLOYMENT.md` and `DOCKER_SETUP.md`.
- On this machine `dotnet` is not on `PATH`; it lives at
  `/snap/dotnet-sdk-80/current/usr/bin/dotnet`.
