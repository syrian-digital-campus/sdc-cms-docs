# Beginner guide: receive the project, run locally, deploy

Read in order; each section builds on the previous one.

**Production deploy & handoff (developer → server admin, subdomains, deliverables):** [16_PRODUCTION_DEPLOY_AND_HANDOFF.md](./16_PRODUCTION_DEPLOY_AND_HANDOFF.md)

---

## 1) What is in this repo?

| Part | Folder | Role |
|------|--------|------|
| **Frontend** | `sdc-cms-frontend` | Angular app in the browser |
| **Backend** | `sdc-cms-backend` | REST API (Spring Boot), security, file storage |
| **Infrastructure** | `sdc-cms-infra` | Docker Compose: database and sometimes the full stack |
| **Docs** | `sdc-cms-docs` | Extra guides and technical detail |

The frontend talks to the API under `/api/v1`. In production, **nginx** usually proxies `/api` to the backend.

---

## 2) Getting the project (“delivery”)

### 2.1 One-time prerequisites on your machine

Install (versions are approximate; use current stable):

1. **Git** — clone the repo  
2. **JDK 17+** (e.g. Temurin from [adoptium.net](https://adoptium.net/)) — backend  
3. **Maven 3.9+** — build backend (`mvn`)  
4. **Node.js 18+ and npm** — frontend  
5. **Docker Desktop** (or Docker Engine + Compose) — **recommended** for PostgreSQL  

Check in a terminal:

```bash
git --version
java -version
mvn -version
node -v
npm -v
docker --version
docker compose version
```

### 2.2 Clone the code

**Monorepo** (everything in one repo):

```bash
git clone <repo-url> syrian-digital-campus
cd syrian-digital-campus
```

If the project is split across several repos, clone each into one workspace as in [02_SETUP_GUIDE.md](./02_SETUP_GUIDE.md).

### 2.3 After you clone

1. Open the folder in your **IDE** (IntelliJ, VS Code, Cursor, …).  
2. Skim **[00_INTRODUCTION.md](./00_INTRODUCTION.md)** and **[01_ARCHITECTURE.md](./01_ARCHITECTURE.md)** (about 15–30 minutes).  
3. Follow one path in section 3.

---

## 3) Local run and daily work

Two common paths.

### Path A — Fastest try: everything with Docker

Good if you want the stack without installing JDK/Maven/Node for each service (depends on your `docker-compose` setup).

```bash
cd sdc-cms-infra
docker compose build
docker compose up -d
```

Then open the UI (often `http://localhost:4200`) and API (`http://localhost:8080`). More detail: **[QUICKSTART.md](../QUICKSTART.md)** and **[sdc-cms-infra/README.md](../sdc-cms-infra/README.md)**.

**Note:** Uploads (materials, assignments) live on the **backend disk**. If you wipe Docker volumes (`docker compose down -v`), files can disappear while the DB still lists old paths — re-upload from the UI as needed.

---

### Path B — Typical for developers: Postgres in Docker, backend + frontend on the host

#### Step 1: Start PostgreSQL only

```bash
cd sdc-cms-infra
docker compose up -d postgres
```

Match DB name, user, and password to the backend `application.yaml` (defaults are often user `postgres`, database `university_cms`, port `5432`). Check `sdc-cms-infra/.env.example` if present.

#### Step 2: Run the backend

```bash
cd sdc-cms-backend
```

**Development with seed data (admin and other test users):**

- **Where?** Folder **`sdc-cms-backend`** (the directory that contains **`pom.xml`**), in a **terminal on your machine** (PowerShell on Windows, Terminal on Linux/macOS). You are **not** inside a Docker shell unless you chose to run Maven there.  
- **What to type?** Set the variable, then run Maven **from that folder**:

```bash
# Windows PowerShell (current directory: ...\sdc-cms-backend)
$env:SPRING_PROFILES_ACTIVE="dev"
mvn spring-boot:run

# Linux / macOS (current directory: .../sdc-cms-backend)
export SPRING_PROFILES_ACTIVE=dev
mvn spring-boot:run
```

Without the **`dev`** profile, **no** demo users/courses are created automatically (`app.data.init.enabled` defaults to `false` for production safety).

**Run on a server (production — short version):** Do **not** enable `dev` or `APP_DATA_INIT_ENABLED`. Copy the JAR after `mvn package` (e.g. `target/cms-1.0.0.jar`) to the server, then for example on Linux:

```bash
export SPRING_PROFILES_ACTIVE=prod
export DB_HOST=localhost
export DB_PORT=5432
export DB_NAME=university_cms
export DB_USER=postgres
export DB_PASSWORD='your-db-password'
export JWT_SECRET='your-long-random-secret'
export FILE_STORAGE_PATH=/var/sdc/files
# If the UI is at https://example.com:
export APP_SECURITY_ALLOWED_ORIGINS_0=https://example.com

java -jar cms-1.0.0.jar
```

PostgreSQL must be running and reachable from the server at `DB_HOST` **before** you start the JAR.

**Build checks:**

```bash
mvn test
mvn package -DskipTests
```

The JAR is under `target/` (exact name from `pom.xml`).

#### Step 3: Run the frontend

```bash
cd sdc-cms-frontend
npm install
npm start
# or: npx ng serve
```

Default URL is often `http://localhost:4200`; dev API URL is in `environment.ts` (`http://localhost:8080/api/v1`).

#### Step 4: Log in for testing

With **`spring.profiles.active=dev`**, see **[TEST_ACCOUNTS_QUICK_REFERENCE.md](../TEST_ACCOUNTS_QUICK_REFERENCE.md)** (e.g. `admin` / `password123` — **never use these in production**).

---

## 4) Handing off your work (short Git workflow)

1. Branch from `main` / `master`: `git checkout -b feature/short-description`  
2. Change code and test locally.  
3. `git add` and `git commit` with a clear message.  
4. `git push` and open a **Pull Request**.

More: **[07_DEVELOPMENT_GUIDE.md](./07_DEVELOPMENT_GUIDE.md)**.

---

## 5) Production build (before deploy)

### Frontend (Angular)

```bash
cd sdc-cms-frontend
npm ci
npm run build
```

- Output: `dist/sdc-cms-frontend/`.  
- **Production** build swaps in **`environment.prod.ts`** with `apiUrl: '/api/v1'` — the browser and API must be on the **same origin**, with nginx proxying `/api` to Spring Boot.

### Backend (Spring Boot)

```bash
cd sdc-cms-backend
mvn package -DskipTests
```

- Run the JAR from `target/*.jar`.  
- Do not rely on **spring-boot-devtools** in production (it should be excluded from the repackaged JAR).

---

## 6) Production deployment

### 6.1 Quick security checklist

- [ ] **`JWT_SECRET`**: strong, long random value (not the placeholder in repo config).  
- [ ] **PostgreSQL password** strong; avoid exposing `postgres` user on the public internet.  
- [ ] **`APP_DATA_INIT_ENABLED`**: keep **`false`** — no demo accounts on a real server.  
- [ ] **HTTPS** in front of users (TLS at nginx or your cloud load balancer).  
- [ ] **CORS**: only your real frontend origins (e.g. `APP_SECURITY_ALLOWED_ORIGINS_0`, …).  
- [ ] **Upload storage**: persistent disk or object storage with backups.  
- [ ] **Database backups** on a schedule.

Also read: **[12_DEPLOYMENT.md](./12_DEPLOYMENT.md)**, **[11_SECURITY.md](./11_SECURITY.md)**, **[PRODUCTION_CHECKLIST.md](../PRODUCTION_CHECKLIST.md)**.

### 6.2 `prod` profile for the backend

```text
--spring.profiles.active=prod
```

or environment:

```text
SPRING_PROFILES_ACTIVE=prod
```

**`application-prod.yaml`** tightens logging and sets **`secure-cookies: true`**. Read comments there for **HTTPS** / **require-https** depending on whether TLS terminates on the app or on nginx.

### 6.3 Important backend environment variables

Values come from `application.yaml` with overrides via the environment (Spring Boot relaxed binding applies):

| Purpose | Example env vars |
|---------|-------------------|
| Database | `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER`, `DB_PASSWORD` |
| JWT | `JWT_SECRET` |
| File uploads | `FILE_STORAGE_PATH` |
| Demo seed data | `APP_DATA_INIT_ENABLED=false` |
| CORS | `APP_SECURITY_ALLOWED_ORIGINS_0`, `_1`, … |

### 6.4 Database and Flyway

On startup, **Flyway** runs migrations automatically (**`spring-boot-starter-flyway`**). In production, do not drop tables by hand; use scripts under `src/main/resources/db/migration/`.

### 6.5 nginx (common layout)

1. Serve Angular static files from `dist/`.  
2. Proxy **`/api/`** to Spring Boot (e.g. `http://127.0.0.1:8080`).  
3. Forward **`X-Forwarded-Proto`** and **`Host`** if TLS is on nginx.

Starter example: **[12_DEPLOYMENT.md](./12_DEPLOYMENT.md)**.

### 6.6 Docker in production

- Pass the env vars above into the **backend** service.  
- Mount a **volume** for `FILE_STORAGE_PATH` inside the container.  
- Do not bake **secrets** into images; use your platform’s secret store or a protected `.env` on the server only.

---

## 7) Read next

| Topic | File |
|--------|------|
| Detailed setup | [02_SETUP_GUIDE.md](./02_SETUP_GUIDE.md) |
| API | [08_API_DOCUMENTATION.md](./08_API_DOCUMENTATION.md) |
| Troubleshooting | [13_TROUBLESHOOTING.md](./13_TROUBLESHOOTING.md) |
| Docker quick start | [QUICKSTART.md](../QUICKSTART.md) |

---

## 8) Tomorrow morning cheat sheet

1. `docker compose up -d postgres` in `sdc-cms-infra`  
2. `SPRING_PROFILES_ACTIVE=dev` + `mvn spring-boot:run` in `sdc-cms-backend`  
3. `npm install` + `npm start` in `sdc-cms-frontend`  
4. Open the browser, log in, try a small change  
5. Before production: `npm run build` + `mvn package` + production env vars + HTTPS + backups  

---

## 9) Deeper explanation (why these pieces exist)

**What `mvn spring-boot:run` does**  
Maven compiles (if needed) and starts the Spring Boot app on your machine. It listens on port **8080** by default (unless you changed `server.port`). The app connects to PostgreSQL using `DB_*` (or defaults in `application.yaml`).

**Why `SPRING_PROFILES_ACTIVE=dev`**  
The **`dev`** profile loads **`application-dev.yaml`**, which sets `app.data.init.enabled=true` so the **`DataInitializer`** can create demo users and sample data. Without **`dev`**, that initializer is skipped so a blank production database never gets accidental `admin` / `password123`.

**Why the backend folder matters**  
Maven reads **`pom.xml`** from the **current working directory**. If you run `mvn` from the repo root or from `sdc-cms-frontend`, it will not find the backend project unless you `cd sdc-cms-backend` first.

**Environment variables on the server**  
Spring Boot maps `DB_HOST` → `spring.datasource` pieces via placeholders in `application.yaml`. You do not commit real passwords; you set them on the server only. **`JWT_SECRET`** must be unpredictable or anyone could forge tokens.

**Frontend vs backend in production**  
The production Angular build calls **`/api/v1`** (relative URL). The browser sends that to **the same host** that served the HTML (e.g. `https://campus.example.com/api/v1/...`). **nginx** must forward `/api` to the JVM on `localhost:8080` (or another internal address).

**Flyway**  
Runs SQL migrations in order so the schema always matches the code. It runs when the app starts, before your business logic serves traffic (with the current project setup).

---

## 10) Delivering the app **without** GitHub access

You do **not** have to give anyone your Git repository. Ship **built artifacts** and **run instructions** instead.

### What recipients need

- **Backend:** one executable JAR (from `mvn package` in `sdc-cms-backend`), e.g. `cms-1.0.0.jar`.  
- **Frontend:** the contents of `dist/sdc-cms-frontend/` after `npm run build` (static HTML/JS/CSS).  
- **Database:** PostgreSQL 14+ (they install it, or you include it via Docker Compose **without** your source tree).  
- **Docs:** a short `README-INSTALL.txt` or PDF: Java version, env vars (`DB_*`, `JWT_SECRET`, `FILE_STORAGE_PATH`, `SPRING_PROFILES_ACTIVE=prod`, CORS), how to run `java -jar`, and how to point nginx at the `dist` folder and `/api`.

### What to put in a release ZIP (example)

```
sdc-cms-release/
  backend/cms-1.0.0.jar
  frontend/          ← copy of dist/sdc-cms-frontend/*
  docker-compose.yml + .env.example   ← optional, if you use Compose for Postgres only
  README-INSTALL.md
```

Do **not** include: `.git/`, real `.env` with secrets, or `SPRING_PROFILES_ACTIVE=dev` / demo passwords for production handoff.

### Other delivery options

- **Docker images:** build images locally, `docker save` → `.tar`, send the tar; they `docker load` and run with env vars (still no Git).  
- **Private registry:** push images to a registry they can pull from (account you control, not your GitHub).  
- **USB / shared drive / encrypted cloud link:** same ZIP or image tar — access to the file is not access to your repo.

### License and updates

If you give only binaries, you still owe them whatever license terms apply to your stack (e.g. open-source dependencies). For **updates**, you send a **new** ZIP or new image version when you release — again without opening the repo.

---

*Last updated: April 2026 — matches current project settings (Spring Boot 4, Flyway via starter, demo data off by default).*
