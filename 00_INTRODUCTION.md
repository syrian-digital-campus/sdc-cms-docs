# Introduction to SDC-CMS

## What is SDC-CMS?

The **Syrian Digital Campus Course Management System (SDC-CMS)** is a comprehensive web-based platform designed for managing courses, assignments, materials, and student submissions in a university environment. It serves as a centralized system where professors, teaching assistants, tutors, and students can collaborate on academic activities.

## Project Goals

### Primary Objectives

1. **Course Management**: Create, manage, and organize courses with proper enrollment and access control
2. **Material Distribution**: Share course materials (documents, files, resources) with timed visibility
3. **Assignment Management**: Create assignments, collect submissions, and facilitate grading
4. **User Management**: Support multiple user roles (Admin, Professor, TA, Tutor, Student) with appropriate permissions
5. **Security**: Ensure secure file access, authentication, and authorization throughout the system

### Target Users

- **Administrators**: System-wide management and configuration
- **Professors**: Course creation, material upload, assignment creation, and grading
- **Teaching Assistants (TAs)**: Help manage courses and grade assignments
- **Tutors**: Assist with course management and student support
- **Students**: Access course materials, submit assignments, view grades

## Key Features

### Implemented Features (MVP)

✅ **Authentication & Authorization**
- JWT-based authentication with refresh tokens
- Role-based access control (RBAC)
- Secure password hashing with BCrypt
- HttpOnly cookies for refresh token storage

✅ **Course Management**
- Create and manage courses
- Public course listing
- Course enrollment with role assignment
- Course slug-based URLs (human-readable)

✅ **Materials Management**
- Upload and organize course materials
- Timed visibility windows (visible from/until dates)
- Secure file download
- File metadata tracking

✅ **Assignments**
- Create assignments with due dates
- Upload specification files (PDFs, documents)
- Maximum points configuration
- Visibility control

✅ **Submissions** (Backend Complete)
- Submit assignments with multiple files
- File upload history tracking
- Deadline enforcement
- Grading system with feedback

### Planned Features

🚧 **Submissions UI** (Frontend)
- Student submission interface
- Staff grading interface
- Submission history viewer

🚧 **News & Announcements**
- Course news management
- Announcement system

🚧 **Timetable**
- Course schedule management
- Time slot booking

## System Overview

### High-Level Architecture

```
┌─────────────────┐
│   Web Browser   │
│   (Angular SPA) │
└────────┬────────┘
         │ HTTP/REST
         │ (JSON)
         │
┌────────▼────────┐
│  Spring Boot    │
│  REST API       │
│  (Port 8080)    │
└────────┬────────┘
         │ JDBC
         │
┌────────▼────────┐
│   PostgreSQL    │
│   Database      │
│   (Port 5432)   │
└─────────────────┘
```

### Technology Stack

**Frontend:**
- Angular 21 (TypeScript)
- RxJS for reactive programming
- Standalone components architecture

**Backend:**
- Spring Boot 4.0.1 (Java 17)
- Spring Security for authentication
- Spring Data JPA for database access
- Flyway for database migrations

**Database:**
- PostgreSQL 16
- Relational database with proper constraints

**Infrastructure:**
- Docker & Docker Compose
- Nginx (for frontend serving)
- Multi-stage Docker builds

### Communication Flow

1. **User Interaction**: User interacts with Angular frontend (browser)
2. **API Requests**: Frontend makes HTTP requests to Spring Boot backend
3. **Authentication**: Backend validates JWT tokens and checks authorization
4. **Data Processing**: Backend processes requests using business logic
5. **Database Queries**: Backend queries PostgreSQL database via JPA/Hibernate
6. **Response**: Backend returns JSON responses to frontend
7. **UI Update**: Frontend updates the user interface based on responses

## Project Structure

### Repository Organization

The project is split into multiple repositories:

```
syrian-digital-campus/
├── sdc-cms-backend/      # Spring Boot REST API
├── sdc-cms-frontend/     # Angular web application
├── sdc-cms-infra/        # Docker Compose infrastructure
└── sdc-cms-docs/         # Documentation (this repo)
```

### Why Multiple Repositories?

- **Separation of Concerns**: Each repository has a clear, focused purpose
- **Independent Development**: Frontend and backend can be developed independently
- **Independent Deployment**: Components can be deployed separately
- **Team Organization**: Different teams can work on different repositories
- **Version Control**: Each component has its own version history

## Design Principles

### 1. Security First
- All endpoints are secured by default
- JWT tokens for stateless authentication
- HttpOnly cookies prevent XSS attacks
- Password hashing with BCrypt
- Input validation and sanitization
- SQL injection prevention via parameterized queries

### 2. RESTful API Design
- Standard HTTP methods (GET, POST, PUT, DELETE)
- Resource-based URLs
- JSON request/response format
- Proper HTTP status codes
- Consistent error handling

### 3. Database Migrations
- Flyway manages all database schema changes
- Version-controlled migrations
- Reproducible database setup
- No manual SQL execution needed

### 4. Modular Architecture
- Clear separation between frontend and backend
- Layered architecture in backend (Controller → Service → Repository)
- Component-based architecture in frontend
- Reusable services and utilities

### 5. Type Safety
- Strong typing in TypeScript (frontend)
- Strong typing in Java (backend)
- Compile-time error detection
- Better IDE support and refactoring

## User Roles and Permissions

### Global Roles

- **ADMIN**: Full system access, can manage all courses and users
- **PROFESSOR**: Can create and manage courses, grade assignments
- **STUDENT**: Can enroll in courses, submit assignments, view materials

### Context Roles (Course-Specific)

- **PROFESSOR**: Course owner, full control over the course
- **TA (Teaching Assistant)**: Can manage materials, grade submissions
- **TUTOR**: Can view and assist with course management
- **STUDENT**: Can view materials, submit assignments

### Permission Matrix

| Action | Admin | Professor | TA | Tutor | Student |
|--------|-------|-----------|----|----|---------|
| Create Course | ✅ | ✅ | ❌ | ❌ | ❌ |
| Manage Course | ✅ | ✅ (own) | ✅ (assigned) | ❌ | ❌ |
| Upload Materials | ✅ | ✅ (own courses) | ✅ (assigned) | ❌ | ❌ |
| Download Materials | ✅ | ✅ | ✅ | ✅ | ✅ (visible) |
| Create Assignments | ✅ | ✅ (own courses) | ✅ (assigned) | ❌ | ❌ |
| Submit Assignments | ❌ | ❌ | ❌ | ❌ | ✅ |
| Grade Submissions | ✅ | ✅ (own courses) | ✅ (assigned) | ❌ | ❌ |
| View Own Grades | ✅ | ✅ | ✅ | ✅ | ✅ |

## Data Flow Examples

### Example 1: Student Submitting Assignment

```
1. Student opens assignment detail page
   → Frontend: GET /api/v1/courses/{courseId}/assignments/{assignmentId}
   → Backend: Validates JWT, checks enrollment, returns assignment data
   → Frontend: Displays assignment details and submission form

2. Student selects files and submits
   → Frontend: POST /api/v1/courses/{courseId}/assignments/{assignmentId}/submissions
   → Backend: Validates deadline, checks existing submission, saves files
   → Database: Inserts submission and file records
   → Backend: Returns submission confirmation
   → Frontend: Shows success message and updates UI
```

### Example 2: Professor Uploading Material

```
1. Professor navigates to Materials page
   → Frontend: GET /api/v1/courses/{courseId}/materials
   → Backend: Validates permissions, returns material list
   → Frontend: Displays materials with upload button

2. Professor uploads material with file
   → Frontend: POST /api/v1/courses/{courseId}/materials (multipart/form-data)
   → Backend: Validates file, saves to storage, creates database record
   → Database: Inserts material record
   → Backend: Returns material data
   → Frontend: Refreshes material list
```

### Example 3: Student Downloading Material

```
1. Student clicks download on material
   → Frontend: GET /api/v1/courses/{courseId}/materials/{materialId}/download
   → Backend: Validates JWT, checks material visibility, reads file
   → Backend: Streams file to response with proper headers
   → Frontend: Browser downloads file
```

## Next Steps

Now that you understand what SDC-CMS is, continue reading:

1. **[01_ARCHITECTURE.md](./01_ARCHITECTURE.md)** - Deep dive into system architecture
2. **[02_SETUP_GUIDE.md](./02_SETUP_GUIDE.md)** - Set up your development environment
3. **[03_TECHNOLOGY_STACK.md](./03_TECHNOLOGY_STACK.md)** - Learn about the technologies used

## Questions?

If you have questions about the project:
- Check the [Architecture document](./01_ARCHITECTURE.md) for design details
- Review the [Setup Guide](./02_SETUP_GUIDE.md) for installation help
- See [Troubleshooting](./13_TROUBLESHOOTING.md) for common issues

---

**Document Version**: 1.0  
**Last Updated**: December 2024




