# Production deployment & programmer → server-admin handoff

This document ties together **what to read**, **what developers must deliver**, and **what server administrators do** to run Syrian Digital Campus (SDC-CMS) on a real host or subdomain—safely and repeatably.

---

## 1) Who should read what

| Role | Primary documents |
|------|-------------------|
| **Developer / release owner** | This file (§3 deliverables), [15_BEGINNER_START_AND_DEPLOY.md](./15_BEGINNER_START_AND_DEPLOY.md) (build commands), [PRODUCTION_CHECKLIST.md](../PRODUCTION_CHECKLIST.md) |
| **Server / DevOps admin** | This file (§4–§6), [12_DEPLOYMENT.md](./12_DEPLOYMENT.md) (nginx, Docker patterns), [11_SECURITY.md](./11_SECURITY.md) |
| **Everyone** | [QUICKSTART.md](../QUICKSTART.md) (local Docker try-out only—not production defaults) |

**Repository layout reminder**

| Part | Path |
|------|------|
| Frontend (Angular) | `sdc-cms-frontend/` |
| Backend (Spring Boot) | `sdc-cms-backend/` |
| Docker Compose (reference) | `sdc-cms-infra/` |

---

## 2) Production documentation map (use these for deploy & operate)

Use this checklist so nothing critical is missed:

1. **[15_BEGINNER_START_AND_DEPLOY.md](./15_BEGINNER_START_AND_DEPLOY.md)** — Build commands (`npm run build`, `mvn package`), **`prod` profile**, environment variables table, HTTPS/CORS notes.  
2. **[12_DEPLOYMENT.md](./12_DEPLOYMENT.md)** — Docker image build, sample Compose, **nginx** TLS + `/api` proxy, backups, health checks.  
3. **[PRODUCTION_CHECKLIST.md](../PRODUCTION_CHECKLIST.md)** — Disable demo data, JWT/DB secrets, remove weak defaults.  
4. **[11_SECURITY.md](./11_SECURITY.md)** — Cookies, HTTPS, hashing, operational security expectations.  
5. **[08_API_DOCUMENTATION.md](./08_API_DOCUMENTATION.md)** — API surface for integration and troubleshooting.  
6. **Backend config** — `sdc-cms-backend/src/main/resources/application.yaml`, `application-prod.yaml` (logging, `secure-cookies`).  
7. **Frontend production API** — `sdc-cms-frontend/src/environments/environment.prod.ts` uses **`apiUrl: '/api/v1'`** (same-origin behind nginx).

Optional deeper reading: `02_SETUP_GUIDE.md`, `13_TROUBLESHOOTING.md`, `sdc-cms-infra/README.md`.

**Binary-only handoff (no Git / no source for the recipient):** [17_RECIPIENT_INSTALL_NO_SOURCE.md](./17_RECIPIENT_INSTALL_NO_SOURCE.md) and template files under `sdc-cms-dist/recipient/`.

---

## 3) What the programmer must deliver (for the server admin)

Hand off a **release package** the admin can run **without** your source tree (or optionally with Compose). Minimum:

### 3.1 Version & support metadata

- **Release ID**: git tag or build number (e.g. `v1.2.0`).  
- **Commit hash** used for the build.  
- **JDK** version backend was built with (project targets **Java 17**).  
- **Node** major version used for the frontend build (e.g. 18+).

### 3.2 Backend artifact

- **`cms-*.jar`** from `sdc-cms-backend/target/` after `mvn package -DskipTests` (or with tests if policy requires).  
- **OR** a published **Docker image** (e.g. `registry.example/sdc-backend:1.2.0`) plus Dockerfile reference.

### 3.3 Frontend artifact

- Contents of **`sdc-cms-frontend/dist/sdc-cms-frontend/`** after production build (`npm ci` + `npm run build`).  
- **OR** a published **Docker image** for the static nginx frontend.

**Important:** The stock production build expects the browser to call **`/api/v1`** on the **same host** as the UI (see §5.1). If the admin uses a different layout, the frontend may need a **custom build** (§5.2).

### 3.4 Configuration template for the server (no secrets inside)

Deliver a filled-out **example** (placeholders only):

- Copy or adapt **`sdc-cms-infra/.env.example`** (repo root: `sdc-cms-infra/.env.example`).  
- List every variable the admin must set (see §6).  
- State clearly: **`APP_DATA_INIT_ENABLED=false`** in production (no demo `admin` / `password123`).

### 3.5 Runbook (short `README-INSTALL.txt` is enough)

The admin should receive **one page** with:

1. Prerequisites: PostgreSQL reachable, Java 17+ *or* Docker, nginx (if not using frontend container).  
2. **Startup order**: database → backend → nginx (or `docker compose up`).  
3. **Health**: `GET /actuator/health` on the backend (if exposed internally).  
4. **Migrations**: Flyway runs **on backend startup**; first deploy may take longer.  
5. **First administrator account**: there is **no** safe default in production—document your chosen one-time process (§7).  
6. **Persistent paths**: database volume + **`FILE_STORAGE_PATH`** for uploads.  
7. **Rollback**: keep previous JAR/image and DB backup before upgrade.

### 3.6 Optional but valuable

- Sample **nginx** server block for their subdomain (adapt from [12_DEPLOYMENT.md](./12_DEPLOYMENT.md)).  
- **`docker-compose`** override file with `SPRING_PROFILES_ACTIVE=prod` and production env vars (no real passwords committed).  
- TLS: point admin to Let’s Encrypt / corporate CA process.

---

## 4) What the server administrator does (high-level)

1. **Provision PostgreSQL** — create database and user; restrict network access.  
2. **Set secrets** — `JWT_SECRET`, DB password, etc. (never commit real values).  
3. **Run backend** with `SPRING_PROFILES_ACTIVE=prod` and **`APP_DATA_INIT_ENABLED=false`**.  
4. **Deploy frontend** static files **or** frontend container.  
5. **Configure reverse proxy** (nginx): HTTPS, proxy `/api/` to backend, correct **forwarded** headers.  
6. **Verify** — open the site, login (after first admin exists), upload a small test file, confirm it persists after restart.  
7. **Backups** — schedule DB dumps and backup of `FILE_STORAGE_PATH`.  
8. **Monitoring** — disk space, DB, app logs, certificate expiry.

Detailed env vars: [15_BEGINNER_START_AND_DEPLOY.md §6.3](./15_BEGINNER_START_AND_DEPLOY.md#63-important-backend-environment-variables).  
Security checklist: [PRODUCTION_CHECKLIST.md](../PRODUCTION_CHECKLIST.md).

---

## 5) Subdomains: recommended layout

### 5.1 Recommended: single public hostname (simplest)

Example: **`https://campus.university.edu`**

- DNS: `campus.university.edu` → your server or load balancer.  
- **nginx** (or equivalent):
  - Serves the **Angular** static files for `/`.
  - Proxies **`/api/`** to the Spring Boot process (e.g. `http://127.0.0.1:8080`).
- **No frontend rebuild** needed: `environment.prod.ts` already uses **`apiUrl: '/api/v1'`**.

This matches the architecture described in [15_BEGINNER_START_AND_DEPLOY.md](./15_BEGINNER_START_AND_DEPLOY.md) (same origin, nginx proxies `/api`).

### 5.2 Alternative: separate app and API hostnames

Example: **`app.university.edu`** (UI) and **`api.university.edu`** (API).

- Requires a **new frontend build** with `apiUrl: 'https://api.university.edu/api/v1'` (or your path convention).  
- Backend **CORS** must allow `https://app.university.edu` (`APP_SECURITY_ALLOWED_ORIGINS_0`, etc.).  
- Cookies / CSRF behaviour may need extra review ([11_SECURITY.md](./11_SECURITY.md)).

Deliver this layout **only** if the admin explicitly requests split domains; document the exact URLs in the runbook.

### 5.3 Internal-only backend

Many teams keep the JAR listening on **localhost:8080** and never expose 8080 publicly—only nginx talks to it. That is the preferred pattern.

---

## 6) Environment variables reference (backend)

Values are read from the environment (Spring Boot relaxed binding). Typical production set:

| Purpose | Examples |
|---------|----------|
| Active profile | `SPRING_PROFILES_ACTIVE=prod` |
| Database | `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD` — *or* `SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD` (Docker Compose style) |
| JWT | `JWT_SECRET` (strong, random, ≥ 256-bit equivalent; see PRODUCTION_CHECKLIST) |
| Upload storage | `FILE_STORAGE_PATH` (persistent disk) |
| Demo / seed data | **`APP_DATA_INIT_ENABLED=false`** (do not enable on real servers) |
| CORS | `APP_SECURITY_ALLOWED_ORIGINS_0`, `_1`, … if the browser origin differs from API origin |

Optional HTTPS-related tuning: see `application-prod.yaml` (`secure-cookies`, `require-https` when TLS terminates on the JVM).

---

## 7) First administrator account (production)

With **`APP_DATA_INIT_ENABLED=false`**, the app **does not** create `admin` / `password123`.

The **release runbook** must state one of:

- **Staging bootstrap**: once on an isolated environment, use **`dev`** profile only to create users, then change passwords and **never** use `dev` on production; **or**  
- **Database procedure**: insert/bootstrap first `ADMIN` with a proper **BCrypt** hash (advanced; schema-aware); **or**  
- A **future** first-run tool if your organization adds one.

Until one admin exists, self-service registration (if enabled) may still require an admin to approve accounts—plan this before go-live.

---

## 8) Quick verification after deploy

- [ ] UI loads over **HTTPS** without mixed-content errors.  
- [ ] Login works; session cookies set with appropriate **Secure** flag in HTTPS.  
- [ ] A test API call from the browser hits **`/api/v1/...`** and returns expected JSON.  
- [ ] File upload (if used) survives **backend restart** (volume still mounted).  
- [ ] PostgreSQL data survives container restart (`postgres` volume or managed DB).  
- [ ] `APP_DATA_INIT_ENABLED` is **not** true in production.

---

## 9) Document change log

| Version | Notes |
|---------|--------|
| 1.0 | Initial production handoff & doc map |

When you change deploy steps or env vars in code, update this file and **15_BEGINNER_START_AND_DEPLOY.md** in the same release.
