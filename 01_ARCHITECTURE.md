# System Architecture

## Overview

SDC-CMS follows a **three-tier architecture** pattern:

1. **Presentation Layer**: Angular frontend (browser)
2. **Application Layer**: Spring Boot REST API
3. **Data Layer**: PostgreSQL database

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    CLIENT (Browser)                          │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              Angular SPA Application                 │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐          │   │
│  │  │Components│  │ Services │  │  Models  │          │   │
│  │  └──────────┘  └──────────┘  └──────────┘          │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐          │   │
│  │  │  Guards  │  │Interceptors│ │  Routes  │          │   │
│  │  └──────────┘  └──────────┘  └──────────┘          │   │
│  └──────────────────────────────────────────────────────┘   │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        │ HTTP/REST (JSON)
                        │ JWT Authentication
                        │
┌───────────────────────▼─────────────────────────────────────┐
│              APPLICATION SERVER (Spring Boot)                │
│  ┌──────────────────────────────────────────────────────┐   │
│  │              REST Controllers                        │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐          │   │
│  │  │   Auth   │  │  Course  │  │ Material │          │   │
│  │  │Controller│  │Controller│  │Controller│  ...     │   │
│  │  └──────────┘  └──────────┘  └──────────┘          │   │
│  └──────────────┬───────────────────────────────────────┘   │
│                 │                                            │
│  ┌──────────────▼───────────────────────────────────────┐   │
│  │              Business Logic Layer                     │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐          │   │
│  │  │  Auth    │  │  Course  │  │ Material │          │   │
│  │  │ Service  │  │ Service  │  │ Service  │  ...     │   │
│  │  └──────────┘  └──────────┘  └──────────┘          │   │
│  └──────────────┬───────────────────────────────────────┘   │
│                 │                                            │
│  ┌──────────────▼───────────────────────────────────────┐   │
│  │              Data Access Layer                        │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐          │   │
│  │  │   User   │  │  Course  │  │ Material │          │   │
│  │  │Repository│  │Repository│  │Repository│  ...     │   │
│  │  └──────────┘  └──────────┘  └──────────┘          │   │
│  └──────────────┬───────────────────────────────────────┘   │
│                 │                                            │
│  ┌──────────────▼───────────────────────────────────────┐   │
│  │              Spring Security                          │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐          │   │
│  │  │  JWT     │  │  Auth    │  │  CORS    │          │   │
│  │  │  Filter  │  │  Filter  │  │  Config  │          │   │
│  │  └──────────┘  └──────────┘  └──────────┘          │   │
│  └──────────────────────────────────────────────────────┘   │
└───────────────────────┬─────────────────────────────────────┘
                        │
                        │ JDBC/SQL
                        │
┌───────────────────────▼─────────────────────────────────────┐
│              DATA LAYER (PostgreSQL)                         │
│  ┌──────────────────────────────────────────────────────┐   │
│  │                  Database Tables                      │   │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐          │   │
│  │  │  users   │  │ courses  │  │ materials│          │   │
│  │  │enrollments│ │assignments│ │submissions│  ...     │   │
│  │  └──────────┘  └──────────┘  └──────────┘          │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## Frontend Architecture (Angular)

### Component Structure

Angular follows a **component-based architecture**:

```
src/app/
├── core/                    # Core functionality (singleton services)
│   ├── guards/             # Route guards (auth, role)
│   ├── interceptors/       # HTTP interceptors (auth, error)
│   ├── models/             # TypeScript interfaces/types
│   └── services/           # Shared services (API, Auth)
│
├── features/               # Feature modules
│   ├── auth/              # Authentication features
│   │   ├── login/
│   │   └── register/
│   ├── courses/           # Course management
│   ├── materials/         # Material management
│   └── assignments/       # Assignment management
│
├── shared/                # Shared components
│   └── components/
│       └── navbar/
│
└── app.component.ts       # Root component
```

### Data Flow in Frontend

```
User Action
    ↓
Component Event Handler
    ↓
Service Method Call
    ↓
HTTP Request (via HttpClient)
    ↓
Auth Interceptor (adds JWT token)
    ↓
Backend API
    ↓
Response
    ↓
Service (processes response)
    ↓
Component (updates view)
    ↓
User Sees Updated UI
```

### Key Frontend Concepts

**1. Services**
- Singleton classes that handle business logic
- Communicate with backend via HTTP
- Share data between components
- Example: `AuthService`, `CourseService`

**2. Components**
- Self-contained UI units
- Have template (HTML), styles (CSS), and logic (TypeScript)
- Communicate via inputs/outputs
- Example: `LoginComponent`, `CourseListComponent`

**3. Guards**
- Protect routes from unauthorized access
- Run before route navigation
- Example: `AuthGuard`, `RoleGuard`

**4. Interceptors**
- Modify HTTP requests/responses globally
- Add headers, handle errors, refresh tokens
- Example: `AuthInterceptor`

**5. Models/Interfaces**
- TypeScript type definitions
- Ensure type safety
- Example: `Course`, `User`, `Material`

## Backend Architecture (Spring Boot)

### Layered Architecture

Spring Boot follows a **layered architecture** pattern:

```
┌─────────────────────────────────────┐
│     Presentation Layer              │
│  ┌───────────────────────────────┐  │
│  │     REST Controllers          │  │
│  │  - Handle HTTP requests       │  │
│  │  - Validate input             │  │
│  │  - Return HTTP responses      │  │
│  └───────────────────────────────┘  │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│     Business Logic Layer            │
│  ┌───────────────────────────────┐  │
│  │     Services                  │  │
│  │  - Implement business logic   │  │
│  │  - Transaction management     │  │
│  │  - Coordinate between layers  │  │
│  └───────────────────────────────┘  │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│     Data Access Layer               │
│  ┌───────────────────────────────┐  │
│  │     Repositories (JPA)        │  │
│  │  - Database operations        │  │
│  │  - Query methods              │  │
│  │  - Entity mapping             │  │
│  └───────────────────────────────┘  │
└──────────────┬──────────────────────┘
               │
┌──────────────▼──────────────────────┐
│     Database                        │
│  ┌───────────────────────────────┐  │
│  │     PostgreSQL                │  │
│  │  - Data storage               │  │
│  │  - ACID transactions          │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

### Package Structure

```
org.scikeys.cms/
├── CmsApplication.java          # Main application class
│
├── auth/                        # Authentication module
│   ├── controller/
│   ├── service/
│   ├── dto/
│   └── config/
│
├── users/                       # User management
│   ├── entity/
│   ├── repository/
│   └── dto/
│
├── courses/                     # Course management
│   ├── controller/
│   ├── service/
│   ├── repository/
│   ├── entity/
│   └── dto/
│
├── materials/                   # Material management
│   ├── controller/
│   ├── service/
│   ├── repository/
│   ├── entity/
│   └── dto/
│
├── assignments/                 # Assignment management
│   └── ...
│
├── submissions/                 # Submission management
│   └── ...
│
├── common/                      # Shared utilities
│   ├── security/               # Security-related utilities
│   ├── exception/              # Exception handlers
│   └── enums/                  # Enumerations
│
└── config/                      # Configuration classes
    ├── SecurityConfig.java
    ├── WebConfig.java
    └── DataInitializer.java
```

### Request Processing Flow

```
HTTP Request
    ↓
Spring Security Filter Chain
    ↓ (JWT validation, authentication)
Controller (REST endpoint)
    ↓ (validates input, extracts parameters)
Service (business logic)
    ↓ (validates business rules, processes data)
Repository (data access)
    ↓ (executes SQL queries)
Database
    ↓ (returns data)
Repository (maps to entities)
    ↓
Service (processes entities, returns DTOs)
    ↓
Controller (returns HTTP response)
    ↓
Spring Security (adds security headers)
    ↓
HTTP Response
```

## Database Architecture

### Entity Relationship Diagram (Simplified)

```
┌──────────┐
│   User   │
└────┬─────┘
     │ 1
     │
     │ *
┌────▼──────────┐
│  Enrollment   │
└────┬──────┬───┘
     │      │
     │ *    │ *
┌────▼──┐ ┌─▼────────┐
│Course │ │Enrollment│
└───┬───┘ │  Role    │
    │     └──────────┘
    │ 1
    │
    │ *
┌───▼─────────┐
│  Material   │
└─────────────┘

┌──────────┐
│   User   │
└────┬─────┘
     │ 1
     │
     │ *
┌────▼──────────┐
│  Assignment   │
└────┬──────────┘
     │ 1
     │
     │ *
┌────▼──────────┐
│  Submission   │
└────┬──────────┘
     │ 1
     │
     │ *
┌────▼──────────┐
│SubmissionFile │
└───────────────┘
```

### Key Database Concepts

**1. Entities (Tables)**
- Represent business objects
- Mapped by JPA/Hibernate
- Example: `User`, `Course`, `Assignment`

**2. Relationships**
- **One-to-Many**: One course has many materials
- **Many-to-Many**: Users enroll in courses (via Enrollment table)
- **One-to-One**: User has one UserProfile

**3. Migrations**
- Flyway manages schema changes
- Version-controlled SQL scripts
- Applied automatically on startup

**4. Transactions**
- Ensure data consistency
- Managed by Spring (`@Transactional`)
- ACID properties guaranteed

## Security Architecture

### Authentication Flow

```
1. User submits login credentials
   ↓
2. Backend validates credentials
   ↓
3. Backend generates:
   - Access Token (JWT, 15 min expiry)
   - Refresh Token (UUID, 7 days expiry)
   ↓
4. Backend returns:
   - Access Token (in response body)
   - Refresh Token (in HttpOnly cookie)
   ↓
5. Frontend stores Access Token in memory
   ↓
6. All API requests include:
   - Authorization: Bearer <access_token>
   ↓
7. When Access Token expires:
   - Frontend calls /auth/refresh
   - Backend validates Refresh Token (cookie)
   - Backend returns new Access Token
```

### Authorization Flow

```
1. User makes API request
   ↓
2. Spring Security extracts JWT token
   ↓
3. Spring Security validates JWT
   ↓
4. Spring Security extracts user info from JWT
   ↓
5. Spring Security checks user roles/permissions
   ↓
6. Controller receives authenticated user info
   ↓
7. Service validates business-level permissions
   ↓
8. Request is processed if authorized
```

### Security Layers

**1. Network Layer**
- HTTPS (in production)
- CORS configuration
- Firewall rules

**2. Application Layer**
- Spring Security filter chain
- JWT token validation
- Role-based access control

**3. Data Layer**
- Parameterized queries (SQL injection prevention)
- Input validation
- Output encoding

**4. File Storage**
- Secure file paths
- Access control
- Virus scanning (recommended in production)

## Communication Protocols

### HTTP/REST

**Request Format:**
```
POST /api/v1/auth/login HTTP/1.1
Host: localhost:8080
Content-Type: application/json

{
  "username": "student1",
  "password": "password123"
}
```

**Response Format:**
```
HTTP/1.1 200 OK
Content-Type: application/json
Set-Cookie: refreshToken=...; HttpOnly; Secure; SameSite=Strict

{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": "...",
    "username": "student1",
    ...
  }
}
```

### WebSocket (Future)

Currently not implemented, but could be added for:
- Real-time notifications
- Live chat
- Collaboration features

## File Storage Architecture

### Local File Storage

```
/data/files/
├── 2024/
│   ├── 12/
│   │   ├── 21/
│   │   │   ├── <file-hash-1>/
│   │   │   │   └── original-filename.pdf
│   │   │   └── <file-hash-2>/
│   │   │       └── document.docx
```

### File Metadata

Files are stored in the database with:
- Original filename
- Content type (MIME type)
- File size
- SHA-256 checksum
- Storage path
- Creation timestamp

### File Access Flow

```
1. User requests file download
   ↓
2. Backend validates:
   - User authentication
   - User authorization (can access this file?)
   - File visibility (within time window?)
   ↓
3. Backend reads file from storage
   ↓
4. Backend streams file to response
   ↓
5. Browser downloads file
```

## Deployment Architecture

### Development Environment

```
Developer Machine
├── Docker Compose
│   ├── PostgreSQL (localhost:5432)
│   ├── Backend (localhost:8080)
│   └── Frontend (localhost:4200)
```

### Production Environment (Recommended)

```
┌─────────────────────────────────────┐
│         Load Balancer               │
│         (Nginx/HAProxy)             │
└───────────┬─────────────┬───────────┘
            │             │
    ┌───────▼──────┐ ┌────▼──────┐
    │   Frontend   │ │  Backend  │
    │   (Nginx)    │ │ (Spring)  │
    │  Static Files│ │  API      │
    └──────────────┘ └─────┬─────┘
                           │
                  ┌────────▼────────┐
                  │   PostgreSQL    │
                  │   (Database)    │
                  └─────────────────┘
```

## Scalability Considerations

### Current Design (MVP)

- Single server deployment
- File storage on local filesystem
- In-memory session management (stateless)

### Future Scalability Options

**Horizontal Scaling:**
- Multiple backend instances behind load balancer
- Shared file storage (S3, NFS, etc.)
- Stateless design supports scaling

**Database Scaling:**
- Read replicas for reporting
- Connection pooling
- Query optimization

**Caching:**
- Redis for session management
- Cache frequently accessed data
- Reduce database load

## Next Steps

- **[02_SETUP_GUIDE.md](./02_SETUP_GUIDE.md)** - Set up the development environment
- **[03_TECHNOLOGY_STACK.md](./03_TECHNOLOGY_STACK.md)** - Learn about technologies used
- **[11_SECURITY.md](./11_SECURITY.md)** - Deep dive into security

---

**Document Version**: 1.0  
**Last Updated**: December 2024

