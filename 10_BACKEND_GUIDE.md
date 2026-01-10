# Backend Development Guide

SDC-CMS specific backend development patterns and practices.

## Project Structure

```
org.scikeys.cms/
├── CmsApplication.java      # Main application
├── auth/                    # Authentication module
│   ├── controller/
│   ├── service/
│   └── dto/
├── courses/                 # Courses module
│   ├── controller/
│   ├── service/
│   ├── repository/
│   ├── entity/
│   └── dto/
├── common/                  # Shared utilities
│   ├── security/
│   ├── exception/
│   └── enums/
└── config/                  # Configuration
```

## Layered Architecture Pattern

### Controller → Service → Repository

**Controller** (thin layer):
```java
@RestController
@RequestMapping("/api/v1/courses")
@RequiredArgsConstructor
public class CourseController {
    private final CourseService courseService;
    
    @GetMapping("/{id}")
    public ResponseEntity<CourseResponse> getCourse(@PathVariable UUID id) {
        CourseResponse course = courseService.getCourse(id);
        return ResponseEntity.ok(course);
    }
}
```

**Service** (business logic):
```java
@Service
@Transactional
@RequiredArgsConstructor
public class CourseService {
    private final CourseRepository courseRepository;
    
    public CourseResponse getCourse(UUID id) {
        Course course = courseRepository.findById(id)
            .orElseThrow(() -> new CourseNotFoundException(id));
        return toCourseResponse(course);
    }
    
    private CourseResponse toCourseResponse(Course course) {
        return CourseResponse.builder()
            .id(course.getId())
            .title(course.getTitle())
            .build();
    }
}
```

**Repository** (data access):
```java
@Repository
public interface CourseRepository extends JpaRepository<Course, UUID> {
    Optional<Course> findByCourseCodeAndYear(String courseCode, Integer year);
}
```

## Entity Pattern

### Standard Entity

```java
@Entity
@Table(name = "courses")
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class Course {
    @Id
    @GeneratedValue
    private UUID id;
    
    @Column(nullable = false)
    private String title;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "owner_user_id")
    private User owner;
    
    @OneToMany(mappedBy = "course", cascade = CascadeType.ALL)
    private List<Material> materials;
    
    @CreationTimestamp
    private Instant createdAt;
    
    @UpdateTimestamp
    private Instant updatedAt;
}
```

## DTO Pattern

### Request DTO

```java
@Data
@NoArgsConstructor
@AllArgsConstructor
public class CourseCreateRequest {
    @NotBlank(message = "Title is required")
    @Size(max = 500)
    private String title;
    
    private String description;
    
    @NotNull
    private Integer year;
}
```

### Response DTO

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class CourseResponse {
    private UUID id;
    private String title;
    private String ownerName;
    private Instant createdAt;
}
```

### Mapping Entity to DTO

```java
private CourseResponse toCourseResponse(Course course) {
    return CourseResponse.builder()
        .id(course.getId())
        .title(course.getTitle())
        .ownerName(course.getOwner().getUsername())
        .createdAt(course.getCreatedAt())
        .build();
}
```

## Exception Handling Pattern

### Custom Exception

```java
public class CourseNotFoundException extends RuntimeException {
    public CourseNotFoundException(UUID id) {
        super("Course not found: " + id);
    }
}
```

### Global Exception Handler

```java
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(CourseNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleCourseNotFound(
            CourseNotFoundException ex) {
        ErrorResponse error = new ErrorResponse("COURSE_NOT_FOUND", ex.getMessage());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(
            MethodArgumentNotValidException ex) {
        String message = ex.getBindingResult().getAllErrors().stream()
            .map(DefaultMessageSourceResolvable::getDefaultMessage)
            .collect(Collectors.joining(", "));
        ErrorResponse error = new ErrorResponse("VALIDATION_ERROR", message);
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(error);
    }
}
```

## Security Pattern

### @CurrentUser Annotation

Custom annotation to inject current user:

```java
@Target(ElementType.PARAMETER)
@Retention(RetentionPolicy.RUNTIME)
public @interface CurrentUser {
}
```

**Usage**:
```java
@GetMapping("/courses")
public List<CourseResponse> getCourses(@CurrentUser UUID userId) {
    // userId extracted from JWT token
    return courseService.getCoursesForUser(userId);
}
```

### Authorization Checks

```java
@Service
public class CourseService {
    
    public void deleteCourse(UUID courseId, UUID userId) {
        Course course = findCourseById(courseId);
        
        // Check authorization
        if (!course.getOwner().getId().equals(userId)) {
            throw new ForbiddenException("You don't have permission to delete this course");
        }
        
        courseRepository.delete(course);
    }
}
```

## Validation Pattern

### Input Validation

```java
@PostMapping("/courses")
public ResponseEntity<CourseResponse> createCourse(
        @Valid @RequestBody CourseCreateRequest request,
        @CurrentUser UUID userId) {
    CourseResponse course = courseService.createCourse(request, userId);
    return ResponseEntity.status(HttpStatus.CREATED).body(course);
}
```

### Service-Level Validation

```java
@Service
public class CourseService {
    
    public CourseResponse createCourse(CourseCreateRequest request, UUID userId) {
        // Business rule validation
        if (courseRepository.existsByCourseCodeAndYear(
                request.getCourseCode(), request.getYear())) {
            throw new ConflictException("Course already exists for this year");
        }
        
        // Create course...
    }
}
```

## Transaction Management

### @Transactional

```java
@Service
@Transactional
public class CourseService {
    
    @Transactional  // All operations in this method are atomic
    public CourseResponse createCourseWithMaterials(
            CourseCreateRequest request, 
            List<MaterialCreateRequest> materials) {
        Course course = createCourse(request);
        
        for (MaterialCreateRequest materialRequest : materials) {
            Material material = createMaterial(materialRequest, course);
            materialRepository.save(material);
        }
        
        // If any operation fails, all are rolled back
        return toCourseResponse(course);
    }
}
```

## Best Practices

1. **Use Constructor Injection**: Prefer over field injection
2. **Keep Controllers Thin**: Delegate to services
3. **Use DTOs**: Don't expose entities directly
4. **Handle Exceptions Globally**: Use `@RestControllerAdvice`
5. **Validate Input**: Use `@Valid` and validation annotations
6. **Use Transactions**: `@Transactional` for multi-step operations
7. **Use Lombok**: Reduce boilerplate (`@Data`, `@RequiredArgsConstructor`)
8. **Follow Naming Conventions**: Clear, descriptive names

## Next Steps

- Read [04_SPRING_BOOT_GUIDE.md](./04_SPRING_BOOT_GUIDE.md) for Spring Boot basics
- Explore existing code for patterns
- See [08_API_DOCUMENTATION.md](./08_API_DOCUMENTATION.md) for API details

---

**Document Version**: 1.0  
**Last Updated**: December 2024




