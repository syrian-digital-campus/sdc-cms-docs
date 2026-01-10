# Complete Setup Guide

This guide will walk you through setting up the entire SDC-CMS development environment from scratch.

## Prerequisites

### Required Software

1. **Docker Desktop** (or Docker Engine + Docker Compose)
   - Download: https://www.docker.com/products/docker-desktop
   - Version: 20.10+ recommended
   - Why: Runs PostgreSQL database and optionally the full stack

2. **Java Development Kit (JDK) 17**
   - Download: https://adoptium.net/ (Temurin recommended)
   - Why: Required to build and run Spring Boot backend
   - Verify: `java -version` should show Java 17+

3. **Maven 3.6+** (or use Maven Wrapper included)
   - Download: https://maven.apache.org/
   - Why: Build tool for Java/Spring Boot
   - Verify: `mvn -version`

4. **Node.js 18+ and npm**
   - Download: https://nodejs.org/
   - Why: Required to build and run Angular frontend
   - Verify: `node -v` and `npm -v`

5. **Git**
   - Download: https://git-scm.com/
   - Why: Version control
   - Verify: `git --version`

### Optional but Recommended

- **IDE**: IntelliJ IDEA (backend), VS Code or WebStorm (frontend)
- **PostgreSQL Client**: pgAdmin or DBeaver (for database inspection)
- **Postman/Insomnia**: For API testing

## Step 1: Clone Repositories

### Create Workspace Directory

```bash
# Windows
mkdir C:\syrian-digital-campus
cd C:\syrian-digital-campus

# Linux/Mac
mkdir ~/syrian-digital-campus
cd ~/syrian-digital-campus
```

### Clone All Repositories

```bash
# Clone backend repository
git clone <backend-repo-url> sdc-cms-backend

# Clone frontend repository
git clone <frontend-repo-url> sdc-cms-frontend

# Clone infrastructure repository
git clone <infra-repo-url> sdc-cms-infra

# Clone documentation repository (optional, for reference)
git clone <docs-repo-url> sdc-cms-docs
```

Your directory structure should look like:

```
syrian-digital-campus/
├── sdc-cms-backend/
├── sdc-cms-frontend/
├── sdc-cms-infra/
└── sdc-cms-docs/
```

## Step 2: Database Setup

### Start PostgreSQL with Docker

**Important**: PostgreSQL runs **ONLY** via Docker. Do NOT install it locally.

```bash
# Navigate to infrastructure directory
cd sdc-cms-infra

# Start PostgreSQL container
docker compose up -d postgres

# Verify it's running
docker compose ps

# You should see:
# sdc-cms-postgres   Up   0.0.0.0:5432->5432/tcp
```

### Verify Database Connection

```bash
# Check logs
docker compose logs postgres

# Connect to database (optional)
docker exec -it sdc-cms-postgres psql -U postgres -d university_cms

# In psql, list tables (should be empty initially)
\dt

# Exit psql
\q
```

### Database Configuration

The database uses these defaults:
- **Host**: `localhost`
- **Port**: `5432`
- **Database**: `university_cms`
- **Username**: `postgres`
- **Password**: `postgres`

To customize, edit `sdc-cms-infra/docker-compose.yml` or create `.env` file:

```bash
# In sdc-cms-infra/
cat > .env << EOF
POSTGRES_DB=university_cms
POSTGRES_USER=postgres
POSTGRES_PASSWORD=your_password_here
POSTGRES_PORT=5432
EOF
```

**Important**: If you change the password, also update `sdc-cms-backend/src/main/resources/application.yaml`:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/university_cms
    username: postgres
    password: your_password_here  # Update this
```

## Step 3: Backend Setup

### Navigate to Backend Directory

```bash
cd sdc-cms-backend
```

### Configure Application Properties

Edit `src/main/resources/application.yaml` if you changed database credentials:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/university_cms
    username: postgres  # Change if needed
    password: postgres  # Change if needed
```

### Build the Project

```bash
# Using Maven Wrapper (recommended, no installation needed)
./mvnw clean install

# Windows
mvnw.cmd clean install

# Or using installed Maven
mvn clean install
```

This will:
- Download dependencies
- Compile Java code
- Run tests
- Package into JAR file

### Run Database Migrations

Migrations run automatically when the application starts, but you can verify:

```bash
# Check migration files
ls src/main/resources/db/migration/

# Migrations will be applied automatically on startup
```

### Start the Backend

```bash
# Using Maven
./mvnw spring-boot:run

# Windows
mvnw.cmd spring-boot:run

# Or run the JAR
java -jar target/cms-0.0.1-SNAPSHOT.jar
```

The backend should start on `http://localhost:8080`

### Verify Backend is Running

```bash
# Check health endpoint (if actuator is enabled)
curl http://localhost:8080/actuator/health

# Or visit in browser
# http://localhost:8080/actuator/health
```

### Create Test User (Optional)

The backend includes a `DataInitializer` that creates test users automatically on first run:
- **Admin**: username `admin`, password `password123`
- **Professors**: `cremers`, `stock`, `fritz` (password: `password123`)
- **Students**: `student1`, `student2`, `student3`, `student4` (password: `password123`)

**⚠️ Important**: These test users are for development only. They should be removed or disabled in production.

## Step 4: Frontend Setup

### Navigate to Frontend Directory

```bash
cd sdc-cms-frontend
```

### Install Dependencies

```bash
npm install
```

This downloads all required packages (Angular, RxJS, etc.)

### Configure API URL

The frontend is pre-configured to use `http://localhost:8080/api/v1` in `src/environments/environment.ts`. If your backend runs on a different port, update it:

```typescript
export const environment = {
  production: false,
  apiUrl: 'http://localhost:8080/api/v1'  // Change if needed
};
```

### Start Development Server

```bash
npm start
```

The frontend will be available at `http://localhost:4200`

Angular's development server supports:
- **Hot Reload**: Changes automatically refresh in browser
- **Source Maps**: Easy debugging
- **Error Overlay**: Errors displayed in browser

### Verify Frontend is Running

Open your browser and navigate to:
```
http://localhost:4200
```

You should see the SDC-CMS login page.

## Step 5: Full Stack Setup (Docker Compose)

### Alternative: Run Everything with Docker

If you prefer to run everything in Docker:

```bash
cd sdc-cms-infra

# Build and start all services
docker compose up -d

# View logs
docker compose logs -f

# Check status
docker compose ps

# Stop all services
docker compose down

# Stop and remove volumes (⚠️ deletes data)
docker compose down -v
```

This will start:
- PostgreSQL on port 5432
- Backend API on port 8080
- Frontend on port 4200

### Access Points

- **Frontend**: http://localhost:4200
- **Backend API**: http://localhost:8080
- **PostgreSQL**: localhost:5432

## Step 6: Verify Complete Setup

### 1. Check All Services

```bash
# Database
docker compose ps postgres  # Should be "Up"

# Backend (if running locally)
curl http://localhost:8080/actuator/health

# Frontend (if running locally)
curl http://localhost:4200
```

### 2. Test Login

1. Open browser: http://localhost:4200
2. Click "Login"
3. Enter credentials:
   - Username: `student1`
   - Password: `password123`
4. You should be redirected to the courses page

### 3. Test API

```bash
# Login via API
curl -X POST http://localhost:8080/api/v1/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"student1","password":"password123"}'

# Should return access token and user info
```

## Common Issues and Solutions

### Issue: "Connection to localhost:5432 refused"

**Problem**: Database is not running.

**Solution**:
```bash
cd sdc-cms-infra
docker compose up -d postgres
docker compose ps  # Verify it's running
```

### Issue: "Port 8080 already in use"

**Problem**: Another application is using port 8080.

**Solution**:
1. Find what's using the port:
   ```bash
   # Windows
   netstat -ano | findstr :8080
   
   # Linux/Mac
   lsof -i :8080
   ```

2. Either:
   - Stop the conflicting application, OR
   - Change backend port in `application.yaml`:
     ```yaml
     server:
       port: 8081
     ```

### Issue: "npm install fails"

**Problem**: Node.js version too old or network issues.

**Solution**:
```bash
# Check Node version (should be 18+)
node -v

# Clear npm cache
npm cache clean --force

# Delete node_modules and reinstall
rm -rf node_modules package-lock.json
npm install
```

### Issue: "Maven build fails"

**Problem**: Java version mismatch or Maven issues.

**Solution**:
```bash
# Verify Java version (should be 17+)
java -version

# Clear Maven cache
./mvnw clean

# Delete target directory and rebuild
rm -rf target
./mvnw clean install
```

### Issue: "Database migrations fail"

**Problem**: Database schema conflicts or connection issues.

**Solution**:
```bash
# Check database connection
docker exec -it sdc-cms-postgres psql -U postgres -d university_cms

# Check migration status in backend logs
# Look for Flyway migration messages

# If needed, reset database (⚠️ deletes all data)
docker compose down -v
docker compose up -d postgres
# Restart backend to run migrations
```

### Issue: "Frontend can't connect to backend"

**Problem**: CORS issues or wrong API URL.

**Solution**:
1. Verify backend is running: `curl http://localhost:8080/actuator/health`
2. Check browser console for CORS errors
3. Verify API URL in `environment.ts` matches backend URL
4. Check backend CORS configuration in `SecurityConfig.java`

## Development Workflow

### Recommended Development Setup

For active development, run services separately for hot-reload:

```bash
# Terminal 1: Database
cd sdc-cms-infra
docker compose up -d postgres

# Terminal 2: Backend (with hot-reload via Spring DevTools)
cd sdc-cms-backend
./mvnw spring-boot:run

# Terminal 3: Frontend (with hot-reload)
cd sdc-cms-frontend
npm start
```

### File Structure Quick Reference

```
Backend:
- Source code: src/main/java/
- Configuration: src/main/resources/application.yaml
- Database migrations: src/main/resources/db/migration/
- Tests: src/test/java/

Frontend:
- Source code: src/app/
- Configuration: src/environments/
- Assets: src/assets/
- Tests: src/app/.../*.spec.ts
```

## Next Steps

Now that your environment is set up:

1. **[07_DEVELOPMENT_GUIDE.md](./07_DEVELOPMENT_GUIDE.md)** - Learn the development workflow
2. **[03_TECHNOLOGY_STACK.md](./03_TECHNOLOGY_STACK.md)** - Understand the technologies
3. **[08_API_DOCUMENTATION.md](./08_API_DOCUMENTATION.md)** - Explore the API

## Additional Resources

- **[13_TROUBLESHOOTING.md](./13_TROUBLESHOOTING.md)** - More troubleshooting tips
- **[06_POSTGRESQL_GUIDE.md](./06_POSTGRESQL_GUIDE.md)** - PostgreSQL basics
- **[04_SPRING_BOOT_GUIDE.md](./04_SPRING_BOOT_GUIDE.md)** - Spring Boot for beginners
- **[05_ANGULAR_GUIDE.md](./05_ANGULAR_GUIDE.md)** - Angular for beginners

---

**Document Version**: 1.0  
**Last Updated**: December 2024




