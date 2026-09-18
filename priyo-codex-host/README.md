# PRIYO_CODEX HOST

A real multi-user Node/React/PostgreSQL hosting control panel with Argon2 authentication, HTTP-only sessions, quotas, expiry enforcement, uploads, and Docker-based isolated deployment.

## Local

Requirements: Docker + Docker Compose.

```bash
docker compose up --build
```

Open `http://localhost:8080`.

Default local admin:
- username: `admin`
- password: `admin12345`

Change these before any non-local use.

### Local deployment executor

The Compose file mounts the Docker socket into the app so deployment containers can be created. User workloads are started with non-root users where applicable, CPU/RAM/PID limits, dropped capabilities, `no-new-privileges`, read-only root filesystem, temporary filesystem, and no network.

**Important:** mounting `/var/run/docker.sock` gives the platform control over the host Docker daemon. It is required for this local first-party executor but should never be exposed to user containers. For a hardened production architecture, put the executor on a separate dedicated Docker host and expose only a restricted Docker API endpoint to the platform.

## GitHub

Create a repository, then:

```bash
git init
git add .
git commit -m "Initial PRIYO_CODEX HOST"
git branch -M main
git remote add origin YOUR_REPO_URL
git push -u origin main
```

Never commit `.env` or production secrets.

## Render

Render can build the supplied Dockerfile and create the PostgreSQL database using `render.yaml`.

1. Push this repository to GitHub.
2. In Render, create a Blueprint and select the repository.
3. Set `ADMIN_USERNAME`, `ADMIN_PASSWORD`, and `PUBLIC_BASE_URL`.
4. The database connection is supplied from the Render PostgreSQL resource.
5. Set `SESSION_SECRET` to a strong secret if you do not use Render's generated value.
6. Deploy.

### Critical deployment-executor limitation

A normal Render Docker web service does **not** provide a Docker daemon/socket for safely launching arbitrary sibling containers. Therefore the platform's Docker deployment feature requires `DOCKER_HOST` to point at a **separate, locked-down Docker executor**. The web service itself still uses Render PostgreSQL.

Do not mount a privileged host socket into a public Render web service and do not give user containers access to that socket.

## URLs

Local admin/user login: `http://localhost:8080/login` (the same login endpoint routes admins to the admin panel).

Render login: `https://YOUR-RENDER-HOST/`.

Admin creates users from the Admin Panel. Users create a project, upload a ZIP/file, then deploy.

## Supported project conventions

HTML: `index.html`, assets.

Node: `package.json` and an app listening on port 3000. The start command defaults to `npm start`.

Python: `requirements.txt` and Flask/FastAPI application. Set a start command such as:
`gunicorn app:app --bind 0.0.0.0:3000`
or
`uvicorn main:app --host 0.0.0.0 --port 3000`

## Security notes

Uploads are stored under `DATA_DIR`, never in the application source tree. ZIP member paths are normalized and rejected when they escape the project directory. File upload size and per-user quota are checked server-side. User code is not executed by the Node process.

For production, add a dedicated executor with rootless Docker/Firecracker/gVisor, egress controls, image scanning, persistent object storage, and an asynchronous job queue before exposing arbitrary untrusted workloads to the public internet.
