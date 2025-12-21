# PostgreSQL Guide

This guide covers PostgreSQL basics and how it's used in SDC-CMS.

## What is PostgreSQL?

PostgreSQL is a powerful, open-source relational database management system (RDBMS). It's:
- **ACID Compliant**: Ensures data reliability
- **Feature Rich**: Advanced features like JSON support, full-text search
- **Open Source**: Free and actively maintained
- **Standards Compliant**: Follows SQL standards

## Basic Concepts

### Databases and Tables

**Database**: Container for related data (e.g., `university_cms`)

**Table**: Structure that stores data in rows and columns
```sql
-- Example: users table
CREATE TABLE users (
    id UUID PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

**Row**: A single record (e.g., one user)
**Column**: A field in a row (e.g., username, email)

### Data Types

**Common Types in SDC-CMS**:

- **UUID**: Universally Unique Identifier (e.g., `123e4567-e89b-12d3-a456-426614174000`)
- **VARCHAR(n)**: Variable-length string (max n characters)
- **TEXT**: Unlimited length string
- **INTEGER**: Whole numbers
- **BIGINT**: Large whole numbers
- **DECIMAL(p, s)**: Fixed-point numbers (p = precision, s = scale)
- **BOOLEAN**: True/false
- **TIMESTAMP**: Date and time
- **DATE**: Date only

### Primary Keys

**What is a Primary Key?**
Uniquely identifies each row in a table.

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,  -- Primary key
    username VARCHAR(50)
);
```

**Why UUID?**
- Globally unique (good for distributed systems)
- No sequence needed (can generate in application)
- Better for security (not predictable)

### Foreign Keys

**What is a Foreign Key?**
References a primary key in another table. Creates a relationship.

```sql
CREATE TABLE courses (
    id UUID PRIMARY KEY,
    owner_user_id UUID NOT NULL,
    FOREIGN KEY (owner_user_id) REFERENCES users(id)  -- Foreign key
);
```

### Constraints

**Common Constraints**:

- **NOT NULL**: Column cannot be null
- **UNIQUE**: Values must be unique
- **PRIMARY KEY**: Unique identifier (implies NOT NULL and UNIQUE)
- **FOREIGN KEY**: References another table
- **CHECK**: Custom validation condition

```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) NOT NULL,
    age INTEGER CHECK (age >= 0)  -- Must be non-negative
);
```

### Indexes

**What is an Index?**
Improves query performance by creating a data structure that speeds up lookups.

```sql
-- Create index on username (speeds up username lookups)
CREATE INDEX idx_users_username ON users(username);

-- Unique index (also enforces uniqueness)
CREATE UNIQUE INDEX idx_users_email ON users(email);
```

**When to Use Indexes**:
- Columns used in WHERE clauses frequently
- Foreign keys (automatically indexed)
- Columns used for sorting/joining

## SQL Basics

### SELECT (Query Data)

```sql
-- Select all columns
SELECT * FROM users;

-- Select specific columns
SELECT id, username, email FROM users;

-- Filter rows
SELECT * FROM users WHERE username = 'student1';

-- Multiple conditions
SELECT * FROM users WHERE username = 'student1' AND enabled = true;

-- Sort results
SELECT * FROM users ORDER BY created_at DESC;

-- Limit results
SELECT * FROM users LIMIT 10;
```

### INSERT (Add Data)

```sql
-- Insert single row
INSERT INTO users (id, username, email, password_hash)
VALUES ('123e4567-e89b-12d3-a456-426614174000', 'john', 'john@example.com', 'hash123');

-- Insert multiple rows
INSERT INTO users (id, username, email, password_hash)
VALUES
    ('111e4567-e89b-12d3-a456-426614174000', 'alice', 'alice@example.com', 'hash456'),
    ('222e4567-e89b-12d3-a456-426614174000', 'bob', 'bob@example.com', 'hash789');
```

### UPDATE (Modify Data)

```sql
-- Update specific rows
UPDATE users SET email = 'newemail@example.com' WHERE username = 'john';

-- Update multiple columns
UPDATE users SET email = 'newemail@example.com', enabled = false WHERE id = '123e4567...';
```

### DELETE (Remove Data)

```sql
-- Delete specific rows
DELETE FROM users WHERE username = 'john';

-- Delete all rows (⚠️ be careful!)
DELETE FROM users;
```

### JOIN (Combine Tables)

**INNER JOIN**: Returns rows where match exists in both tables
```sql
SELECT c.title, u.username
FROM courses c
INNER JOIN users u ON c.owner_user_id = u.id;
```

**LEFT JOIN**: Returns all rows from left table, matching rows from right
```sql
SELECT c.title, COUNT(e.id) as enrollment_count
FROM courses c
LEFT JOIN enrollments e ON c.id = e.course_id
GROUP BY c.id, c.title;
```

## Database Design in SDC-CMS

### Entity Relationship Model

```
User (1) ──────< Enrollment (Many) >────── (1) Course
                                    │
                                    ├─── Material
                                    ├─── Assignment
                                    └─── CourseNews
```

### Key Tables

**users**: User accounts
```sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    enabled BOOLEAN NOT NULL DEFAULT true,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

**user_profiles**: User profile information
```sql
CREATE TABLE user_profiles (
    user_id UUID PRIMARY KEY REFERENCES users(id),
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    global_role VARCHAR(20) NOT NULL CHECK (global_role IN ('ADMIN', 'PROFESSOR', 'STUDENT')),
    matriculation_number VARCHAR(50),
    office_location VARCHAR(255),
    language_preference VARCHAR(10)
);
```

**courses**: Course information
```sql
CREATE TABLE courses (
    id UUID PRIMARY KEY,
    title VARCHAR(500) NOT NULL,
    description TEXT,
    course_code VARCHAR(50) NOT NULL,
    semester VARCHAR(20) NOT NULL CHECK (semester IN ('WINTER', 'SUMMER')),
    year INTEGER NOT NULL,
    owner_user_id UUID NOT NULL REFERENCES users(id),
    created_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

**enrollments**: User-course relationships
```sql
CREATE TABLE enrollments (
    id UUID PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id),
    course_id UUID NOT NULL REFERENCES courses(id),
    role VARCHAR(20) NOT NULL CHECK (role IN ('PROFESSOR', 'TA', 'TUTOR', 'STUDENT')),
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    UNIQUE(user_id, course_id)  -- One enrollment per user per course
);
```

### Relationships

**One-to-Many**:
- User → Courses (one user owns many courses)
- Course → Materials (one course has many materials)

**Many-to-Many** (via join table):
- Users ↔ Courses (via enrollments table)
- Users can enroll in many courses, courses have many users

## Using pgAdmin

### Connecting to Database

1. Open pgAdmin
2. Right-click "Servers" → "Create" → "Server"
3. General tab:
   - Name: `SDC-CMS Local`
4. Connection tab:
   - Host: `localhost`
   - Port: `5432`
   - Database: `university_cms`
   - Username: `postgres`
   - Password: `postgres`
5. Click "Save"

### Viewing Data

**Browse Tables**:
1. Expand: Servers → SDC-CMS Local → Databases → university_cms → Schemas → public → Tables
2. Right-click table → "View/Edit Data" → "All Rows"

**Run Queries**:
1. Right-click database → "Query Tool"
2. Type SQL query
3. Click Execute (F5)

### Useful Queries

**View all users**:
```sql
SELECT * FROM users;
```

**Count enrollments per course**:
```sql
SELECT c.title, COUNT(e.id) as enrollment_count
FROM courses c
LEFT JOIN enrollments e ON c.id = e.course_id
GROUP BY c.id, c.title;
```

**Find courses for a user**:
```sql
SELECT c.title, e.role
FROM courses c
INNER JOIN enrollments e ON c.id = e.course_id
WHERE e.user_id = '123e4567-e89b-12d3-a456-426614174000';
```

## Database Migrations with Flyway

**What is Flyway?**
Tool that manages database schema changes in a version-controlled way.

**How It Works**:
1. SQL scripts in `src/main/resources/db/migration/`
2. Named with version: `V1__description.sql`, `V2__description.sql`
3. Flyway tracks which migrations have run
4. Applies new migrations on application startup

**Example Migration**:
```sql
-- V1__initial_schema.sql
CREATE TABLE users (
    id UUID PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    -- ...
);

-- V2__add_courses.sql
CREATE TABLE courses (
    id UUID PRIMARY KEY,
    title VARCHAR(500) NOT NULL,
    -- ...
);
```

**Best Practices**:
- Never modify existing migration files (create new ones)
- Test migrations before deploying
- Use transactions when possible
- Keep migrations small and focused

## Common Tasks

### Backup Database

```bash
# Using pg_dump
pg_dump -h localhost -U postgres -d university_cms > backup.sql

# Via Docker
docker exec sdc-cms-postgres pg_dump -U postgres university_cms > backup.sql
```

### Restore Database

```bash
# Using psql
psql -h localhost -U postgres -d university_cms < backup.sql

# Via Docker
docker exec -i sdc-cms-postgres psql -U postgres university_cms < backup.sql
```

### Reset Database (Development)

```bash
# Stop backend, then:
docker compose down -v  # Removes volumes (⚠️ deletes all data)
docker compose up -d postgres  # Recreate database
# Start backend (migrations will run automatically)
```

### Check Database Size

```sql
SELECT pg_size_pretty(pg_database_size('university_cms'));
```

### List All Tables

```sql
SELECT table_name 
FROM information_schema.tables 
WHERE table_schema = 'public';
```

## Performance Tips

1. **Use Indexes**: On frequently queried columns
2. **Avoid SELECT ***: Select only needed columns
3. **Use LIMIT**: When you don't need all results
4. **Use EXPLAIN**: To analyze query performance
   ```sql
   EXPLAIN ANALYZE SELECT * FROM users WHERE username = 'john';
   ```
5. **Connection Pooling**: HikariCP (already configured in Spring Boot)

## Security Considerations

1. **Never store passwords**: Store password hashes (BCrypt)
2. **Use parameterized queries**: Prevents SQL injection (JPA does this automatically)
3. **Limit database access**: Only necessary users have access
4. **Regular backups**: In case of data loss
5. **Use SSL**: For production connections

## Next Steps

- Practice with basic SQL queries
- Explore SDC-CMS database schema
- Read PostgreSQL documentation: https://www.postgresql.org/docs/
- Use pgAdmin to visualize data
- Read [07_DEVELOPMENT_GUIDE.md](./07_DEVELOPMENT_GUIDE.md) for development workflows

---

**Document Version**: 1.0  
**Last Updated**: December 2024

