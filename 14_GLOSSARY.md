# Glossary

Terminology and definitions used in SDC-CMS.

## General Terms

**CMS**: Course Management System - software for managing courses and educational content.

**SDC-CMS**: Syrian Digital Campus Course Management System - this project.

**SPA**: Single-Page Application - web application that loads once and updates content dynamically.

**API**: Application Programming Interface - set of endpoints for communication between frontend and backend.

**REST**: Representational State Transfer - architectural style for web APIs.

**JSON**: JavaScript Object Notation - data format used for API communication.

## Architecture Terms

**Frontend**: Client-side application (Angular) that users interact with.

**Backend**: Server-side application (Spring Boot) that processes requests.

**Database**: Data storage system (PostgreSQL).

**Repository**: Pattern for data access layer.

**Service**: Business logic layer.

**Controller**: REST endpoint handler.

**Entity**: Database table representation in code.

**DTO**: Data Transfer Object - object for transferring data between layers.

## Authentication & Security

**JWT**: JSON Web Token - token format for authentication.

**Access Token**: Short-lived token for API authentication (15 minutes).

**Refresh Token**: Long-lived token for getting new access tokens (7 days).

**HttpOnly Cookie**: Cookie that cannot be accessed via JavaScript (prevents XSS).

**BCrypt**: Password hashing algorithm.

**CORS**: Cross-Origin Resource Sharing - mechanism for cross-origin requests.

**XSS**: Cross-Site Scripting - security vulnerability.

**CSRF**: Cross-Site Request Forgery - security attack.

**RBAC**: Role-Based Access Control - access control based on user roles.

## Database Terms

**ORM**: Object-Relational Mapping - technique for mapping objects to database tables.

**JPA**: Java Persistence API - Java standard for ORM.

**Hibernate**: JPA implementation used in Spring Boot.

**Migration**: Script for database schema changes.

**Flyway**: Database migration tool.

**Primary Key**: Unique identifier for a table row.

**Foreign Key**: Reference to a primary key in another table.

**Index**: Database structure for faster queries.

**Transaction**: Atomic database operation (all or nothing).

**ACID**: Atomicity, Consistency, Isolation, Durability - database transaction properties.

## Angular Terms

**Component**: Building block of Angular application (UI + logic).

**Service**: Singleton class for shared functionality.

**Module**: Organizes code (not used in standalone components).

**Guard**: Protects routes from unauthorized access.

**Interceptor**: Modifies HTTP requests/responses globally.

**Signal**: Reactive primitive for state management (Angular 17+).

**Observable**: Stream of data (RxJS).

**Directive**: Instruction for DOM manipulation.

**Pipe**: Transforms data in templates.

## Spring Boot Terms

**Bean**: Object managed by Spring container.

**Dependency Injection**: Automatic provision of dependencies.

**Annotation**: Metadata for code (e.g., `@Service`, `@Controller`).

**Auto-Configuration**: Automatic component configuration.

**Starter**: Pre-configured dependency set.

**Actuator**: Production-ready features (health checks, metrics).

## Project-Specific Terms

**Global Role**: User's system-wide role (ADMIN, PROFESSOR, STUDENT).

**Context Role**: User's role within a specific course (PROFESSOR, TA, TUTOR, STUDENT).

**Course Slug**: Human-readable URL identifier (e.g., "farwsp25").

**Enrollment**: Relationship between user and course.

**Material**: Course resource (file, document).

**Assignment**: Task with due date and grading.

**Submission**: Student's work for an assignment.

**Submission History**: Complete file upload history (never deleted).

## Technical Terms

**UUID**: Universally Unique Identifier - unique ID format.

**MIME Type**: Media type (e.g., "application/pdf", "image/png").

**Multipart**: HTTP content type for file uploads.

**Environment Variable**: Configuration value from environment.

**Docker**: Containerization platform.

**Docker Compose**: Tool for multi-container applications.

**Nginx**: Web server and reverse proxy.

**Reverse Proxy**: Server that forwards requests to other servers.

## Development Terms

**Hot Reload**: Automatic application restart/reload on code changes.

**Build**: Process of compiling/transpiling code.

**Deploy**: Process of making application available.

**CI/CD**: Continuous Integration/Continuous Deployment.

**Git**: Version control system.

**Branch**: Independent line of development.

**Commit**: Snapshot of changes.

**PR**: Pull Request - request to merge changes.

## File Formats

**YAML**: YAML Ain't Markup Language - configuration format.

**SQL**: Structured Query Language - database query language.

**TypeScript**: Typed superset of JavaScript.

**Java**: Programming language (backend).

---

**Document Version**: 1.0  
**Last Updated**: December 2024




