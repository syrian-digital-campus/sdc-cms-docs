# Security Guide

This document outlines the security measures implemented in SDC-CMS and best practices for maintaining security.

## Security Architecture

### Authentication Flow

SDC-CMS uses **JWT (JSON Web Tokens)** for stateless authentication:

1. **Login**: User submits credentials → Backend validates → Returns access token + refresh token
2. **Access Token**: Short-lived (15 minutes), stored in memory (frontend)
3. **Refresh Token**: Long-lived (7 days), stored in HttpOnly cookie
4. **Token Refresh**: When access token expires, frontend uses refresh token to get new access token

### Why This Approach?

- **Stateless**: No server-side session storage needed
- **Scalable**: Works across multiple servers
- **Secure**: HttpOnly cookies prevent XSS attacks on refresh tokens
- **Performance**: No database lookup for each request

## Security Measures

### 1. Password Security

**Hashing Algorithm**: BCrypt
- **Why BCrypt?**: Designed to be slow, resistant to brute force attacks
- **Salt**: Automatically generated unique salt per password
- **Cost Factor**: Can be increased over time (default: 10)

**Implementation**:
```java
// Password hashing
String hashedPassword = BCrypt.hashpw(plainPassword, BCrypt.gensalt());

// Password verification
boolean isValid = BCrypt.checkpw(plainPassword, hashedPassword);
```

**Best Practices**:
- Never store plain text passwords
- Never log passwords
- Use strong password requirements (min length, complexity)
- Implement password reset flow (not just password change)

### 2. JWT Token Security

**Access Token**:
- Short expiry (15 minutes)
- Contains user ID and roles
- Stored in memory (JavaScript, not localStorage)
- Sent in `Authorization: Bearer <token>` header

**Refresh Token**:
- Long expiry (7 days)
- Stored in HttpOnly cookie
- Not accessible via JavaScript (prevents XSS)
- Used only for getting new access tokens

**Token Structure**:
```json
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "user-id",
    "username": "student1",
    "roles": ["STUDENT"],
    "iat": 1234567890,
    "exp": 1234567890
  },
  "signature": "HMACSHA256(...)"
}
```

### 3. HTTPS/SSL

**In Production**:
- Always use HTTPS
- Terminate SSL at load balancer or nginx
- Use strong cipher suites
- Enable HSTS (HTTP Strict Transport Security)

**Configuration** (nginx example):
```nginx
server {
    listen 443 ssl http2;
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    
    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
}
```

### 4. CORS (Cross-Origin Resource Sharing)

**Configuration**:
- Frontend origin allowed: `http://localhost:4200` (dev), `https://yourdomain.com` (prod)
- Credentials: `true` (for cookies)
- Methods: GET, POST, PUT, DELETE, OPTIONS
- Headers: Authorization, Content-Type

**Why Important?**
Prevents unauthorized websites from making requests to your API.

### 5. Input Validation

**Backend Validation**:
- Use Jakarta Bean Validation (`@NotNull`, `@Size`, `@Email`, etc.)
- Validate all user input
- Sanitize file uploads
- Prevent SQL injection (JPA uses parameterized queries)

**Example**:
```java
@PostMapping("/users")
public ResponseEntity<UserResponse> createUser(@Valid @RequestBody UserCreateRequest request) {
    // @Valid ensures request is validated before method executes
    // If validation fails, 400 Bad Request is returned
}
```

**Frontend Validation**:
- Client-side validation for better UX
- **Never trust client-side validation alone**
- Always validate on backend

### 6. SQL Injection Prevention

**How We Prevent It**:
- **JPA/Hibernate**: Uses parameterized queries automatically
- **No raw SQL**: Avoid using raw SQL queries
- **If raw SQL needed**: Always use parameterized queries

**Example** (Safe):
```java
// JPA automatically uses parameterized queries
List<User> users = userRepository.findByUsername(username);

// If custom query needed, use parameters
@Query("SELECT u FROM User u WHERE u.username = :username")
Optional<User> findByUsername(@Param("username") String username);
```

**Example** (Unsafe - DON'T DO THIS):
```java
// NEVER do this - vulnerable to SQL injection
String query = "SELECT * FROM users WHERE username = '" + username + "'";
```

### 7. File Upload Security

**Measures**:
- **File Type Validation**: Check file extensions and MIME types
- **File Size Limits**: Prevent DoS attacks
- **Virus Scanning**: Recommended in production
- **Secure Storage**: Store outside web root
- **Path Traversal Prevention**: Validate file paths

**Implementation**:
```java
// Validate file type
if (!allowedMimeTypes.contains(file.getContentType())) {
    throw new InvalidFileTypeException();
}

// Validate file size
if (file.getSize() > MAX_FILE_SIZE) {
    throw new FileTooLargeException();
}

// Sanitize filename
String sanitizedFilename = sanitizeFilename(file.getOriginalFilename());
```

### 8. Authorization (Access Control)

**Role-Based Access Control (RBAC)**:

1. **Global Roles**: ADMIN, PROFESSOR, STUDENT
2. **Context Roles**: Course-specific roles (PROFESSOR, TA, TUTOR, STUDENT)

**Implementation**:
```java
// Method-level security
@PreAuthorize("hasRole('ADMIN') or hasRole('PROFESSOR')")
public void createCourse(CourseCreateRequest request) {
    // Only admins and professors can create courses
}

// Context-based authorization
if (!user.isCourseOwner(courseId) && !user.hasRole("ADMIN")) {
    throw new ForbiddenException("Not authorized");
}
```

### 9. XSS (Cross-Site Scripting) Prevention

**Measures**:
- **Output Encoding**: Encode all user-generated content
- **Content Security Policy (CSP)**: Restrict script execution
- **HttpOnly Cookies**: Prevent JavaScript access to cookies
- **Angular Sanitization**: Angular automatically sanitizes HTML

**CSP Header** (nginx):
```nginx
add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline';";
```

### 10. CSRF (Cross-Site Request Forgery) Protection

**Current Status**: 
- CSRF protection is enabled for state-changing operations
- JWT tokens provide additional protection

**How It Works**:
- Spring Security generates CSRF token
- Frontend includes token in requests
- Backend validates token

### 11. Rate Limiting

**Recommended in Production**:
- Limit login attempts (prevent brute force)
- Limit API requests per user/IP
- Use Spring Security Rate Limiting or external service (Redis)

**Example** (using Spring):
```java
@RateLimiter(name = "default")
public ResponseEntity<?> login(@RequestBody LoginRequest request) {
    // Limited to X requests per minute
}
```

### 12. Security Headers

**Recommended Headers**:
- **X-Frame-Options**: Prevent clickjacking
- **X-Content-Type-Options**: Prevent MIME type sniffing
- **X-XSS-Protection**: Enable browser XSS filter
- **Strict-Transport-Security**: Force HTTPS
- **Content-Security-Policy**: Restrict resource loading

**Configuration** (Spring Security):
```java
http.headers()
    .frameOptions().deny()
    .contentTypeOptions().and()
    .httpStrictTransportSecurity(hsts -> hsts
        .maxAgeInSeconds(31536000)
        .includeSubdomains(true));
```

## Security Best Practices

### For Developers

1. **Never commit secrets**: Use environment variables
2. **Keep dependencies updated**: Regularly update Spring Boot, libraries
3. **Follow principle of least privilege**: Users should have minimum necessary permissions
4. **Log security events**: Log failed login attempts, authorization failures
5. **Review code**: Peer review security-sensitive code
6. **Use HTTPS in production**: Never transmit sensitive data over HTTP

### For Deployment

1. **Environment Variables**: Store secrets (JWT secret, DB passwords) as environment variables
2. **Firewall**: Restrict access to database
3. **Backup**: Regular database backups
4. **Monitoring**: Monitor for suspicious activity
5. **Updates**: Keep server and dependencies updated
6. **Access Logs**: Enable and monitor access logs

### Common Vulnerabilities to Avoid

1. **SQL Injection**: Always use parameterized queries
2. **XSS**: Encode user input, use CSP
3. **CSRF**: Use CSRF tokens
4. **Session Fixation**: Use secure session management
5. **Insecure Direct Object References**: Check authorization before access
6. **Security Misconfiguration**: Review all security settings
7. **Sensitive Data Exposure**: Encrypt sensitive data at rest
8. **Broken Authentication**: Use secure password hashing, token management

## Security Checklist

### Development
- [ ] All passwords hashed with BCrypt
- [ ] JWT tokens properly configured
- [ ] Input validation on all endpoints
- [ ] Authorization checks in place
- [ ] File uploads validated
- [ ] No hardcoded secrets
- [ ] CORS properly configured
- [ ] Security headers set

### Production
- [ ] HTTPS enabled
- [ ] Strong JWT secret (256-bit)
- [ ] Database password changed from default
- [ ] Environment variables for secrets
- [ ] Rate limiting enabled
- [ ] Monitoring/logging enabled
- [ ] Regular backups
- [ ] Firewall rules configured
- [ ] Dependencies updated
- [ ] Security headers configured

## Incident Response

If a security breach is detected:

1. **Immediate**: Revoke affected tokens, change secrets
2. **Investigate**: Review logs, identify scope
3. **Notify**: Inform affected users if necessary
4. **Fix**: Patch vulnerability
5. **Prevent**: Implement measures to prevent recurrence

## Security Testing

### Manual Testing

1. **Authentication**: Test login, logout, token refresh
2. **Authorization**: Test role-based access
3. **Input Validation**: Test with malicious input
4. **File Upload**: Test with various file types, sizes

### Automated Testing

- **OWASP ZAP**: Security vulnerability scanner
- **Dependency Check**: Check for vulnerable dependencies
- **Penetration Testing**: Regular security audits

## Additional Resources

- **OWASP Top 10**: https://owasp.org/www-project-top-ten/
- **Spring Security Documentation**: https://spring.io/projects/spring-security
- **JWT Best Practices**: https://datatracker.ietf.org/doc/html/rfc8725

## Next Steps

- Review security configuration regularly
- Keep dependencies updated
- Monitor for security advisories
- Conduct regular security audits
- Train team on security best practices

---

**Document Version**: 1.0  
**Last Updated**: December 2024




