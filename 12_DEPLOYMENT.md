# Deployment Guide

Guide for deploying SDC-CMS to production.

**End-to-end handoff (deliverables, subdomains, doc map):** [16_PRODUCTION_DEPLOY_AND_HANDOFF.md](./16_PRODUCTION_DEPLOY_AND_HANDOFF.md)

## Production Checklist

### Security
- [ ] Change JWT secret to strong random value
- [ ] Change database password
- [ ] Enable HTTPS/SSL
- [ ] Configure CORS properly
- [ ] Review security headers
- [ ] Enable rate limiting
- [ ] Remove test data
- [ ] Disable debug logging

### Configuration
- [ ] Set production environment variables
- [ ] Configure database connection
- [ ] Set file storage path
- [ ] Configure email (if needed)
- [ ] Set proper timezone

### Database
- [ ] Run migrations
- [ ] Create database backup
- [ ] Configure connection pooling
- [ ] Set up database backups

### Infrastructure
- [ ] Set up reverse proxy (nginx)
- [ ] Configure SSL certificates
- [ ] Set up monitoring
- [ ] Configure logging
- [ ] Set up error tracking

## Docker Deployment

### Build Images

```bash
# Backend
cd sdc-cms-backend
docker build -t sdc-cms-backend:latest .

# Frontend
cd sdc-cms-frontend
docker build -t sdc-cms-frontend:latest .
```

### Production docker-compose.yml

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    restart: unless-stopped
    networks:
      - sdc-cms-network

  backend:
    image: sdc-cms-backend:latest
    environment:
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/${POSTGRES_DB}
      SPRING_DATASOURCE_USERNAME: ${POSTGRES_USER}
      SPRING_DATASOURCE_PASSWORD: ${POSTGRES_PASSWORD}
      JWT_SECRET: ${JWT_SECRET}
      FILE_STORAGE_PATH: /data/files
    volumes:
      - backend_files:/data/files
    depends_on:
      - postgres
    restart: unless-stopped
    networks:
      - sdc-cms-network

  frontend:
    image: sdc-cms-frontend:latest
    depends_on:
      - backend
    restart: unless-stopped
    networks:
      - sdc-cms-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - frontend
      - backend
    restart: unless-stopped
    networks:
      - sdc-cms-network

volumes:
  postgres_data:
  backend_files:

networks:
  sdc-cms-network:
```

## Environment Variables

### Backend (.env)

```bash
SPRING_DATASOURCE_URL=jdbc:postgresql://postgres:5432/university_cms
SPRING_DATASOURCE_USERNAME=postgres
SPRING_DATASOURCE_PASSWORD=strong-password-here
JWT_SECRET=your-256-bit-secret-key-here
FILE_STORAGE_PATH=/data/files
```

### Frontend

Build with production environment:

```typescript
// environment.prod.ts
export const environment = {
  production: true,
  apiUrl: '/api/v1'  // Relative URL (nginx proxies)
};
```

## Nginx Configuration

### nginx.conf

```nginx
upstream backend {
    server backend:8080;
}

upstream frontend {
    server frontend:80;
}

server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;

    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;

    # Security headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Proxy API requests
    location /api/ {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Serve frontend
    location / {
        proxy_pass http://frontend;
        proxy_set_header Host $host;
    }
}
```

## Database Migration

Migrations run automatically on backend startup. For production:

1. **Test migrations locally first**
2. **Backup database before deployment**
3. **Deploy during maintenance window** (if breaking changes)
4. **Monitor logs** during startup

## Monitoring

### Health Checks

Backend health endpoint (if actuator enabled):
```
GET /actuator/health
```

### Logging

Configure logging levels in `application.yaml`:

```yaml
logging:
  level:
    org.scikeys.cms: INFO
    org.springframework.security: WARN
    org.hibernate: WARN
  file:
    name: /var/log/cms/application.log
```

### Monitoring Tools

- **Prometheus**: Metrics collection
- **Grafana**: Visualization
- **ELK Stack**: Log aggregation
- **Sentry**: Error tracking

## Backup Strategy

### Database Backups

**Automated Backup Script**:
```bash
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
docker exec sdc-cms-postgres pg_dump -U postgres university_cms > backup_$DATE.sql
```

**Restore**:
```bash
docker exec -i sdc-cms-postgres psql -U postgres university_cms < backup_20241221_120000.sql
```

### File Backups

Backup file storage directory:
```bash
tar -czf files_backup_$(date +%Y%m%d).tar.gz /data/files
```

## Scaling

### Horizontal Scaling

**Backend**:
- Run multiple backend instances
- Use load balancer
- Stateless design supports scaling

**Frontend**:
- Static files can be served from CDN
- Multiple nginx instances with load balancer

### Database Scaling

- Read replicas for reporting
- Connection pooling (already configured)
- Query optimization

## Troubleshooting Production

### Application Won't Start

1. Check logs: `docker compose logs backend`
2. Verify database connection
3. Check environment variables
4. Verify port availability

### Database Connection Issues

1. Check database is running: `docker compose ps postgres`
2. Verify credentials in environment variables
3. Check network connectivity
4. Review database logs

### Performance Issues

1. Check database query performance
2. Monitor resource usage (CPU, memory)
3. Review application logs
4. Check for N+1 query problems

## Next Steps

- Set up CI/CD pipeline
- Configure monitoring and alerting
- Set up automated backups
- Schedule regular security updates

---

**Document Version**: 1.0  
**Last Updated**: December 2024




