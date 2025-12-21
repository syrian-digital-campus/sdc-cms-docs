# Angular Guide for Beginners

This guide explains Angular concepts and how they're used in SDC-CMS for developers new to Angular.

## What is Angular?

Angular is a TypeScript-based web framework for building single-page applications (SPAs). It provides:

- **Component-Based Architecture**: Build UI with reusable components
- **Dependency Injection**: Automatic dependency management
- **Two-Way Data Binding**: Automatic synchronization between model and view
- **Routing**: Navigate between pages without full page reload
- **RxJS Integration**: Powerful reactive programming

## Core Concepts

### 1. Components

**What is a Component?**
A component is a building block of an Angular application. It consists of:
- **TypeScript Class**: Contains logic and data
- **HTML Template**: Defines the view
- **CSS Styles**: Styles the component

**Example Component**:

```typescript
import { Component } from '@angular/core';
import { CommonModule } from '@angular/common';

@Component({
  selector: 'app-course-list',  // How to use: <app-course-list></app-course-list>
  standalone: true,  // Standalone component (modern Angular)
  imports: [CommonModule],  // Import Angular modules needed
  template: `
    <div>
      <h1>Courses</h1>
      @for (course of courses(); track course.id) {
        <div>{{ course.title }}</div>
      }
    </div>
  `,
  styles: [`
    h1 { color: blue; }
  `]
})
export class CourseListComponent {
  // Component logic
  courses = signal<Course[]>([]);
}
```

**Key Points**:
- `selector`: HTML tag name for the component
- `standalone: true`: Modern Angular (no NgModule needed)
- `imports`: What Angular modules/directives this component needs
- `template`: HTML markup (inline) or `templateUrl` for external file
- `styles`: CSS styles (inline) or `styleUrls` for external file

### 2. Standalone Components (Modern Angular)

**What are Standalone Components?**
In Angular 14+, components can be standalone (don't need NgModules). SDC-CMS uses standalone components.

**Benefits**:
- Simpler: No need to declare in NgModule
- Better tree-shaking: Only import what you need
- Easier to understand: Dependencies are explicit

**Example**:
```typescript
@Component({
  selector: 'app-course-list',
  standalone: true,  // Standalone component
  imports: [CommonModule, RouterLink],  // Explicitly import what you need
  // ...
})
```

### 3. Signals (Modern Angular)

**What are Signals?**
Signals are reactive primitives that hold a value and notify when it changes.

**Why Signals?**
- Better performance: Fine-grained reactivity
- Simpler code: No need for `Observable` subscriptions
- Better change detection

**Example**:
```typescript
import { signal } from '@angular/core';

export class CourseListComponent {
  // Create a signal
  courses = signal<Course[]>([]);
  loading = signal(false);
  
  loadCourses() {
    this.loading.set(true);  // Update signal
    
    this.courseService.getCourses().subscribe({
      next: (courses) => {
        this.courses.set(courses);  // Update signal
        this.loading.set(false);
      }
    });
  }
}
```

**In Template**:
```html
@if (loading()) {  <!-- Call signal as function -->
  <div>Loading...</div>
}
@for (course of courses(); track course.id) {
  <div>{{ course.title }}</div>
}
```

### 4. Services

**What is a Service?**
A service is a class that provides functionality that can be shared across components.

**Why Services?**
- **Separation of Concerns**: Business logic separate from UI
- **Reusability**: Use same service in multiple components
- **Singleton**: One instance shared across app (by default)

**Example Service**:

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'  // Available app-wide (singleton)
})
export class CourseService {
  private http = inject(HttpClient);
  private apiUrl = 'http://localhost:8080/api/v1';
  
  getCourses(): Observable<Course[]> {
    return this.http.get<Course[]>(`${this.apiUrl}/courses`);
  }
  
  getCourse(id: string): Observable<Course> {
    return this.http.get<Course>(`${this.apiUrl}/courses/${id}`);
  }
}
```

**Using a Service in Component**:

```typescript
export class CourseListComponent {
  private courseService = inject(CourseService);  // Inject service
  
  courses = signal<Course[]>([]);
  
  ngOnInit() {
    this.courseService.getCourses().subscribe({
      next: (courses) => this.courses.set(courses)
    });
  }
}
```

### 5. Dependency Injection

**What is Dependency Injection?**
Angular automatically provides dependencies (services) to components that need them.

**Two Ways to Inject**:

**1. Constructor Injection (Traditional)**:
```typescript
export class CourseListComponent {
  constructor(private courseService: CourseService) {}
}
```

**2. inject() Function (Modern)**:
```typescript
import { inject } from '@angular/core';

export class CourseListComponent {
  private courseService = inject(CourseService);
}
```

**Why inject()?**
- Works better with signals
- Can be used in functions
- Cleaner code

### 6. HTTP Client and Observables

**HTTP Client**:
Angular's `HttpClient` makes HTTP requests and returns Observables.

**Observables**:
Observables represent streams of data. You subscribe to them to get values.

**Example**:
```typescript
// Service
getCourses(): Observable<Course[]> {
  return this.http.get<Course[]>('/api/v1/courses');
}

// Component
this.courseService.getCourses().subscribe({
  next: (courses) => {
    // Handle success
    this.courses.set(courses);
  },
  error: (error) => {
    // Handle error
    console.error('Failed to load courses', error);
  },
  complete: () => {
    // Called when observable completes
  }
});
```

**RxJS Operators**:
Transform data with operators:

```typescript
import { map, catchError } from 'rxjs/operators';
import { of } from 'rxjs';

this.courseService.getCourses()
  .pipe(
    map(courses => courses.filter(c => c.isActive)),  // Transform
    catchError(error => {
      console.error(error);
      return of([]);  // Return empty array on error
    })
  )
  .subscribe(courses => this.courses.set(courses));
```

### 7. Routing

**What is Routing?**
Routing allows navigation between different views without full page reload.

**Route Configuration**:

```typescript
// app.routes.ts
import { Routes } from '@angular/router';

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
  },
  {
    path: '',
    redirectTo: '/courses',
    pathMatch: 'full'
  }
];
```

**Navigation in Template**:
```html
<a routerLink="/courses">Courses</a>
<a [routerLink]="['/courses', course.slug]">{{ course.title }}</a>
```

**Navigation in Code**:
```typescript
import { Router } from '@angular/router';

export class CourseComponent {
  private router = inject(Router);
  
  goToCourses() {
    this.router.navigate(['/courses']);
  }
  
  goToCourse(slug: string) {
    this.router.navigate(['/courses', slug]);
  }
}
```

**Route Parameters**:
```typescript
import { ActivatedRoute } from '@angular/router';

export class CourseDetailComponent {
  private route = inject(ActivatedRoute);
  
  ngOnInit() {
    // Get route parameter
    const slug = this.route.snapshot.paramMap.get('slug');
    
    // Or subscribe to changes
    this.route.paramMap.subscribe(params => {
      const slug = params.get('slug');
      this.loadCourse(slug);
    });
  }
}
```

### 8. Route Guards

**What are Guards?**
Guards control access to routes (e.g., require authentication).

**Example Guard**:

```typescript
import { inject } from '@angular/core';
import { Router, CanActivateFn } from '@angular/router';
import { AuthService } from '../services/auth.service';

export const authGuard: CanActivateFn = (route, state) => {
  const authService = inject(AuthService);
  const router = inject(Router);
  
  if (authService.isAuthenticated()) {
    return true;  // Allow access
  } else {
    router.navigate(['/login']);
    return false;  // Block access
  }
};
```

**Using Guard**:
```typescript
{
  path: 'dashboard',
  canActivate: [authGuard],  // Protect route
  loadComponent: () => import('./dashboard.component')
}
```

### 9. HTTP Interceptors

**What are Interceptors?**
Interceptors can modify HTTP requests/responses globally.

**Example: Auth Interceptor**:

```typescript
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { AuthService } from './auth.service';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
  const authService = inject(AuthService);
  const token = authService.getAccessToken();
  
  if (token) {
    // Add Authorization header
    req = req.clone({
      setHeaders: {
        Authorization: `Bearer ${token}`
      }
    });
  }
  
  return next(req);
};
```

**Register Interceptor**:
```typescript
// app.config.ts
export const appConfig: ApplicationConfig = {
  providers: [
    provideHttpClient(
      withInterceptors([authInterceptor])  // Register interceptor
    )
  ]
};
```

### 10. Forms

**Reactive Forms** (Recommended):

```typescript
import { FormBuilder, FormGroup, Validators, ReactiveFormsModule } from '@angular/forms';

export class LoginComponent {
  private fb = inject(FormBuilder);
  
  loginForm: FormGroup = this.fb.group({
    username: ['', Validators.required],
    password: ['', [Validators.required, Validators.minLength(6)]]
  });
  
  onSubmit() {
    if (this.loginForm.valid) {
      const { username, password } = this.loginForm.value;
      // Submit form
    }
  }
}
```

**Template**:
```html
<form [formGroup]="loginForm" (ngSubmit)="onSubmit()">
  <input formControlName="username" 
         [class.error]="loginForm.get('username')?.invalid && loginForm.get('username')?.touched">
  @if (loginForm.get('username')?.invalid && loginForm.get('username')?.touched) {
    <span class="error">Username is required</span>
  }
  
  <input type="password" formControlName="password">
  
  <button type="submit" [disabled]="loginForm.invalid">Login</button>
</form>
```

### 11. Template Syntax

**Control Flow (New in Angular 17+)**:
```html
<!-- @if / @else -->
@if (loading()) {
  <div>Loading...</div>
} @else {
  <div>Loaded!</div>
}

<!-- @for -->
@for (course of courses(); track course.id) {
  <div>{{ course.title }}</div>
} @empty {
  <div>No courses found</div>
}

<!-- @switch -->
@switch (status()) {
  @case ('active') {
    <span>Active</span>
  }
  @case ('inactive') {
    <span>Inactive</span>
  }
  @default {
    <span>Unknown</span>
  }
}
```

**Property Binding**:
```html
<img [src]="imageUrl()">
<button [disabled]="isDisabled()">
<div [class.active]="isActive()">
```

**Event Binding**:
```html
<button (click)="handleClick()">Click</button>
<input (input)="onInput($event)">
```

**Two-Way Binding**:
```html
<input [(ngModel)]="username">
```

### 12. Component Communication

**Parent → Child (Input)**:
```typescript
// Parent Component
<app-course-card [course]="selectedCourse()"></app-course-card>

// Child Component
@Component({...})
export class CourseCardComponent {
  @Input() course!: Course;  // Receive from parent
}
```

**Child → Parent (Output)**:
```typescript
// Child Component
@Output() courseSelected = new EventEmitter<Course>();

selectCourse() {
  this.courseSelected.emit(this.course);
}

// Parent Template
<app-course-card (courseSelected)="onCourseSelected($event)"></app-course-card>
```

**Through Services (Shared State)**:
```typescript
// Service holds shared state
@Injectable({ providedIn: 'root' })
export class CourseService {
  private selectedCourse = signal<Course | null>(null);
  
  selectCourse(course: Course) {
    this.selectedCourse.set(course);
  }
  
  getSelectedCourse() {
    return this.selectedCourse.asReadonly();
  }
}

// Components use service
export class ComponentA {
  private courseService = inject(CourseService);
  
  selectCourse(course: Course) {
    this.courseService.selectCourse(course);
  }
}

export class ComponentB {
  private courseService = inject(CourseService);
  selectedCourse = this.courseService.getSelectedCourse();
}
```

## Project Structure in SDC-CMS

```
src/app/
├── core/                    # Core functionality (singleton services)
│   ├── guards/             # Route guards
│   ├── interceptors/       # HTTP interceptors
│   ├── models/             # TypeScript interfaces
│   └── services/           # Shared services
│
├── features/               # Feature modules
│   ├── auth/              # Authentication
│   ├── courses/           # Course management
│   ├── materials/         # Materials
│   └── assignments/       # Assignments
│
├── shared/                # Shared components
│   └── components/
│       └── navbar/
│
├── app.component.ts       # Root component
├── app.routes.ts          # Route configuration
└── app.config.ts          # App configuration
```

## Common Patterns in SDC-CMS

### Pattern 1: Service Extending ApiService

```typescript
// Base API service
export class ApiService {
  protected http = inject(HttpClient);
  protected apiUrl = environment.apiUrl;
  
  protected get<T>(endpoint: string): Observable<T> {
    return this.http.get<T>(`${this.apiUrl}${endpoint}`);
  }
}

// Feature service extends base
@Injectable({ providedIn: 'root' })
export class CourseService extends ApiService {
  getCourses(): Observable<Course[]> {
    return this.get<Course[]>('/courses');
  }
}
```

### Pattern 2: Component with Signal State

```typescript
export class CourseListComponent {
  courses = signal<Course[]>([]);
  loading = signal(false);
  error = signal<string | null>(null);
  
  loadCourses() {
    this.loading.set(true);
    this.error.set(null);
    
    this.courseService.getCourses().subscribe({
      next: (courses) => {
        this.courses.set(courses);
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

### Pattern 3: Route Parameter Loading

```typescript
export class CourseDetailComponent implements OnInit {
  private route = inject(ActivatedRoute);
  course = signal<Course | null>(null);
  
  ngOnInit() {
    this.route.paramMap.subscribe(params => {
      const slug = params.get('slug');
      if (slug) {
        this.loadCourse(slug);
      }
    });
  }
  
  loadCourse(slug: string) {
    this.courseService.getCourseBySlug(slug).subscribe({
      next: (course) => this.course.set(course)
    });
  }
}
```

## Best Practices

1. **Use Standalone Components**: Modern Angular pattern
2. **Use Signals**: For reactive state management
3. **Use Services for Business Logic**: Keep components thin
4. **Use TypeScript**: Strong typing prevents errors
5. **Lazy Load Routes**: Load components on demand
6. **Handle Errors**: Always handle errors in subscriptions
7. **Unsubscribe**: Or use async pipe (handles subscription automatically)
8. **Use Guards**: Protect routes that require authentication
9. **Use Interceptors**: For cross-cutting concerns (auth, errors)

## Next Steps

- Practice with small examples
- Read Angular documentation: https://angular.dev
- Explore SDC-CMS codebase to see patterns in action
- Read [09_FRONTEND_GUIDE.md](./09_FRONTEND_GUIDE.md) for SDC-CMS specific patterns

---

**Document Version**: 1.0  
**Last Updated**: December 2024

