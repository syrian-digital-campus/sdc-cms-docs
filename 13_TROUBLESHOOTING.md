# Troubleshooting Guide

Common issues and solutions for SDC-CMS.

## Backend Issues

### "Connection to localhost:5432 refused"

**Problem**: Database is not running or not accessible.

**Solutions**:
1. Check if PostgreSQL container is running:
   ```bash
   docker compose ps postgres
   ```

2. Start PostgreSQL:
   ```bash
   cd sdc-cms-infra
   docker compose up -d postgres
   ```

3. Verify connection:
   ```bash
   docker exec -it sdc-cms-postgres psql -U postgres -d university_cms
   ```

4. Check database credentials in `application.yaml`

### "Port 8080 already in use"

**Problem**: Another application is using port 8080.

**Solutions**:
1. Find what's using the port:
   ```bash
   # Windows
   netstat -ano | findstr :8080
   
   # Linux/Mac
   lsof -i :8080
   ```

2. Either:
   - Stop the conflicting application, OR
   - Change backend port in `application.yaml`:
     ```yaml
     server:
       port: 8081
     ```

### "Schema validation: missing table [table_name]"

**Problem**: Database migrations haven't run.

**Solutions**:
1. Check if Flyway migrations ran (check logs)
2. Verify migration files exist in `src/main/resources/db/migration/`
3. Manually run migrations if needed:
   ```bash
   ./mvnw flyway:migrate
   ```

### "User not found" after initialization

**Problem**: DataInitializer didn't run or database was reset.

**Solutions**:
1. Check if DataInitializer ran (check logs for "Initializing test data...")
2. Reset database and restart:
   ```bash
   docker compose down -v
   docker compose up -d postgres
   # Restart backend
   ```

### JWT Token Errors

**Problem**: "Invalid token" or "Token expired" errors.

**Solutions**:
1. Check JWT secret in `application.yaml` matches
2. Verify token format in request header: `Authorization: Bearer <token>`
3. Check token expiration time
4. Try logging in again to get new token

## Frontend Issues

### "Cannot find module" errors

**Problem**: Dependencies not installed.

**Solutions**:
```bash
rm -rf node_modules package-lock.json
npm install
```

### Build fails with TypeScript errors

**Problem**: Type errors in code.

**Solutions**:
1. Check TypeScript errors in terminal
2. Fix type mismatches
3. Ensure all imports are correct
4. Run `npm run build` to see detailed errors

### API requests return 401 Unauthorized

**Problem**: Token not being sent or expired.

**Solutions**:
1. Check if user is logged in
2. Verify token is in request headers (check Network tab)
3. Check AuthInterceptor is working
4. Try refreshing the page (may trigger token refresh)
5. Log in again

### CORS Errors

**Problem**: "Access-Control-Allow-Origin" errors.

**Solutions**:
1. Verify backend CORS configuration in `SecurityConfig.java`
2. Check if frontend URL is in allowed origins
3. Verify API URL in `environment.ts` matches backend URL

### Page shows blank or errors

**Problem**: Runtime errors in browser.

**Solutions**:
1. Open browser DevTools (F12)
2. Check Console tab for errors
3. Check Network tab for failed requests
4. Check Sources tab for breakpoints
5. Clear browser cache and reload

## Database Issues

### Migration fails

**Problem**: SQL syntax error or constraint violation.

**Solutions**:
1. Check migration file for syntax errors
2. Verify SQL is valid PostgreSQL
3. Check if migration conflicts with existing schema
4. Review Flyway logs for specific error

### Database locked or busy

**Problem**: Multiple connections or transactions blocking.

**Solutions**:
1. Check for long-running transactions
2. Restart database:
   ```bash
   docker compose restart postgres
   ```
3. Check connection pool settings

### Data not persisting

**Problem**: Changes not saved to database.

**Solutions**:
1. Check if `@Transactional` is used correctly
2. Verify repository methods are being called
3. Check database logs for INSERT/UPDATE statements
4. Verify transaction is committing (not rolling back)

## Docker Issues

### Container won't start

**Problem**: Container exits immediately.

**Solutions**:
1. Check logs:
   ```bash
   docker compose logs <service-name>
   ```
2. Verify Dockerfile is correct
3. Check environment variables
4. Verify port mappings

### Images not building

**Problem**: Build fails during docker build.

**Solutions**:
1. Check Dockerfile syntax
2. Verify base images exist
3. Check build context includes necessary files
4. Review build logs for specific errors

### Volume permission issues

**Problem**: Cannot write to mounted volumes.

**Solutions**:
1. Check volume permissions
2. Use named volumes instead of bind mounts
3. Verify user permissions in container

## General Issues

### Application runs but doesn't work correctly

**Problem**: Logic or configuration issues.

**Solutions**:
1. Check application logs
2. Enable debug logging temporarily
3. Verify configuration values
4. Test API endpoints directly (Postman/curl)
5. Check browser console for frontend errors

### Slow performance

**Problem**: Application is slow.

**Solutions**:
1. Check database query performance
2. Look for N+1 query problems
3. Verify indexes exist on frequently queried columns
4. Check for unnecessary data loading
5. Monitor resource usage (CPU, memory)

### Feature not working

**Problem**: New feature or recent change broken.

**Solutions**:
1. Check if code was committed and deployed
2. Verify environment variables are set
3. Check if database migrations ran
4. Review recent changes in git history
5. Test in development environment first

## Getting Help

If issues persist:

1. **Check Logs**: Review application and database logs
2. **Review Documentation**: Check relevant documentation files
3. **Search Issues**: Check if issue was reported before
4. **Debug Mode**: Enable debug logging to see more details
5. **Isolate Problem**: Try to reproduce in minimal setup

## Common Error Messages

### Backend

- **"BeanCreationException"**: Dependency injection issue, check annotations
- **"EntityNotFoundException"**: Database entity not found, check ID
- **"ConstraintViolationException"**: Database constraint violation
- **"HttpRequestMethodNotSupportedException"**: Wrong HTTP method used

### Frontend

- **"NG0304: Can't bind to..."**: Property binding error, check template
- **"Cannot read property of undefined"**: Null/undefined value accessed
- **"HttpErrorResponse"**: HTTP request failed, check API

### Database

- **"relation does not exist"**: Table missing, run migrations
- **"duplicate key value violates unique constraint"**: Unique constraint violation
- **"foreign key constraint fails"**: Referenced row doesn't exist

## Prevention Tips

1. **Test Locally**: Always test changes locally first
2. **Check Logs**: Regularly review logs for warnings/errors
3. **Backup Database**: Regular backups before major changes
4. **Version Control**: Commit changes frequently
5. **Document Changes**: Document configuration changes
6. **Monitor Resources**: Watch CPU, memory, disk usage

---

**Document Version**: 1.0  
**Last Updated**: December 2024




