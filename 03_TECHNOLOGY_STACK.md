# Technology Stack

This document provides a comprehensive overview of all technologies, frameworks, and tools used in SDC-CMS.

## Overview

SDC-CMS uses a modern, enterprise-grade technology stack:

- **Frontend**: Angular 21 (TypeScript)
- **Backend**: Spring Boot 4.0.1 (Java 17)
- **Database**: PostgreSQL 16
- **Containerization**: Docker & Docker Compose
- **Web Server**: Nginx (production frontend)

## Frontend Technologies

### Core Framework: Angular 21

**What is Angular?**
Angular is a TypeScript-based web framework for building single-page applications (SPAs). It provides a component-based architecture, dependency injection, and powerful tools for building complex user interfaces.

**Why Angular?**
- **Type Safety**: TypeScript catches errors at compile time
- **Component Architecture**: Reusable, maintainable components
- **Rich Ecosystem**: Extensive libraries and tools
- **Enterprise Ready**: Used by large organizations
- **RxJS Integration**: Powerful reactive programming

**Key Concepts**:
- **Components**: Building blocks of Angular apps (HTML + TypeScript + CSS)
- **Services**: Singleton classes for shared logic and data
- **Modules**: Organize code into feature modules
- **Dependency Injection**: Automatic dependency management
- **RxJS Observables**: Handle asynchronous operations

**Learning Resources**:
- Official Docs: https://angular.dev
- Tutorial: https://angular.dev/tutorials/first-app

### TypeScript

**What is TypeScript?**
TypeScript is JavaScript with static type checking. It compiles to JavaScript but adds type safety.

**Why TypeScript?**
- Catch errors before runtime
- Better IDE support (autocomplete, refactoring)
- Self-documenting code (types serve as documentation)
- Easier refactoring

**Example**:
```typescript
// JavaScript (no types)
function add(a, b) {
  return a + b;
}

// TypeScript (with types)
function add(a: number, b: number): number {
  return a + b;
}
```

### RxJS (Reactive Extensions)

**What is RxJS?**
RxJS provides reactive programming using Observables. It's perfect for handling asynchronous operations like HTTP requests.

**Key Concepts**:
- **Observable**: Represents a stream of data
- **Observer**: Subscribes to observables
- **Operators**: Transform data (map, filter, merge, etc.)

**Example**:
```typescript
// HTTP request returns an Observable
this.http.get<User>('/api/users/1')
  .pipe(
    map(user => user.name),  // Transform data
    catchError(error => of('Error'))  // Handle errors
  )
  .subscribe(name => {
    console.log(name);  // Handle result
  });
```

### HTTP Client

Angular's built-in HTTP client for making API requests:
- Promise-like API (but uses Observables)
- Request/response interceptors
- Automatic JSON parsing
- Error handling

## Backend Technologies

### Core Framework: Spring Boot 4.0.1

**What is Spring Boot?**
Spring Boot is a framework for building Java applications. It simplifies Spring-based development by providing:
- Auto-configuration (sensible defaults)
- Embedded server (runs without external server)
- Production-ready features (metrics, health checks)

**Why Spring Boot?**
- **Rapid Development**: Get started quickly
- **Convention over Configuration**: Less boilerplate
- **Enterprise Features**: Security, transactions, caching built-in
- **Large Ecosystem**: Many Spring projects integrate seamlessly
- **Production Ready**: Health checks, metrics, etc.

**Key Concepts**:
- **Dependency Injection**: Automatic object management
- **Annotations**: Metadata-driven configuration (`@RestController`, `@Service`, etc.)
- **Auto-Configuration**: Automatically configures components
- **Starter Dependencies**: Pre-configured dependency sets

**Learning Resources**:
- Official Docs: https://spring.io/projects/spring-boot
- Guides: https://spring.io/guides

### Java 17

**What is Java 17?**
Java 17 is a Long-Term Support (LTS) version of Java. It's the language used for the backend.

**Why Java 17?**
- **LTS Version**: Long-term support until 2029
- **Modern Features**: Records, pattern matching, sealed classes
- **Performance**: Excellent performance for server applications
- **Enterprise Standard**: Widely used in enterprise applications

**Key Features Used**:
- **Records**: Simple data classes
- **Pattern Matching**: Modern switch expressions
- **Text Blocks**: Multi-line strings

### Spring Security

**What is Spring Security?**
Spring Security provides authentication and authorization for Spring applications.

**Why Spring Security?**
- **Comprehensive**: Handles authentication, authorization, CSRF, etc.
- **Flexible**: Supports multiple authentication methods
- **JWT Support**: Works well with token-based auth
- **Security Filters**: Chain of filters for security

**Key Concepts**:
- **Authentication**: Verifying who the user is
- **Authorization**: Determining what user can do
- **Security Filter Chain**: Filters requests for security
- **JWT**: JSON Web Tokens for stateless authentication

### Spring Data JPA

**What is Spring Data JPA?**
Spring Data JPA simplifies database access. It provides repositories that automatically generate queries.

**Why Spring Data JPA?**
- **Less Boilerplate**: No need to write SQL for simple queries
- **Type Safety**: Compile-time checked queries
- **Repository Pattern**: Clean data access layer
- **Automatic Query Generation**: Generate queries from method names

**Example**:
```java
// Define repository interface
interface UserRepository extends JpaRepository<User, UUID> {
    Optional<User> findByUsername(String username);
    List<User> findByGlobalRole(GlobalRole role);
}

// Use it in service
@Service
class UserService {
    private final UserRepository userRepository;
    
    User findByUsername(String username) {
        return userRepository.findByUsername(username)
            .orElseThrow(() -> new UserNotFoundException(username));
    }
}
```

### Hibernate (via JPA)

**What is Hibernate?**
Hibernate is the JPA implementation used by Spring Data JPA. It maps Java objects to database tables.

**Why Hibernate?**
- **ORM**: Object-Relational Mapping (no SQL needed for basic operations)
- **Lazy Loading**: Loads data on demand
- **Caching**: Improves performance
- **Schema Generation**: Can generate/update database schema

**Key Concepts**:
- **Entity**: Java class mapped to database table
- **Repository**: Interface for database operations
- **Relationships**: One-to-Many, Many-to-Many, etc.
- **Lazy/Eager Loading**: When to load related data

### Flyway

**What is Flyway?**
Flyway is a database migration tool. It manages database schema changes in a version-controlled way.

**Why Flyway?**
- **Version Control**: Track all schema changes
- **Reproducible**: Same schema across all environments
- **Safe**: Can rollback migrations
- **Automatic**: Runs on application startup

**How it Works**:
1. SQL scripts in `src/main/resources/db/migration/`
2. Named with version: `V1__initial_schema.sql`, `V2__add_courses.sql`
3. Flyway tracks which migrations have run
4. Applies new migrations on startup

### JWT (JSON Web Tokens)

**What is JWT?**
JWT is a standard for securely transmitting information as a JSON object. Used for authentication tokens.

**Why JWT?**
- **Stateless**: No server-side session storage needed
- **Scalable**: Works across multiple servers
- **Secure**: Signed and optionally encrypted
- **Standard**: Widely supported

**Structure**:
```
header.payload.signature
```

**Example Token**:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyfQ.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
```

### BCrypt

**What is BCrypt?**
BCrypt is a password hashing algorithm. It's designed to be slow (to prevent brute force attacks).

**Why BCrypt?**
- **Secure**: Industry standard for password hashing
- **Adaptive**: Can increase cost factor over time
- **Built-in Salt**: Automatically generates unique salts

**How it Works**:
1. User registers with password
2. Password is hashed with BCrypt
3. Hash is stored in database (not the password)
4. On login, compare hash of entered password with stored hash

### Lombok

**What is Lombok?**
Lombok is a Java library that reduces boilerplate code through annotations.

**Why Lombok?**
- **Less Code**: Generate getters, setters, constructors automatically
- **Cleaner Code**: Focus on business logic
- **Popular**: Widely used in Java projects

**Common Annotations**:
- `@Data`: Generates getters, setters, toString, equals, hashCode
- `@AllArgsConstructor`: Generates constructor with all fields
- `@NoArgsConstructor`: Generates no-args constructor
- `@Builder`: Generates builder pattern

**Example**:
```java
// Without Lombok (verbose)
public class User {
    private String username;
    private String email;
    
    public String getUsername() { return username; }
    public void setUsername(String username) { this.username = username; }
    // ... more boilerplate
}

// With Lombok (concise)
@Data
public class User {
    private String username;
    private String email;
}
```

### MapStruct (Optional)

**What is MapStruct?**
MapStruct generates mapping code between objects (e.g., Entity to DTO).

**Why MapStruct?**
- **Type Safe**: Compile-time checked mappings
- **Fast**: No runtime reflection overhead
- **Simple**: Less boilerplate than manual mapping

## Database Technologies

### PostgreSQL 16

**What is PostgreSQL?**
PostgreSQL is a powerful, open-source relational database management system.

**Why PostgreSQL?**
- **Reliable**: ACID compliant, robust
- **Feature Rich**: Advanced features (JSON support, full-text search, etc.)
- **Open Source**: Free and actively maintained
- **Performance**: Excellent performance
- **Standards Compliant**: Follows SQL standards

**Key Concepts**:
- **Tables**: Store data in rows and columns
- **Relationships**: Foreign keys link related data
- **ACID**: Atomicity, Consistency, Isolation, Durability
- **Transactions**: Group operations together
- **Indexes**: Speed up queries

**SQL Basics**:
```sql
-- Create table
CREATE TABLE users (
    id UUID PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(255) NOT NULL
);

-- Insert data
INSERT INTO users (id, username, email) 
VALUES ('123e4567-e89b-12d3-a456-426614174000', 'john', 'john@example.com');

-- Query data
SELECT * FROM users WHERE username = 'john';
```

## Infrastructure Technologies

### Docker

**What is Docker?**
Docker is a platform for containerizing applications. It packages an application and its dependencies into a container.

**Why Docker?**
- **Consistency**: Same environment everywhere (dev, staging, prod)
- **Isolation**: Applications don't interfere with each other
- **Portable**: Run anywhere Docker runs
- **Scalable**: Easy to scale containers

**Key Concepts**:
- **Image**: Template for creating containers
- **Container**: Running instance of an image
- **Dockerfile**: Instructions for building an image
- **Docker Compose**: Define and run multi-container applications

### Docker Compose

**What is Docker Compose?**
Docker Compose is a tool for defining and running multi-container Docker applications.

**Why Docker Compose?**
- **Orchestration**: Manage multiple containers together
- **Configuration**: Define services in YAML file
- **Networking**: Automatic networking between containers
- **Volume Management**: Manage data persistence

**Example**:
```yaml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: university_cms
    ports:
      - "5432:5432"
```

### Nginx

**What is Nginx?**
Nginx is a web server that can also act as a reverse proxy and load balancer.

**Why Nginx?**
- **Performance**: Very fast and efficient
- **Reverse Proxy**: Route requests to backend
- **Static Files**: Serve Angular static files
- **SSL/TLS**: Handle HTTPS termination

**Use Cases in SDC-CMS**:
- Serve Angular frontend (static files)
- Proxy API requests to Spring Boot backend
- Handle routing for Angular SPA

## Build Tools

### Maven

**What is Maven?**
Maven is a build tool and dependency manager for Java projects.

**Why Maven?**
- **Dependency Management**: Automatically downloads dependencies
- **Standard Structure**: Conventions for project structure
- **Plugins**: Extensible with plugins
- **Lifecycle**: Standard build lifecycle (compile, test, package, etc.)

**Key Concepts**:
- **pom.xml**: Project Object Model (project configuration)
- **Dependencies**: External libraries the project uses
- **Plugins**: Tools for building, testing, etc.
- **Lifecycle**: Phases (validate, compile, test, package, install, deploy)

### npm

**What is npm?**
npm is the package manager for Node.js. It manages JavaScript dependencies.

**Why npm?**
- **Dependency Management**: Install and manage packages
- **Scripts**: Define custom scripts (build, test, start)
- **Registry**: Access to millions of packages

**Key Concepts**:
- **package.json**: Project configuration and dependencies
- **node_modules/**: Directory containing installed packages
- **npm install**: Install dependencies
- **npm scripts**: Run custom commands (npm start, npm build)

## Development Tools

### Git

**What is Git?**
Git is a version control system for tracking changes in code.

**Why Git?**
- **Version Control**: Track all changes
- **Collaboration**: Multiple developers can work together
- **Branching**: Work on features independently
- **History**: See what changed and when

**Key Concepts**:
- **Repository**: Project folder with version control
- **Commit**: Snapshot of changes
- **Branch**: Independent line of development
- **Merge**: Combine changes from branches

## Next Steps

Now that you understand the technology stack:

1. **[04_SPRING_BOOT_GUIDE.md](./04_SPRING_BOOT_GUIDE.md)** - Deep dive into Spring Boot
2. **[05_ANGULAR_GUIDE.md](./05_ANGULAR_GUIDE.md)** - Deep dive into Angular
3. **[06_POSTGRESQL_GUIDE.md](./06_POSTGRESQL_GUIDE.md)** - Deep dive into PostgreSQL
4. **[07_DEVELOPMENT_GUIDE.md](./07_DEVELOPMENT_GUIDE.md)** - Start developing

---

**Document Version**: 1.0  
**Last Updated**: December 2024




