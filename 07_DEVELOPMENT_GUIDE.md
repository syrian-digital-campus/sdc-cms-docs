# Development Guide

This guide covers development workflows, best practices, and how to contribute to SDC-CMS.

## Development Workflow

### Daily Development Process

1. **Pull Latest Changes**
   ```bash
   git pull origin main
   ```

2. **Create Feature Branch**
   ```bash
   git checkout -b feature/your-feature-name
   ```

3. **Make Changes**
   - Write code
   - Test locally
   - Commit frequently

4. **Commit Changes**
   ```bash
   git add .
   git commit -m "feat: Add new feature"
   ```

5. **Push and Create Pull Request**
   ```bash
   git push origin feature/your-feature-name
   # Then create PR on GitHub
   ```

### Git Workflow

**Branch Strategy**:
- `main`: Production-ready code
- `develop`: Integration branch (if using Git Flow)
- `feature/*`: New features
- `bugfix/*`: Bug fixes
- `hotfix/*`: Critical production fixes

**Commit Messages**:
Follow Conventional Commits:
- `feat:` New feature
- `fix:` Bug fix
- `docs:` Documentation changes
- `style:` Code style changes (formatting)
- `refactor:` Code refactoring
- `test:` Adding tests
- `chore:` Maintenance tasks

**Examples**:
```bash
git commit -m "feat: Add course enrollment feature"
git commit -m "fix: Correct material visibility check"
git commit -m "docs: Update API documentation"
```

## Code Style and Conventions

### Backend (Java)

**Naming Conventions**:
- Classes: `PascalCase` (e.g., `CourseService`)
- Methods: `camelCase` (e.g., `getCourses`)
- Constants: `UPPER_SNAKE_CASE` (e.g., `MAX_FILE_SIZE`)
- Packages: `lowercase` (e.g., `org.scikeys.cms.courses`)

**File Organization**:
```
package/
├── controller/    # REST controllers
├── service/       # Business logic
├── repository/    # Data access
├── entity/        # JPA entities
└── dto/          # Data transfer objects
```

**Annotations**:
- Use constructor injection (not field injection)
- Use `@RequiredArgsConstructor` from Lombok
- Keep controllers thin (delegate to services)

### Frontend (TypeScript/Angular)

**Naming Conventions**:
- Classes: `PascalCase` (e.g., `CourseService`)
- Variables/Functions: `camelCase` (e.g., `getCourses`)
- Constants: `UPPER_SNAKE_CASE` (e.g., `API_URL`)
- Files: `kebab-case` (e.g., `course-service.ts`)

**Component Structure**:
```typescript
// 1. Imports
import { Component, inject } from '@angular/core';

// 2. Component decorator
@Component({...})

// 3. Class
export class ComponentName {
  // Properties (signals)
  data = signal<Type[]>([]);
  
  // Injected services
  private service = inject(Service);
  
  // Methods
  loadData() { ... }
}
```

**File Organization**:
```
feature/
├── component.ts        # Component logic
├── component.html      # Template (if separate)
└── component.css       # Styles (if separate)
```

## Testing

### Backend Testing

**Unit Tests**:
```java
@ExtendWith(MockitoExtension.class)
class CourseServiceTest {
    @Mock
    private CourseRepository courseRepository;
    
    @InjectMocks
    private CourseService courseService;
    
    @Test
    void shouldGetCourseById() {
        // Given
        UUID courseId = UUID.randomUUID();
        Course course = new Course();
        when(courseRepository.findById(courseId)).thenReturn(Optional.of(course));
        
        // When
        CourseResponse response = courseService.getCourse(courseId);
        
        // Then
        assertNotNull(response);
        // Add more assertions
    }
}
```

**Integration Tests**:
```java
@SpringBootTest
@AutoConfigureMockMvc
class CourseControllerIntegrationTest {
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    void shouldGetCourses() throws Exception {
        mockMvc.perform(get("/api/v1/courses")
                .header("Authorization", "Bearer " + token))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$").isArray());
    }
}
```

### Frontend Testing

**Component Tests**:
```typescript
import { ComponentFixture, TestBed } from '@angular/core/testing';

describe('CourseListComponent', () => {
  let component: CourseListComponent;
  let fixture: ComponentFixture<CourseListComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [CourseListComponent]
    }).compileComponents();
    
    fixture = TestBed.createComponent(CourseListComponent);
    component = fixture.componentInstance;
  });

  it('should create', () => {
    expect(component).toBeTruthy();
  });
});
```

## Debugging

### Backend Debugging

**IDE Debugging** (IntelliJ IDEA):
1. Set breakpoints in code
2. Run → Debug 'CmsApplication'
3. Step through code, inspect variables

**Logging**:
```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

public class CourseService {
    private static final Logger log = LoggerFactory.getLogger(CourseService.class);
    
    public void createCourse(CourseCreateRequest request) {
        log.debug("Creating course: {}", request.getTitle());
        // ...
        log.info("Course created successfully: {}", course.getId());
    }
}
```

**View Logs**:
```bash
# Backend logs in console
# Or check application logs
tail -f logs/application.log
```

### Frontend Debugging

**Browser DevTools**:
1. Open DevTools (F12)
2. Console tab: View errors, log messages
3. Network tab: Inspect API requests
4. Sources tab: Set breakpoints

**Console Logging**:
```typescript
console.log('Debug:', data);
console.error('Error:', error);
console.warn('Warning:', message);
```

**Angular DevTools**:
- Install Angular DevTools browser extension
- Inspect component tree, state, performance

## Hot Reload

### Backend (Spring Boot DevTools)

**Automatic Restart**:
- Changes to Java files trigger automatic restart
- Configured in `pom.xml` (devtools dependency)

**Live Reload** (browser):
- Install LiveReload browser extension
- Backend changes trigger browser refresh

### Frontend (Angular CLI)

**Automatic Reload**:
- `npm start` enables hot reload
- Changes to TypeScript/HTML/CSS trigger browser update
- No page reload needed (instant updates)

## Common Development Tasks

### Adding a New Feature

**Backend**:
1. Create entity (if needed)
2. Create repository interface
3. Create DTOs (request/response)
4. Create service with business logic
5. Create controller with endpoints
6. Add validation
7. Write tests

**Frontend**:
1. Create models/interfaces
2. Create service (extends ApiService)
3. Create component
4. Add route
5. Update navigation (if needed)
6. Style component

### Adding a New Database Table

1. Create migration file: `V{N}__description.sql`
2. Define table structure
3. Run application (migration applies automatically)
4. Create entity class
5. Create repository interface
6. Update services/controllers as needed

### Modifying Existing Feature

1. Understand current implementation
2. Make changes incrementally
3. Test after each change
4. Update tests if needed
5. Update documentation

## Performance Considerations

### Backend

- **Database Queries**: Use indexes, avoid N+1 queries
- **Caching**: Cache frequently accessed data
- **Connection Pooling**: Already configured (HikariCP)
- **Lazy Loading**: Use LAZY fetch for relationships

### Frontend

- **Lazy Loading**: Load components on demand
- **Image Optimization**: Compress images
- **Bundle Size**: Keep dependencies minimal
- **Change Detection**: Use signals for better performance

## Code Review Guidelines

### What to Review

- **Functionality**: Does it work as intended?
- **Code Quality**: Is code clean and maintainable?
- **Testing**: Are tests adequate?
- **Security**: Any security concerns?
- **Performance**: Any performance issues?
- **Documentation**: Is code documented?

### Review Checklist

- [ ] Code follows style guidelines
- [ ] Tests pass
- [ ] No hardcoded secrets
- [ ] Error handling is appropriate
- [ ] Security considerations addressed
- [ ] Documentation updated if needed

## Troubleshooting

### Common Issues

**Backend won't start**:
- Check database connection
- Check port availability
- Check logs for errors

**Frontend build fails**:
- Delete `node_modules` and `package-lock.json`
- Run `npm install`
- Check TypeScript errors

**Database migration fails**:
- Check migration SQL syntax
- Verify database connection
- Check Flyway migration history

See [13_TROUBLESHOOTING.md](./13_TROUBLESHOOTING.md) for more.

## Best Practices

1. **Write Clean Code**: Readable, maintainable
2. **Test Your Code**: Unit and integration tests
3. **Handle Errors**: Proper error handling
4. **Document**: Code comments where needed
5. **Review**: Code reviews before merging
6. **Refactor**: Improve code quality over time
7. **Security**: Always consider security implications
8. **Performance**: Consider performance impact

## Next Steps

- Read [08_API_DOCUMENTATION.md](./08_API_DOCUMENTATION.md) for API details
- Read [09_FRONTEND_GUIDE.md](./09_FRONTEND_GUIDE.md) for frontend patterns
- Read [10_BACKEND_GUIDE.md](./10_BACKEND_GUIDE.md) for backend patterns

---

**Document Version**: 1.0  
**Last Updated**: December 2024

