# Spring Boot Guide for Beginners

This guide is designed for developers new to Spring Boot. It explains core concepts and how they're used in SDC-CMS.

## What is Spring Boot?

Spring Boot is a Java framework that simplifies building production-ready applications. It provides:

- **Auto-configuration**: Automatically configures components based on dependencies
- **Embedded Server**: Runs without external server (Tomcat embedded)
- **Starter Dependencies**: Pre-configured dependency sets
- **Production Features**: Health checks, metrics, externalized configuration

## Core Concepts

### 1. Dependency Injection (DI)

**What is it?**
Dependency Injection means Spring automatically provides objects (dependencies) to classes that need them, rather than classes creating them manually.

**Why is it useful?**
- **Loose Coupling**: Classes depend on interfaces, not concrete implementations
- **Testability**: Easy to replace dependencies with mocks in tests
- **Flexibility**: Change implementations without modifying dependent classes

**Example in SDC-CMS**:

```java
// Service class - Spring automatically creates and manages this
@Service
public class CourseService {
    
    // Spring automatically injects UserRepository
    private final UserRepository userRepository;
    
    // Constructor injection (recommended)
    public CourseService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public Course createCourse(CourseCreateRequest request, UUID userId) {
        // Use injected repository
        User owner = userRepository.findById(userId)
            .orElseThrow(() -> new UserNotFoundException(userId));
        // ... create course logic
    }
}
```

**How Spring Knows What to Inject**:
- Spring scans for classes annotated with `@Component`, `@Service`, `@Repository`, `@Controller`
- Creates instances (beans) of these classes
- When a class needs a dependency, Spring provides an instance

### 2. Annotations

Annotations are metadata that tell Spring how to handle classes and methods.

**Common Annotations**:

#### `@SpringBootApplication`
Marks the main application class. Spring Boot scans from this package downward.

```java
@SpringBootApplication
public class CmsApplication {
    public static void main(String[] args) {
        SpringApplication.run(CmsApplication.class, args);
    }
}
```

#### `@RestController`
Marks a class as a REST controller. Combines `@Controller` and `@ResponseBody`.

```java
@RestController
@RequestMapping("/api/v1/courses")
public class CourseController {
    // Methods return JSON responses automatically
}
```

#### `@Service`
Marks a class as a service (business logic layer).

```java
@Service
public class CourseService {
    // Business logic here
}
```

#### `@Repository`
Marks a class as a repository (data access layer). Spring Data JPA provides implementations automatically.

```java
@Repository
public interface CourseRepository extends JpaRepository<Course, UUID> {
    // Spring generates implementation automatically
}
```

#### `@Autowired` / Constructor Injection

**Old way (field injection - not recommended)**:
```java
@Service
public class CourseService {
    @Autowired
    private UserRepository userRepository;  // Field injection
}
```

**New way (constructor injection - recommended)**:
```java
@Service
public class CourseService {
    private final UserRepository userRepository;
    
    // Constructor injection - Spring automatically calls this
    public CourseService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
}
```

**Why Constructor Injection?**
- Required dependencies are clear
- Fields can be `final` (immutable)
- Easier to test (can pass mocks in constructor)
- Fail fast if dependency is missing

#### `@RequestMapping`
Maps HTTP requests to methods. Can be on class or method level.

```java
@RestController
@RequestMapping("/api/v1/courses")  // Base path for all methods
public class CourseController {
    
    @GetMapping  // GET /api/v1/courses
    public List<Course> getAllCourses() { ... }
    
    @GetMapping("/{id}")  // GET /api/v1/courses/{id}
    public Course getCourse(@PathVariable UUID id) { ... }
    
    @PostMapping  // POST /api/v1/courses
    public Course createCourse(@RequestBody CourseCreateRequest request) { ... }
}
```

#### HTTP Method Annotations
- `@GetMapping` - Handle GET requests
- `@PostMapping` - Handle POST requests
- `@PutMapping` - Handle PUT requests
- `@DeleteMapping` - Handle DELETE requests
- `@PatchMapping` - Handle PATCH requests

#### Parameter Annotations

**@PathVariable**: Extract path variables
```java
@GetMapping("/courses/{id}")
public Course getCourse(@PathVariable UUID id) {
    // id comes from URL: /courses/123e4567-e89b-12d3-a456-426614174000
}
```

**@RequestParam**: Extract query parameters
```java
@GetMapping("/courses")
public List<Course> getCourses(@RequestParam(required = false) Integer year) {
    // year comes from URL: /courses?year=2024
}
```

**@RequestBody**: Extract JSON body
```java
@PostMapping("/courses")
public Course createCourse(@RequestBody CourseCreateRequest request) {
    // request is parsed from JSON body
}
```

**@CurrentUser**: Custom annotation (SDC-CMS specific)
```java
@GetMapping("/courses")
public List<Course> getCourses(@CurrentUser UUID userId) {
    // userId comes from JWT token (extracted by custom resolver)
}
```

### 3. Spring Data JPA

**What is JPA?**
Java Persistence API - standard for object-relational mapping (ORM).

**What is Spring Data JPA?**
Provides repositories that automatically generate queries.

**Basic Repository**:
```java
@Repository
public interface CourseRepository extends JpaRepository<Course, UUID> {
    // JpaRepository provides:
    // - save(entity)
    // - findById(id)
    // - findAll()
    // - delete(entity)
    // - count()
    // etc.
}
```

**Custom Query Methods**:
```java
@Repository
public interface CourseRepository extends JpaRepository<Course, UUID> {
    // Spring generates query from method name
    List<Course> findByYear(Integer year);
    
    // Multiple conditions
    List<Course> findByYearAndSemester(Integer year, Semester semester);
    
    // Optional return
    Optional<Course> findByCourseCodeAndYear(String courseCode, Integer year);
    
    // Custom query with @Query
    @Query("SELECT c FROM Course c WHERE c.owner.id = :userId")
    List<Course> findByOwnerId(@Param("userId") UUID userId);
}
```

**How It Works**:
1. Spring creates implementation of interface at runtime
2. Method names like `findByYear` automatically generate SQL: `SELECT * FROM courses WHERE year = ?`
3. Spring executes query and maps results to entities

### 4. Entities and Relationships

**Entity**: Java class mapped to database table.

```java
@Entity
@Table(name = "courses")
@Data  // Lombok - generates getters, setters, toString, etc.
@NoArgsConstructor
@AllArgsConstructor
public class Course {
    
    @Id
    @GeneratedValue
    private UUID id;
    
    @Column(nullable = false)
    private String title;
    
    private String description;
    
    // Relationships
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "owner_user_id")
    private User owner;
    
    @OneToMany(mappedBy = "course", cascade = CascadeType.ALL)
    private List<Material> materials;
}
```

**Relationship Annotations**:
- `@OneToOne`: One-to-one relationship
- `@OneToMany`: One-to-many relationship
- `@ManyToOne`: Many-to-one relationship
- `@ManyToMany`: Many-to-many relationship

**Fetch Types**:
- `LAZY`: Load data only when accessed (default for @OneToMany, @ManyToMany)
- `EAGER`: Load data immediately (default for @ManyToOne, @OneToOne)

**Why LAZY by default?**
- Performance: Don't load data you don't need
- Avoid N+1 problem: Loading related data can cause many queries

### 5. DTOs (Data Transfer Objects)

**What is a DTO?**
Objects used to transfer data between layers (e.g., Controller ↔ Service).

**Why use DTOs?**
- **Security**: Don't expose internal entity structure
- **API Design**: Control what data is sent/received
- **Performance**: Only transfer needed data
- **Versioning**: Change DTOs without changing entities

**Example**:

```java
// Entity (internal)
@Entity
public class Course {
    private UUID id;
    private String title;
    private User owner;  // Full user object
    // ... many other fields
}

// Request DTO (what client sends)
@Data
public class CourseCreateRequest {
    private String title;
    private String description;
    private String courseCode;
    // Only fields needed to create course
}

// Response DTO (what client receives)
@Data
public class CourseResponse {
    private UUID id;
    private String title;
    private String ownerName;  // Just the name, not full User object
    // Only fields client needs
}
```

### 6. Service Layer Pattern

**Three-Layer Architecture**:

```
Controller (REST endpoints)
    ↓
Service (Business logic)
    ↓
Repository (Data access)
```

**Controller**: Handles HTTP requests/responses
```java
@RestController
public class CourseController {
    private final CourseService courseService;
    
    @GetMapping("/courses/{id}")
    public CourseResponse getCourse(@PathVariable UUID id) {
        return courseService.getCourse(id);  // Delegates to service
    }
}
```

**Service**: Contains business logic
```java
@Service
public class CourseService {
    private final CourseRepository courseRepository;
    
    public CourseResponse getCourse(UUID id) {
        Course course = courseRepository.findById(id)
            .orElseThrow(() -> new CourseNotFoundException(id));
        // Business logic here
        return toCourseResponse(course);  // Convert entity to DTO
    }
}
```

**Repository**: Handles database operations
```java
@Repository
public interface CourseRepository extends JpaRepository<Course, UUID> {
    // Database operations
}
```

### 7. Exception Handling

**Global Exception Handler**:

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(CourseNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleCourseNotFound(
            CourseNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            "COURSE_NOT_FOUND",
            ex.getMessage()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
            ValidationException ex) {
        ErrorResponse error = new ErrorResponse(
            "VALIDATION_ERROR",
            ex.getMessage()
        );
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
}
```

**Custom Exceptions**:

```java
public class CourseNotFoundException extends RuntimeException {
    public CourseNotFoundException(UUID id) {
        super("Course not found: " + id);
    }
}
```

### 8. Validation

**Bean Validation Annotations**:

```java
@Data
public class CourseCreateRequest {
    
    @NotBlank(message = "Title is required")
    @Size(max = 500, message = "Title must not exceed 500 characters")
    private String title;
    
    @NotNull(message = "Year is required")
    @Min(value = 2020, message = "Year must be 2020 or later")
    private Integer year;
    
    @Email(message = "Email must be valid")
    private String email;
}
```

**Enable Validation in Controller**:

```java
@PostMapping("/courses")
public CourseResponse createCourse(@Valid @RequestBody CourseCreateRequest request) {
    // @Valid triggers validation
    // If validation fails, 400 Bad Request is returned automatically
    return courseService.createCourse(request);
}
```

### 9. Transaction Management

**@Transactional Annotation**:

```java
@Service
public class CourseService {
    
    @Transactional  // All database operations in this method are atomic
    public Course createCourseWithMaterials(CourseCreateRequest request) {
        Course course = new Course();
        // ... set fields
        course = courseRepository.save(course);
        
        Material material = new Material();
        material.setCourse(course);
        materialRepository.save(material);
        
        // If any operation fails, all are rolled back
        return course;
    }
}
```

**Why @Transactional?**
- **Atomicity**: All operations succeed or all fail
- **Consistency**: Database stays in valid state
- **Isolation**: Changes are isolated until commit

### 10. Configuration

**application.yaml**:

```yaml
spring:
  datasource:
    url: jdbc:postgresql://localhost:5432/university_cms
    username: postgres
    password: postgres
  jpa:
    hibernate:
      ddl-auto: validate  # Don't auto-create tables (Flyway does it)
    show_sql: true  # Log SQL queries (useful for debugging)

app:
  jwt:
    secret: ${JWT_SECRET:default-secret}
    access-token-validity: 900000  # 15 minutes
```

**Environment Variables**:
```bash
# Set in environment or .env file
export JWT_SECRET=your-secret-key
```

Access in code:
```java
@Value("${app.jwt.secret}")
private String jwtSecret;
```

## Common Patterns in SDC-CMS

### Pattern 1: Controller → Service → Repository

```java
// Controller
@RestController
@RequestMapping("/api/v1/courses")
public class CourseController {
    private final CourseService courseService;
    
    @GetMapping("/{id}")
    public CourseResponse getCourse(@PathVariable UUID id) {
        return courseService.getCourse(id);
    }
}

// Service
@Service
public class CourseService {
    private final CourseRepository courseRepository;
    
    public CourseResponse getCourse(UUID id) {
        Course course = courseRepository.findById(id)
            .orElseThrow(() -> new CourseNotFoundException(id));
        return toCourseResponse(course);
    }
}

// Repository
@Repository
public interface CourseRepository extends JpaRepository<Course, UUID> {
}
```

### Pattern 2: Custom Exception with Global Handler

```java
// Custom Exception
public class CourseNotFoundException extends RuntimeException {
    public CourseNotFoundException(UUID id) {
        super("Course not found: " + id);
    }
}

// Global Handler
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(CourseNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleCourseNotFound(
            CourseNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
            .body(new ErrorResponse("COURSE_NOT_FOUND", ex.getMessage()));
    }
}
```

### Pattern 3: DTO Mapping

```java
// Convert Entity to DTO
private CourseResponse toCourseResponse(Course course) {
    return CourseResponse.builder()
        .id(course.getId())
        .title(course.getTitle())
        .ownerName(course.getOwner().getUsername())
        .createdAt(course.getCreatedAt())
        .build();
}
```

## Spring Boot Starter Dependencies

**What are they?**
Pre-configured dependency sets that include everything needed for a feature.

**Examples in SDC-CMS**:

```xml
<!-- Web MVC (REST API) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webmvc</artifactId>
</dependency>

<!-- Security -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- Data JPA (Database access) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- Validation -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```

Each starter includes:
- The library itself
- Dependencies it needs
- Auto-configuration
- Sensible defaults

## Best Practices

1. **Use Constructor Injection** (not field injection)
2. **Separate Layers**: Controller → Service → Repository
3. **Use DTOs**: Don't expose entities directly
4. **Handle Exceptions Globally**: Use `@RestControllerAdvice`
5. **Validate Input**: Use `@Valid` and validation annotations
6. **Use Transactions**: `@Transactional` for multi-step operations
7. **Keep Services Stateless**: No instance variables in services
8. **Use Repository Pattern**: Don't access database directly in services

## Next Steps

- Practice with small examples
- Read Spring Boot documentation: https://spring.io/projects/spring-boot
- Explore SDC-CMS codebase to see patterns in action
- Read [10_BACKEND_GUIDE.md](./10_BACKEND_GUIDE.md) for SDC-CMS specific patterns

---

**Document Version**: 1.0  
**Last Updated**: December 2024

