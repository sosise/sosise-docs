# Docker Integration

## Quick Start

Get your Sosise application running in Docker containers instantly! Perfect for development, testing, and production deployments.

```bash
# Start your stack with one command
docker-compose up -d

# Your API is now running at http://localhost:10000
curl http://localhost:10000/
```

Includes Redis for caching and queue management out of the box! 🐳

## Why Docker with Sosise?

Docker provides consistent environments across all stages:

- ✅ **Development Consistency** - Same environment for all developers
- ✅ **Easy Redis Setup** - Redis service included for caching and queues
- ✅ **Production Ready** - Optimized Alpine Linux builds
- ✅ **Quick Deployment** - Build once, deploy anywhere
- ✅ **Environment Isolation** - No conflicts with local installations

## Docker Compose Setup

### Default Stack

Your Sosise project includes a minimal but complete development stack:

```yaml
# docker-compose.yml
services:
  app:
    build:
      dockerfile: "./docker/Dockerfile"
      context: .
    container_name: sosise
    restart: unless-stopped
    ports:
      - "10000:10000"
    volumes:
      - "./.env:/var/www/app/.env"
      - "./cron:/etc/crontabs/root"
      - "./logs:/var/www/app/storage/logs"
  redis:
    image: redis:alpine3.21
    container_name: redis
    restart: unless-stopped
    entrypoint: redis-server --appendonly yes
    volumes:
      - "./redis:/data"
```

### Services Included

| Service | Port | Purpose |
|---------|------|---------|
| **app** | 10000 | Your Sosise application |
| **redis** | Internal | Caching and queue management |

## Dockerfile Architecture

Sosise uses a multi-process Alpine Linux container with supervisord:

```dockerfile
# docker/Dockerfile (multi-stage build)

# Stage 1: Build TypeScript
FROM alpine:3.21 AS builder
WORKDIR /var/www/app
RUN apk add --no-cache nodejs npm
COPY package.json package-lock.json ./
RUN npm ci
COPY src/ ./src/
COPY tsconfig.json ./tsconfig.json
COPY docs/ ./docs/
RUN npm run build

# Stage 2: Production image
FROM alpine:3.21
WORKDIR /var/www/app
RUN apk add --no-cache nodejs npm tzdata supervisor dcron
COPY ./docker/configs/supervisord/supervisord.conf /etc/supervisord.conf
COPY ./docker/docker-entrypoint.sh /docker-entrypoint.sh
RUN chmod +x /docker-entrypoint.sh
COPY --from=builder /var/www/app/build ./build
COPY --from=builder /var/www/app/docs ./docs
COPY --from=builder /var/www/app/package.json ./package.json
COPY --from=builder /var/www/app/package-lock.json ./package-lock.json
COPY artisan ./artisan
RUN npm ci --omit=dev
RUN mkdir -p storage/logs storage/sessions storage/cache storage/framework
ENTRYPOINT ["/docker-entrypoint.sh"]
```

The multi-stage build ensures that TypeScript, ESLint, Jest, and other dev dependencies are **not** included in the production image, keeping it small (~85MB compressed).

### Container Architecture

The container runs multiple processes via supervisord:

- **Node.js Application** (v22 LTS) - Your Sosise API server
- **Cron Daemon** - Scheduled tasks from your `cron` file
- **Queue Workers** - Background job processing (if enabled)

## Development Commands

### Basic Docker Operations

```bash
# Start development stack
docker-compose up -d

# View logs
docker-compose logs -f app
docker-compose logs -f redis

# Stop containers
docker-compose down

# Rebuild after code changes
docker-compose build app
docker-compose up -d
```

### Application Management

```bash
# Access container shell
docker exec -it sosise sh

# Run artisan commands
docker exec sosise ./artisan migrate
docker exec sosise ./artisan seed

# Watch application logs
docker exec sosise tail -f storage/logs/sosise.log
```

### Database Setup

**Important**: Sosise doesn't include a database container by default. For development, you can:

1. **Use external database** (recommended for production):
   ```bash
   # Add to your .env
   DB_HOST=your-external-database-host
   DB_DATABASE=your_database
   ```

2. **Add database service** to docker-compose.yml if needed:
   ```yaml
   mysql:
     image: mysql:8.0
     environment:
       MYSQL_DATABASE: sosise_db
       MYSQL_ROOT_PASSWORD: password
     ports:
       - "3306:3306"
   ```

## Production Deployment

### Environment Configuration

```bash
# .env.production
NODE_ENV=production
APP_PORT=10000

# Database (external)
DB_HOST=your-production-db-host
DB_DATABASE=your_production_db

# Redis (can use container or external)
REDIS_HOST=redis
REDIS_PORT=6379

# Caching
CACHE_DRIVER=redis

# Queues
QUEUE_DRIVER=redis
```

### Production Docker Compose

```yaml
# docker-compose.prod.yml
services:
  app:
    image: your-registry/sosise-app:latest
    container_name: sosise-prod
    restart: unless-stopped
    ports:
      - "10000:10000"
    volumes:
      - "./.env:/var/www/app/.env"
      - "./logs:/var/www/app/storage/logs"
  redis:
    image: redis:alpine3.21
    container_name: redis-prod
    restart: unless-stopped
    entrypoint: redis-server --appendonly yes
    volumes:
      - "./redis:/data"
```

## Build Process

### Local Development

```bash
# Install dependencies
npm install

# Build TypeScript
npm run build

# Build Docker image
docker-compose build

# Start services
docker-compose up -d
```

### Production Build

```bash
# Build optimized image
./build-docker-image.sh

# The script handles:
# 1. TypeScript compilation
# 2. Docker image creation
# 3. Dependency optimization
# 4. Asset preparation
```

## Container Management

### Health Monitoring

```bash
# Check container status
docker-compose ps

# Monitor resource usage
docker stats sosise redis

# Check application logs
docker-compose logs -f app | grep ERROR
```

### Performance Tuning

```bash
# Adjust memory limits
docker run --memory="512m" your-sosise-app

# Configure multiple threads
docker run -e THREADS_AMOUNT=4 your-sosise-app

# Optimize Redis persistence
redis-server --save 300 10 --appendonly yes
```

## Troubleshooting

### Common Issues

**Container won't start:**
```bash
# Check logs
docker-compose logs app

# Verify .env file
docker exec sosise cat .env

# Check permissions
docker exec sosise ls -la /var/www/app
```

**Database connection issues:**
```bash
# Test database connectivity
docker exec sosise node -e "console.log(require('./build/config/database.ts'))"

# Check network connectivity
docker exec sosise ping your-database-host
```

**Redis connection problems:**
```bash
# Test Redis connection
docker exec sosise redis-cli -h redis ping

# Check Redis logs
docker-compose logs redis
```

## Best Practices

### Development Workflow

1. **Keep containers running** during development
2. **Use volume mounts** for logs and configuration  
3. **External database** for better performance
4. **Regular container cleanup** to save disk space

### Production Considerations

1. **External managed databases** (RDS, etc.)
2. **Redis clustering** for high availability
3. **Log aggregation** with ELK or similar
4. **Container orchestration** with Kubernetes/Docker Swarm

## Summary

Sosise's Docker setup provides:

- ✅ **Minimal footprint** with Alpine Linux
- ✅ **Multi-process architecture** with supervisord
- ✅ **Redis integration** for caching and queues
- ✅ **Development-friendly** volume mounts
- ✅ **Production-ready** configuration
- ✅ **Flexible database** connection options

Perfect for both development and production deployments! 🚀