# Frontend Development Guide

SDC-CMS specific frontend development patterns and practices.

## Project Structure

```
src/app/
├── core/                    # Core functionality (singleton)
│   ├── guards/             # Route guards
│   ├── interceptors/       # HTTP interceptors
│   ├── models/             # TypeScript interfaces
│   └── services/           # Shared services
├── features/               # Feature modules
│   ├── auth/
│   ├── courses/
│   ├── materials/
│   └── assignments/
├── shared/                 # Shared components
└── layouts/                # Layout components
```

## Service Pattern

### Base ApiService

All API services extend `ApiService`:

```typescript
@Injectable({ providedIn: 'root' })
export class ApiService {
  protected http = inject(HttpClient);
  protected apiUrl = environment.apiUrl;
  
  protected get<T>(endpoint: string): Observable<T> {
    return this.http.get<T>(`${this.apiUrl}${endpoint}`);
  }
  
  // post, put, delete methods...
}
```

### Feature Service

```typescript
@Injectable({ providedIn: 'root' })
export class CourseService extends ApiService {
  getCourses(): Observable<Course[]> {
    return this.get<Course[]>('/courses');
  }
  
  getCourse(id: string): Observable<Course> {
    return this.get<Course>(`/courses/${id}`);
  }
  
  createCourse(request: CourseCreateRequest): Observable<Course> {
    return this.post<Course>('/courses', request);
  }
}
```

## Component Pattern

### Standard Component Structure

```typescript
@Component({
  selector: 'app-feature-list',
  standalone: true,
  imports: [CommonModule, ReactiveFormsModule],
  template: `...`,
  styles: [`...`]
})
export class FeatureListComponent implements OnInit {
  // Signals for reactive state
  items = signal<Item[]>([]);
  loading = signal(false);
  error = signal<string | null>(null);
  
  // Injected services
  private featureService = inject(FeatureService);
  private route = inject(ActivatedRoute);
  
  ngOnInit(): void {
    this.loadItems();
  }
  
  loadItems(): void {
    this.loading.set(true);
    this.error.set(null);
    
    this.featureService.getItems().subscribe({
      next: (items) => {
        this.items.set(items);
        this.loading.set(false);
      },
      error: (err) => {
        this.error.set(err.message);
        this.loading.set(false);
      }
    });
  }
}
```

## Authentication Flow

### AuthService

Manages authentication state:

```typescript
@Injectable({ providedIn: 'root' })
export class AuthService extends ApiService {
  currentUser = signal<UserInfo | null>(null);
  accessToken = signal<string | null>(null);
  
  login(credentials: LoginRequest): Observable<LoginResponse> {
    return this.post<LoginResponse>('/auth/login', credentials).pipe(
      tap(response => {
        this.accessToken.set(response.accessToken);
        // Store user info...
      })
    );
  }
  
  isAuthenticated(): boolean {
    return this.accessToken() !== null;
  }
  
  hasRole(role: string): boolean {
    return this.currentUser()?.profile.globalRole === role;
  }
}
```

### Auth Guard

```typescript
export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);
  
  if (authService.isAuthenticated()) {
    return true;
  } else {
    router.navigate(['/login']);
    return false;
  }
};
```

### Auth Interceptor

Adds JWT token to requests:

```typescript
export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getAccessToken();
  
  if (token) {
    req = req.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`
      }
    });
  }
  
  return next(req).pipe(
    catchError(error => {
      if (error.status === 401) {
        // Handle token refresh or redirect to login
      }
      return throwError(() => error);
    })
  );
};
```

## Forms Pattern

### Reactive Forms

```typescript
export class LoginComponent {
  private fb = inject(FormBuilder);
  private authService = inject(AuthService);
  
  loginForm: FormGroup = this.fb.group({
    username: ['', Validators.required],
    password: ['', [Validators.required, Validators.minLength(6)]]
  });
  
  onSubmit(): void {
    if (this.loginForm.valid) {
      const { username, password } = this.loginForm.value;
      this.authService.login({ username, password }).subscribe({
        next: () => this.router.navigate(['/courses']),
        error: (err) => this.error.set(err.message)
      });
    }
  }
}
```

## Routing Pattern

### Route Configuration

```typescript
export const routes: Routes = [
  {
    path: 'courses',
    loadComponent: () => import('./features/courses/courses.component')
      .then(m => m.CoursesComponent)
  },
  {
    path: 'courses/:slug',
    loadComponent: () => import('./features/courses/course-news.component')
      .then(m => m.CourseNewsComponent)
  }
];
```

### Route Parameters

```typescript
export class CourseDetailComponent implements OnInit {
  private route = inject(ActivatedRoute);
  course = signal<Course | null>(null);
  
  ngOnInit(): void {
    this.route.paramMap.subscribe(params => {
      const slug = params.get('slug');
      if (slug) {
        this.loadCourse(slug);
      }
    });
  }
}
```

## State Management

### Signals (Recommended)

Use signals for reactive state:

```typescript
export class Component {
  data = signal<Data[]>([]);
  loading = signal(false);
  
  loadData() {
    this.loading.set(true);
    this.service.getData().subscribe({
      next: (data) => {
        this.data.set(data);
        this.loading.set(false);
      }
    });
  }
}
```

In template:
```html
@if (loading()) {
  <div>Loading...</div>
}
@for (item of data(); track item.id) {
  <div>{{ item.name }}</div>
}
```

## Best Practices

1. **Use Signals**: For reactive state management
2. **Extend ApiService**: For consistent API calls
3. **Handle Errors**: Always handle errors in subscriptions
4. **Type Safety**: Use TypeScript interfaces for all data
5. **Component Decomposition**: Keep components focused and small
6. **Lazy Loading**: Load routes/components on demand
7. **Reactive Forms**: Use reactive forms for complex forms

## Next Steps

- Read [05_ANGULAR_GUIDE.md](./05_ANGULAR_GUIDE.md) for Angular basics
- Explore existing components for patterns
- See [08_API_DOCUMENTATION.md](./08_API_DOCUMENTATION.md) for API details

---

**Document Version**: 1.0  
**Last Updated**: December 2024

