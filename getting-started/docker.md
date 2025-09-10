# Docker Integration

## Quick Start

Get your Sosise application running in Docker containers instantly! Perfect for development, testing, and production deployments.

```bash
# Start your entire stack with one command
docker-compose up -d

# Your API is now running at http://localhost:10000
curl http://localhost:10000/api/health
```

Includes MySQL, Redis, and Adminer out of the box! 🐳

## Why Docker with Sosise?

Docker provides consistent environments across all stages:

- ✅ **Development Consistency** - Same environment for all developers
- ✅ **Easy Dependencies** - MySQL, Redis, and other services included
- ✅ **Production Ready** - Optimized multi-stage builds
- ✅ **Quick Deployment** - Build once, deploy anywhere
- ✅ **Environment Isolation** - No conflicts with local installations

## Docker Compose Setup

### Default Stack

Your project includes a complete development stack:

```yaml
# docker-compose.yml
version: '3.8'
services:
  app:
    build: .
    ports:
      - "10000:10000"
    environment:
      - NODE_ENV=development
    volumes:
      - ./src:/app/src
      - ./storage:/app/storage
    depends_on:
      - mysql
      - redis
      
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: password
      MYSQL_DATABASE: sosise_app
    ports:
      - "3306:3306"
    volumes:
      - mysql_data:/var/lib/mysql
      
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
      
  adminer:
    image: adminer:latest
    ports:
      - "8080:8080"
```

### Starting Your Stack

```bash
# Start all services in background
docker-compose up -d

# View logs from all services
docker-compose logs -f

# View logs from specific service
docker-compose logs -f app

# Stop all services
docker-compose down

# Stop and remove volumes (clean slate)
docker-compose down -v
```

### Development Workflow

```bash
# Start development environment
docker-compose up -d

# Execute commands inside app container
docker-compose exec app ./artisan migrate:fresh
docker-compose exec app ./artisan seed
docker-compose exec app npm test

# Access database via Adminer
# Visit http://localhost:8080
# Server: mysql, Username: root, Password: password

# Monitor application logs
docker-compose logs -f app
```

## Custom Docker Build

### Using Build Script

Sosise includes an intelligent build script for custom images:

```bash
# Interactive build with platform selection
./build-docker-image.sh

# Follow prompts for:
# - Image name and tag
# - Target platform (linux/amd64, linux/arm64)
# - Build optimizations
```

### Manual Docker Build

```bash
# Build for current platform
docker build -t my-sosise-app .

# Build for multiple platforms (requires buildx)
docker buildx build \
  --platform linux/amd64,linux/arm64 \
  -t my-sosise-app:latest \
  --push .

# Build for production (optimized)
docker build \
  --target production \
  -t my-sosise-app:prod .
```

## Dockerfile Explained

Sosise uses a multi-stage Dockerfile for optimal builds:

```dockerfile
# Multi-stage build for optimized production images
FROM node:18-alpine AS base
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

# Development stage
FROM base AS development
RUN npm ci
COPY . .
CMD ["npm", "run", "serve"]

# Build stage
FROM base AS builder
RUN npm ci
COPY . .
RUN npm run build

# Production stage
FROM node:18-alpine AS production
WORKDIR /app
COPY --from=base /app/node_modules ./node_modules
COPY --from=builder /app/build ./build
COPY package*.json ./
COPY artisan ./
EXPOSE 10000
CMD ["node", "build/server.js"]
```

**Benefits:**
- **Small Production Images** - Only runtime dependencies included
- **Fast Builds** - Leverages Docker layer caching
- **Security** - Minimal attack surface in production
- **Efficiency** - Separate stages for different use cases

## Environment Configuration

### Docker Environment Variables

```bash
# .env.docker (for containerized development)
NODE_ENV=development
APP_PORT=10000
APP_URL=http://localhost:10000

# Database (using Docker Compose service names)
DB_CONNECTION=mysql
DB_HOST=mysql
DB_PORT=3306
DB_DATABASE=sosise_app
DB_USERNAME=root
DB_PASSWORD=password

# Redis (using Docker Compose service name)
REDIS_HOST=redis
REDIS_PORT=6379

# Session configuration
SESSION_SECRET=your-docker-session-secret
SESSION_DRIVER=redis

# Logging
LOG_CHANNEL=file
LOG_LEVEL=debug
```

### Volume Mounting

```yaml
# Development with hot reload
volumes:
  - ./src:/app/src           # Source code hot reload
  - ./storage:/app/storage   # Persistent storage
  - ./tests:/app/tests       # Test files
  - node_modules:/app/node_modules  # Prevent overwriting
```

## Production Deployment

### Production Docker Compose

```yaml
# docker-compose.prod.yml
version: '3.8'
services:
  app:
    build:
      context: .
      target: production
    ports:
      - "10000:10000"
    environment:
      - NODE_ENV=production
    env_file:
      - .env.production
    restart: unless-stopped
    depends_on:
      - mysql
      - redis
      
  mysql:
    image: mysql:8.0
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: ${DB_DATABASE}
      MYSQL_USER: ${DB_USERNAME}
      MYSQL_PASSWORD: ${DB_PASSWORD}
    volumes:
      - mysql_prod_data:/var/lib/mysql
    restart: unless-stopped
    
  redis:
    image: redis:7-alpine
    command: redis-server --requirepass ${REDIS_PASSWORD}
    restart: unless-stopped
    
  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./ssl:/etc/nginx/ssl
    depends_on:
      - app
    restart: unless-stopped

volumes:
  mysql_prod_data:
```

### Deploy to Production

```bash
# Deploy with production compose file
docker-compose -f docker-compose.prod.yml up -d

# Run migrations in production
docker-compose -f docker-compose.prod.yml exec app ./artisan migrate

# Check production logs
docker-compose -f docker-compose.prod.yml logs -f app
```

## Scheduled Jobs with Cron

### Cron Configuration

Sosise includes cron support for scheduled tasks:

```bash
# cron file in project root
# Listen to default queue
* * * * * cd /app && ./artisan queue:listen default

# Daily reports at 2 AM
0 2 * * * cd /app && ./artisan report:daily

# Weekly cleanup on Sundays at 1 AM
0 1 * * 0 cd /app && ./artisan cache:clear
```

### Docker Compose with Cron

```yaml
services:
  app:
    build: .
    volumes:
      - ./cron:/etc/crontabs/root  # Mount cron configuration
    command: >
      sh -c "crond && npm run serve"  # Start cron and app
```

### Cron Management

```bash
# Edit cron jobs
vim cron

# Restart container to apply cron changes
docker-compose restart app

# View cron logs
docker-compose logs app | grep cron

# Test cron job manually
docker-compose exec app ./artisan report:daily
```

## Development Tools Integration

### Database Management

```bash
# Access MySQL directly
docker-compose exec mysql mysql -u root -p

# Import database dump
docker-compose exec -T mysql mysql -u root -ppassword sosise_app < backup.sql

# Export database
docker-compose exec mysql mysqldump -u root -ppassword sosise_app > backup.sql

# Use Adminer web interface
open http://localhost:8080
```

### Redis Management

```bash
# Access Redis CLI
docker-compose exec redis redis-cli

# Monitor Redis operations
docker-compose exec redis redis-cli monitor

# View Redis info
docker-compose exec redis redis-cli info
```

### Log Management

```bash
# Follow all logs
docker-compose logs -f

# Follow specific service logs
docker-compose logs -f app
docker-compose logs -f mysql
docker-compose logs -f redis

# View last 100 lines
docker-compose logs --tail=100 app
```

## Troubleshooting

### Common Issues

**Port Already in Use:**
```bash
# Find process using port
lsof -i :10000
# Stop conflicting services
docker-compose down
```

**Database Connection Failed:**
```bash
# Check if MySQL is running
docker-compose ps mysql
# View MySQL logs
docker-compose logs mysql
# Restart MySQL
docker-compose restart mysql
```

**File Permission Issues:**
```bash
# Fix ownership (Linux/macOS)
sudo chown -R $USER:$USER storage/
chmod -R 755 storage/
```

**Container Build Failures:**
```bash
# Clean Docker cache
docker system prune -f
# Rebuild without cache
docker-compose build --no-cache
```

### Performance Optimization

```bash
# Use .dockerignore to exclude unnecessary files
echo "node_modules\n.git\n*.log" > .dockerignore

# Optimize for Apple Silicon (M1/M2/M3)
docker buildx build --platform linux/arm64 -t my-app .

# Enable BuildKit for faster builds
export DOCKER_BUILDKIT=1
docker build -t my-app .
```

## Docker Compose Services

### Available Services

| Service | Port | Purpose | Access |
|---------|------|---------|--------|
| **app** | 10000 | Sosise API | http://localhost:10000 |
| **mysql** | 3306 | Database | mysql://root:password@localhost:3306 |
| **redis** | 6379 | Cache/Sessions | redis://localhost:6379 |
| **adminer** | 8080 | DB Admin | http://localhost:8080 |

### Service Commands

```bash
# Start specific services
docker-compose up app mysql

# Scale services (if needed)
docker-compose up --scale app=3

# Update specific service
docker-compose build app
docker-compose up -d app

# Remove specific service
docker-compose rm -f app
```

## Best Practices

### ✅ **DO: Use Multi-stage Builds**
```dockerfile
# Separate build and runtime stages
FROM node:18-alpine AS builder
# Build steps...

FROM node:18-alpine AS production
# Runtime only
```

### ✅ **DO: Use .dockerignore**
```
node_modules
.git
*.log
build
storage/logs
```

### ✅ **DO: Health Checks**
```dockerfile
HEALTHCHECK --interval=30s --timeout=3s \
  CMD curl -f http://localhost:10000/health || exit 1
```

### ❌ **DON'T: Run as Root in Production**
```dockerfile
# Create non-root user
RUN adduser -D -s /bin/sh sosise
USER sosise
```

### ❌ **DON'T: Store Secrets in Images**
```bash
# Use environment variables instead
ENV SESSION_SECRET=${SESSION_SECRET}
```

## Summary

Docker integration with Sosise provides:

- 🚀 **Fast Development Setup** - Complete stack in minutes
- 🔄 **Consistent Environments** - Same setup across team and deployments
- 📦 **Easy Dependencies** - MySQL, Redis included out of the box
- 🏗️ **Production Ready** - Optimized multi-stage builds
- 🛡️ **Environment Isolation** - No conflicts with local installations
- ⚡ **Hot Reload** - Development changes reflected instantly
- 📊 **Monitoring Tools** - Built-in database admin interface
- 📅 **Scheduled Jobs** - Cron support for background tasks

Develop and deploy your Sosise applications with confidence using Docker! 🐳