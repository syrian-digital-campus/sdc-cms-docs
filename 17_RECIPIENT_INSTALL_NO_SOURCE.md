# Syrian Digital Campus — install package (no source code)

This guide is for **server operators** who receive a **binary delivery** only (no Git repository). Follow the steps in order until you can sign in as **admin** and use the application in the browser.

---

## 1) What you should receive (ZIP contents)

Your developer should give you a ZIP that matches the tree below (same files may also be summarized in the table under it).

### Layout of the delivered ZIP (after unzip)

Use any root folder name (example: **`SDC-CMS`**). File names may differ slightly (e.g. **`cms-1.1.0.jar`**).

**Not inside the ZIP:** **`.env`** (secrets) — you create it on the server by copying **`.env.example`** and editing values.

```text
SDC-CMS/
├── 17_RECIPIENT_INSTALL_NO_SOURCE.md    # this full guide (recommended)
├── cms-1.0.0.jar                         # Spring Boot backend (version may differ)
├── docker-compose.postgres.yml
├── docker-compose.ui.yml
├── nginx-docker.conf                     # must be a file, not a folder
├── .env.example                          # copy → .env before starting Postgres
├── start-backend.example.ps1             # optional Windows helper (copy/edit → start-backend.ps1)
├── README-FIRST-INSTALL.txt
├── Prerequisites.txt
├── RELEASE-NOTES.txt
└── frontend/                             # production Angular build (contents of dist/.../browser/)
    ├── index.html
    ├── assets/                           # icons, i18n, hashed bundles, etc.
    └── …                                 # other .js / .css files from the build
```

The **`uploads/`** directory (for **`FILE_STORAGE_PATH`**) is **not** shipped in the ZIP; it is created when you first run the backend or your start script.

---

### File list (same content as the tree)

| Item | Description |
|------|-------------|
| **`cms-1.0.0.jar`** | Backend application (version number may differ). |
| **`frontend/`** | **All files** from the production web build (the `browser` output folder — `index.html`, `*.js`, `assets/`, etc.). |
| **`nginx-docker.conf`** | Nginx site config for the optional Docker UI container (see §5). |
| **`docker-compose.postgres.yml`** | Starts PostgreSQL only (see §3). |
| **`README-FIRST-INSTALL.txt`** | Short install checklist. |
| **`Prerequisites.txt`** | Required software before install. |
| **`RELEASE-NOTES.txt`** | Version line, upgrade hints, admin UI build-badge behaviour. |
| **`.env.example`** | Template for Postgres variables; copy to **`.env`** before `docker compose` (see §3). |
| **`start-backend.example.ps1`** | Optional Windows helper: reads **`.env`**, sets **`DB_*`**, runs **`java -jar`** — copy to `start-backend.ps1`, edit profile/init flags at bottom. |
| **`17_RECIPIENT_INSTALL_NO_SOURCE.md`** | This installation guide (recommended inside the ZIP). |

You do **not** need Maven, Node.js, or the source tree to **run** this package.

### Default ports used in this guide (change if busy)

| Service | Default | Used for |
|---------|---------|----------|
| PostgreSQL (host) | **5432** | Database connections from the JAR |
| Spring Boot (JAR) | **8080** | REST API |
| Nginx UI (Docker) | **8081** → container 80 | Browser UI |

If any of these ports are **already in use** on the machine, you **must** change the published port or the JAR port — see **§9**.

For a **full list** of environment variables the backend understands (and what each one does), see **§11**.

---

## 2) Prerequisites on the server or PC

Install:

1. **Java 17 or newer** (e.g. [Eclipse Temurin](https://adoptium.net/)) — required to run the JAR.  
2. **Docker Desktop** (Windows/macOS) or **Docker Engine + Compose** (Linux) — recommended for PostgreSQL.

Check in a terminal:

```text
java -version
docker --version
docker compose version
```

---

## 3) Start PostgreSQL (Docker)

1. Place `docker-compose.postgres.yml` in an empty folder (e.g. `C:\sdc-install` or `/opt/sdc-install`).
2. Create a `.env` file **in the same folder** with strong values:

```env
POSTGRES_DB=university_cms
POSTGRES_USER=postgres
POSTGRES_PASSWORD=Choose-A-Strong-Password-Here
POSTGRES_PORT=5432
```

3. Start the database:

```bash
docker compose -f docker-compose.postgres.yml up -d
```

4. Wait until the container is healthy / running (`docker ps`).

**Note:** The backend expects database name **`university_cms`** and user **`postgres`** by default unless you override `DB_*` variables (see §4).

---

## 4) First startup — create demo admin and sample data

The first run uses the **`dev`** profile and **data initialization** so that an **admin** account and demo content are created automatically. **Use this only for the first install or a closed lab** — not for a public internet deployment without hardening.

### 4.1 Choose folders

- **`INSTALL_DIR`** — folder containing `cms-1.0.0.jar` (e.g. `C:\sdc-install`).  
- **`FILES_DIR`** — folder where uploaded files will be stored (must persist across restarts), e.g. `C:\sdc-install\uploads`.

Create the uploads folder if it does not exist.

### 4.2 Environment variables (Windows PowerShell example)

Run in **PowerShell** from `INSTALL_DIR` (adjust paths and secrets):

```powershell
$env:SPRING_PROFILES_ACTIVE = "dev"
$env:APP_DATA_INIT_ENABLED = "true"
$env:JWT_SECRET = "REPLACE-WITH-LONG-RANDOM-STRING-AT-LEAST-32-CHARS"
$env:DB_HOST = "localhost"
$env:DB_PORT = "5432"
$env:DB_NAME = "university_cms"
$env:DB_USER = "postgres"
$env:DB_PASSWORD = "Choose-A-Strong-Password-Here"
$env:FILE_STORAGE_PATH = "C:\sdc-install\uploads"
```

Linux/macOS (bash):

```bash
export SPRING_PROFILES_ACTIVE=dev
export APP_DATA_INIT_ENABLED=true
export JWT_SECRET='REPLACE-WITH-LONG-RANDOM-STRING-AT-LEAST-32-CHARS'
export DB_HOST=localhost
export DB_PORT=5432
export DB_NAME=university_cms
export DB_USER=postgres
export DB_PASSWORD='Choose-A-Strong-Password-Here'
export FILE_STORAGE_PATH=/opt/sdc-install/uploads
```

### 4.3 Run the backend

```powershell
cd C:\sdc-install
java -jar .\cms-1.0.0.jar
```

Wait until the log shows that the application **started** (e.g. “Started CmsApplication”) and, if present, lines about **seed** / **Synced** demo accounts.

The API listens on **port 8080** by default (`http://localhost:8080`).

---

## 5) Serve the web UI (nginx in Docker — recommended)

The production build uses **`/api/v1`** on the **same hostname** as the UI. Nginx should:

- serve files from **`frontend/`**  
- proxy **`/api/`** to the JVM on the host (**port 8080**).

1. Put **`nginx-docker.conf`** next to **`frontend/`** and **`docker-compose.ui.yml`** (see files at the end of this document).
2. On **Windows**, Docker must reach the host JVM: the sample uses **`host.docker.internal`**.
3. Start the UI:

```bash
docker compose -f docker-compose.ui.yml up -d
```

4. Open **`http://localhost:8081`** (or the port you mapped).

### CORS

**`localhost` and `127.0.0.1` are different origins.** If you open the UI at **`http://127.0.0.1:8081`** but only `http://localhost:8081` is allowed, the browser shows **“Invalid CORS request”**. Use the **same** host you type in the address bar everywhere, or allow both (default YAML includes `localhost` and `127.0.0.1` for ports **4200** and **8081** in current builds).

If the UI is on **`http://localhost:8081`** and the API on **`http://localhost:8080`**, the shipped JAR allows that origin. If you use **another origin** (different host or port), set before starting the JAR:

```text
APP_SECURITY_ALLOWED_ORIGINS_0=https://your-exact-ui-origin
```

---

## 6) Sign in

| Field | Value |
|--------|--------|
| **Username** | `admin` |
| **Password** | `password123` |

Change this password immediately after first login (through your planned account policy or DB procedure).

Other demo users may exist (e.g. students); treat them as **non-production** data.

---

## 7) After everything works — lock down (important)

Before exposing the system widely:

1. **Stop** the backend (Ctrl+C).  
2. Turn **off** automatic demo seeding:

```powershell
$env:APP_DATA_INIT_ENABLED = "false"
```

(bash: `export APP_DATA_INIT_ENABLED=false`)

3. Switch to the **production** profile:

```powershell
$env:SPRING_PROFILES_ACTIVE = "prod"
```

(bash: `export SPRING_PROFILES_ACTIVE=prod`)

4. Keep a **strong** `JWT_SECRET` and **strong** DB password.  
5. Put **HTTPS** in front (reverse proxy) for real users; see your operator guide for TLS.  
6. **Back up** PostgreSQL and `FILE_STORAGE_PATH` regularly.

Re-start:

```powershell
java -jar .\cms-1.0.0.jar
```

---

## 8) Troubleshooting (short)

| Symptom | What to check |
|---------|----------------|
| Backend exits on start | PostgreSQL running? `DB_*` and password correct? |
| Browser login fails / CORS | Exact UI URL must be allowed; set `APP_SECURITY_ALLOWED_ORIGINS_0`. |
| 403 on login, wrong path | API path must be **`/api/v1/auth/login`** (via nginx **`/api/`** proxy). |
| No admin user | First run must use **`dev`** + **`APP_DATA_INIT_ENABLED=true`** once; check logs for errors. |
| “Port already in use” / cannot bind | Another program owns the port — change mapping or `SERVER_PORT` (see **§9**). |

---

## 9) Port conflicts — what to change and how

Use this when **5432**, **8080**, or **8081** (or any port you chose) is already taken by another application on the same server.

### 9.1 PostgreSQL port already used (e.g. another Postgres on 5432)

**Goal:** Publish SDC’s Postgres on a **different host port** (example **5433**) while the database inside the container stays on 5432.

1. In **`.env`**, set:

```env
POSTGRES_PORT=5433
```

2. When starting the JAR, point the app at that **host** port:

```powershell
$env:DB_PORT = "5433"
```

(`DB_HOST` stays `localhost` if Docker maps the port to this machine.)

3. Recreate the container so the new mapping applies:

```bash
docker compose -f docker-compose.postgres.yml down
docker compose -f docker-compose.postgres.yml up -d
```

**If you use an existing company PostgreSQL server** (no Docker Postgres from this package): do not start `docker-compose.postgres.yml`. Set `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, and `DB_PASSWORD` to match that server. The database **`university_cms`** must exist (or your chosen name in `DB_NAME`), and the user needs rights to create/use objects after Flyway runs.

### 9.2 Backend API port already used (8080 busy)

**Goal:** Run the JAR on another port (example **18080**).

Before `java -jar`:

```powershell
$env:SERVER_PORT = "18080"
```

(bash: `export SERVER_PORT=18080`)

Spring Boot binds the embedded Tomcat to this port.

**Then update anything that talks to the API:**

- In **`nginx-docker.conf`**, change the upstream line to match:

```nginx
upstream backend {
    server host.docker.internal:18080;
}
```

- Reload or recreate the UI container after editing the file:

```bash
docker compose -f docker-compose.ui.yml up -d --force-recreate
```

### 9.3 UI port already used (8081 busy)

**Goal:** Expose the nginx UI on another host port (example **9081**).

In **`docker-compose.ui.yml`**, change the ports line from `"8081:80"` to:

```yaml
ports:
  - "9081:80"
```

Then open **`http://localhost:9081`** (or your server name).

**CORS:** If the browser URL is no longer `http://localhost:8081`, set an allowed origin before starting the JAR:

```powershell
$env:APP_SECURITY_ALLOWED_ORIGINS_0 = "http://localhost:9081"
```

Use the **exact** origin users type in the address bar (scheme + host + port, no trailing slash).

### 9.4 Quick checklist

| Conflict on | Change |
|-------------|--------|
| Host **5432** | `POSTGRES_PORT` in `.env` + `DB_PORT` for JAR |
| Host **8080** | `SERVER_PORT` + `nginx-docker.conf` `upstream` port |
| Host **8081** | `docker-compose.ui.yml` left side of `ports:` + `APP_SECURITY_ALLOWED_ORIGINS_0` if needed |

---

## 10) Developer: delivering a newer version (upgrade)

This section is for **developers** who ship updates to an operator who **does not** have the source tree.

### 10.1 If only the **frontend** changed

**Deliver:**

- A fresh **`frontend/`** folder: full contents of `dist/sdc-cms-frontend/browser/` after `npm ci` and `npm run build` (production).

**Operator steps:**

1. **Back up** the current `frontend/` folder (zip or copy).  
2. Replace **`frontend/`** entirely with the new build.  
3. Restart the UI container or clear CDN/browser cache if you use a cache layer:

```bash
docker compose -f docker-compose.ui.yml up -d --force-recreate
```

**Database / API:** unchanged. No JAR replacement required.

### 10.2 If only the **backend** changed

**Deliver:**

- The new **`cms-x.y.z.jar`** (same or newer version label).  
- Updated **`17_RECIPIENT_INSTALL_NO_SOURCE.md`** (or release notes) if ports, env vars, or first-install steps changed.

**Operator steps:**

1. **Stop** the running JAR (Ctrl+C or stop the service).  
2. **Back up PostgreSQL** (dump) and, if possible, the **`FILE_STORAGE_PATH`** directory.  
3. Replace the old JAR with the new file (keep the same `INSTALL_DIR` or update your service script).  
4. Start with the **same** environment variables as before (`DB_*`, `JWT_SECRET`, `FILE_STORAGE_PATH`, `SPRING_PROFILES_ACTIVE`, etc.).  
5. Watch the log on first start: **Flyway** may run new migrations automatically.

**Frontend:** unchanged unless the developer says the new API is incompatible (rare if versioned together).

### 10.3 If **both** frontend and backend changed

Deliver **both** new **`frontend/`** and new **JAR**. Apply **§10.2** then **§10.1** (order: stop backend → replace JAR → start backend → verify API → replace frontend → restart nginx).

### 10.4 What happens to the **database** (users, courses, uploads) on upgrade?

| Topic | What happens |
|-------|----------------|
| **Existing data** | Generally **kept**. The app uses **Flyway** migrations: on startup the new JAR applies any **new** SQL migrations to the **same** database. Rows (users, courses, enrollments, etc.) remain unless a migration explicitly changes or removes them (unusual for routine releases). |
| **Schema** | New JAR may add tables/columns/indexes via Flyway; the database evolves forward. |
| **First install vs upgrade** | **First install** with `APP_DATA_INIT_ENABLED=true` creates seed users/courses. On **upgrade**, keep **`APP_DATA_INIT_ENABLED=false`** in production so seed logic does not fight real data (dev profile sync may still reset known demo passwords if you mistakenly run `dev` with init — avoid on production). |
| **Uploaded files** | Stored under **`FILE_STORAGE_PATH`**. Replacing the JAR does **not** delete them if that path is unchanged. Always back up this folder with the DB. |
| **JWT / sessions** | After upgrade, if **`JWT_SECRET`** stays the same, existing access tokens may remain valid until expiry; if you **change** `JWT_SECRET`, users typically need to **sign in again**. |
| **Rollback** | If something goes wrong: restore the **previous JAR**, restore the **database dump** from before upgrade, restore **`FILE_STORAGE_PATH`** backup. |

**Operator rule of thumb:** before any backend upgrade, take a **DB dump** + copy **`FILE_STORAGE_PATH`**.

### 10.5 Version communication (developer → operator)

In each delivery, state clearly:

- **Backend:** JAR file name / version (e.g. `cms-1.1.0.jar`).  
- **Frontend:** build date or release tag.  
- **Database:** “Flyway will migrate from previous version X” or “fresh DB required” (only if you ever ship a breaking change — document it).  
- **Config changes:** new env vars, default port changes, or CORS notes.

---

## 11) Environment variables handbook

This section lists variables **operators** commonly set. Names follow **Spring Boot** relaxed binding: dotted properties in config become `UPPER_SNAKE_CASE` env vars (e.g. `app.security.require-https` → `APP_SECURITY_REQUIRE_HTTPS`).

### 11.1 Two different “`.env`” contexts (do not confuse them)

| Where | Purpose |
|-------|---------|
| **`.env` next to `docker-compose.postgres.yml`** | Read by **Docker Compose** only. Sets the Postgres **container** (image env + host port mapping). The JAR does **not** read this file automatically. |
| **Shell / Windows service / systemd** | Variables you set **before** `java -jar …`. The **JAR** reads these. |

If you use Compose for Postgres, you must align **Compose** (`POSTGRES_*`) with **JAR** (`DB_*`) manually (same database name, user, password, and the **host** port you mapped).

### 11.2 Backend JAR — database (`DB_*`)

| Variable | Default (if unset) | Responsibility |
|----------|-------------------|----------------|
| **`DB_HOST`** | `localhost` | Hostname or IP of PostgreSQL **as seen from the JVM** (use service name or IP if the DB runs in another container/network). |
| **`DB_PORT`** | `5432` | TCP port PostgreSQL listens on **at `DB_HOST`** (use your mapped host port if Postgres is published on e.g. 5433). |
| **`DB_NAME`** | `university_cms` | Database name inside Postgres. |
| **`DB_USER`** | `postgres` | DB login user. |
| **`DB_PASSWORD`** | `postgres` | DB password. **Must** be set to a strong value in production. |

**Alternative (advanced):** you may use standard Spring datasource variables instead, e.g. `SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD`, if your platform injects them that way.

### 11.3 Backend JAR — profiles and HTTP port

| Variable | Typical values | Responsibility |
|----------|----------------|----------------|
| **`SPRING_PROFILES_ACTIVE`** | `dev`, `prod`, or comma-separated list | Selects Spring profiles. **`dev`** enables verbose SQL logging and (unless overridden) seed-friendly defaults; **`prod`** tightens logging and sets stricter cookie behaviour (see shipped `application-prod.yaml`). |
| **`SERVER_PORT`** | `8080` (Spring default) | TCP port for the **embedded Tomcat** (REST API). Change when 8080 is busy; then update nginx `upstream` (§9). |

### 11.4 Backend JAR — demo seeding

| Variable | Default (if unset) | Responsibility |
|----------|-------------------|----------------|
| **`APP_DATA_INIT_ENABLED`** | `false` in base config; profile **`dev`** enables seed in YAML unless you override with this var | When **`true`**, runs startup logic that creates **demo admin, users, and sample courses** (`DataInitializer`). Use **`true`** only for **first install / lab**. For production after go-live, set **`false`** so real data is not overwritten or merged unexpectedly by seed logic. |

### 11.5 Backend JAR — JWT and sessions

| Variable | Default (if unset) | Responsibility |
|----------|-------------------|----------------|
| **`JWT_SECRET`** | A placeholder in the default YAML (unsafe) | **Signing key** for access and refresh tokens. **Required:** long random string (e.g. ≥ 32 characters) in any real deployment. Changing it **invalidates** existing tokens — users sign in again. |
| **`APP_JWT_ACCESS_TOKEN_VALIDITY`** | `900000` (15 minutes, milliseconds) | Lifetime of **access** JWT. |
| **`APP_JWT_REFRESH_TOKEN_VALIDITY`** | `86400000` (1 day, milliseconds) | Lifetime of **refresh** JWT / refresh cookie window. |

(These JWT TTL names follow Spring’s binding of `app.jwt.*` from `application.yaml`.)

### 11.6 Backend JAR — uploaded files on disk

| Variable | Default (if unset) | Responsibility |
|----------|-------------------|----------------|
| **`FILE_STORAGE_PATH`** | `/data/files` | Directory where **uploaded course materials and similar files** are stored. Must be **persistent** and included in **backups** with the database. Use an absolute path on the server. |

### 11.7 Backend JAR — security, CORS, HTTPS, cookies

Bound to `app.security.*` in configuration. Used for CORS (browser → API), optional HTTPS enforcement, and the **Secure** flag on refresh cookies.

| Variable | Default behaviour | Responsibility |
|----------|-------------------|----------------|
| **`APP_SECURITY_ALLOWED_ORIGINS_0`**, **`APP_SECURITY_ALLOWED_ORIGINS_1`**, … | Built-in list includes `http://localhost:4200`, `http://localhost:8081`, `http://127.0.0.1:4200`, `http://127.0.0.1:8081` in the shipped YAML | **Browser origins** allowed for CORS when the UI and API are on **different** schemes/hosts/ports. Use the **exact** origin (no path, no trailing slash). Add `_1`, `_2`, … for multiple UIs or admin portals. |
| **`APP_SECURITY_REQUIRE_HTTPS`** | `false` in base YAML | If **`true`**, Spring Security requires HTTPS on every request to the JVM. Usually **`false`** when **nginx** terminates TLS and forwards `X-Forwarded-Proto`. Set **`true`** only if the JAR serves TLS directly. |
| **`APP_SECURITY_SECURE_COOKIES`** | `false` in base YAML; **`true`** under **`prod`** profile | If **`true`**, refresh-token cookies get **`Secure`**, so browsers send them only over HTTPS. Required for real HTTPS frontends; if there is no HTTPS, logins using cookies may fail. |

### 11.8 Backend JAR — optional logging tweaks

| Variable | Responsibility |
|----------|----------------|
| **`LOGGING_LEVEL_ROOT`** | Overall log verbosity (e.g. `INFO`, `DEBUG`). |
| **`LOGGING_LEVEL_ORG_FLYWAYDB`** | Flyway migration logs (`INFO`, `DEBUG`, `WARN`). |

Useful when diagnosing startup or migration issues; align with your organisation’s logging policy.

### 11.9 Docker Compose only — PostgreSQL service (`.env` beside `docker-compose.postgres.yml`)

These variables are **not** Spring properties; Compose substitutes them into the compose file.

| Variable | Responsibility |
|----------|----------------|
| **`POSTGRES_DB`** | Database name created inside the container (default in compose file: `university_cms`). Should match **`DB_NAME`** for the JAR. |
| **`POSTGRES_USER`** | Superuser / application user inside the container. Should match **`DB_USER`** for the JAR. |
| **`POSTGRES_PASSWORD`** | Password for that user. Should match **`DB_PASSWORD`** for the JAR. |
| **`POSTGRES_PORT`** | **Host** port mapped to container `5432` (default `5432`). If changed, set **`DB_PORT`** on the JAR to the same value when connecting to `localhost`. |

### 11.10 Quick reference — who reads what

| Variable group | Read by |
|----------------|---------|
| `DB_*`, `JWT_SECRET`, `FILE_STORAGE_PATH`, `APP_*`, `SPRING_*`, `SERVER_PORT`, `LOGGING_LEVEL_*` | **Spring Boot JAR** |
| `POSTGRES_*` in `.env` | **Docker Compose** (Postgres container + port publish) |

---

## Appendix A — `docker-compose.postgres.yml`

```yaml
services:
  postgres:
    image: postgres:16-alpine
    container_name: sdc-postgres
    restart: unless-stopped
    environment:
      POSTGRES_DB: ${POSTGRES_DB:-university_cms}
      POSTGRES_USER: ${POSTGRES_USER:-postgres}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?set POSTGRES_PASSWORD in .env}
    ports:
      - "${POSTGRES_PORT:-5432}:5432"
    volumes:
      - sdc_pg_data:/var/lib/postgresql/data

volumes:
  sdc_pg_data:
```

---

## Appendix B — `docker-compose.ui.yml`

Place next to folder **`frontend`** (rename if your folder name differs).

```yaml
services:
  ui:
    image: nginx:1.25-alpine
    container_name: sdc-ui
    restart: unless-stopped
    ports:
      - "8081:80"
    volumes:
      - ./frontend:/usr/share/nginx/html:ro
      - ./nginx-docker.conf:/etc/nginx/conf.d/default.conf:ro

```

---

## Appendix C — `nginx-docker.conf`

**Note:** `proxy_pass` uses `host.docker.internal` so the container can reach the JAR on the host (Windows/macOS Docker Desktop; on Linux you may need `extra_hosts` or use the host gateway IP).

```nginx
upstream backend {
    server host.docker.internal:8080;
}

server {
    listen 80;
    server_name localhost;
    root /usr/share/nginx/html;
    index index.html;

    location /api/ {
        proxy_pass http://backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

---

## What the developer must put in `frontend/`

From the **built** Angular app (developer machine), copy the **contents** of:

`dist/sdc-cms-frontend/browser/`

into your ZIP folder **`frontend/`** (so `index.html` sits directly inside `frontend/`).

---

*Document version: 1.3 — §1 adds delivered ZIP directory tree.*
