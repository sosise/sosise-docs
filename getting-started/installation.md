# Installation & Setup

## Quick Start

Get your Sosise project running in minutes! Perfect for building enterprise APIs with TypeScript.

```bash
# Install the Sosise CLI globally
npm install sosise-cli -g

# Create your new project
sosise new my-awesome-api
cd my-awesome-api

# Install dependencies and start coding
npm install
npm run serve
```

Your API server is now running at `http://localhost:10000`! 🚀

## Prerequisites

Before diving in, make sure you have these tools installed:

### Required
- **Node.js** (v16.0.0 or higher)
- **npm** (v8.0.0 or higher) or **yarn** (v1.22.0 or higher)

### Optional but Recommended
- **Docker** and **Docker Compose** for containerized development
- **Redis** for caching and queue management
- **MySQL/PostgreSQL** for production databases

```bash
# Verify your installation
node --version   # Should be v16.0.0+
npm --version    # Should be v8.0.0+
```

## Installation Methods

### 1. Using Sosise CLI

You can install Sosise CLI to quickly scaffold new projects:

```bash
# Install globally
npm install sosise-cli -g

# Create new project
sosise new my-project
cd my-project

# Install dependencies
npm install
```

> **Note**: sosise-cli v0.0.15 is available but may be outdated. For the most current setup, consider manual installation.

### 2. Manual Installation (Recommended)

For reliable setup, clone the official skeleton:

```bash
# Clone the starter template
git clone https://github.com/sosise/sosise.git my-project
cd my-project

# Remove git history and initialize your own
rm -rf .git
git init

# Install dependencies
npm install

# Copy environment configuration
cp .env.example .env

# Build the project
npm run build

# Start development server
npm run serve
```

## Project Setup

### Environment Configuration

After installation, configure your environment variables:

```bash
# Copy the example environment file
cp .env.example .env
```

**Essential Environment Variables:**

```bash
# .env
NODE_ENV=development
APP_PORT=10000
APP_URL=http://localhost:10000

# Database Configuration
DB_CONNECTION=mysql
DB_HOST=localhost
DB_PORT=3306
DB_DATABASE=my_database
DB_USERNAME=root
DB_PASSWORD=password

# Session Secret (generate a strong key)
SESSION_SECRET=your-super-secret-session-key-min-32-chars

# Optional: Cache & Queue (Redis)
REDIS_HOST=localhost
REDIS_PORT=6379
REDIS_PASSWORD=
```

### Database Setup

```bash
# Create your database (MySQL example)
mysql -u root -p
CREATE DATABASE my_database;
EXIT;

# Run migrations to set up tables
./artisan migrate:fresh

# Optional: Seed with sample data
./artisan seed
```

### Verification

Test your installation:

```bash
# Start the development server
npm run serve

# In another terminal, test the API
curl http://localhost:10000/
# Should return the basic response from IndexController

# Check available artisan commands
./artisan
```

## Development Workflow

### Daily Commands

```bash
# Start development server with hot reload
npm run serve

# Run tests
npm run test

# Check code quality
npm run lint
npm run lint:fix

# Database operations
./artisan migrate          # Run new migrations
./artisan migrate:rollback # Rollback last migration
./artisan seed             # Seed database
```

### Code Generation

```bash
# Generate application components
./artisan make:controller UserController
./artisan make:service UserService
./artisan make:repository UserRepository
./artisan make:middleware AuthMiddleware
./artisan make:unifier CreateUserUnifier
./artisan make:exception UserNotFoundException
```

## Development Tools

### IDE Configuration

For the best development experience:

**Visual Studio Code Extensions:**
- **TypeScript Hero** - Auto imports and organization
- **ESLint** - Code linting
- **Prettier** - Code formatting
- **Thunder Client** - API testing
- **MySQL/PostgreSQL** - Database management

**WebStorm/IntelliJ IDEA:**
- Built-in TypeScript support
- Database tools
- REST Client plugin

### Debug Configuration

**VS Code launch.json:**
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "Launch Sosise App",
            "program": "${workspaceFolder}/build/server.js",
            "env": {
                "NODE_ENV": "development"
            },
            "console": "integratedTerminal",
            "internalConsoleOptions": "neverOpen"
        }
    ]
}
```

## Docker Development (Optional)

For containerized development:

```bash
# Start with Docker Compose
docker-compose up -d

# Access your application
curl http://localhost:10000/api/health

# View logs
docker-compose logs -f app

# Stop containers
docker-compose down
```

**Docker Compose Services:**
- **app** - Your Sosise application
- **mysql** - Database server
- **redis** - Cache and queue server
- **adminer** - Database admin interface

## Project Structure Overview

After installation, your project will have this structure:

```
my-project/
├── src/
│   ├── app/                 # Application logic
│   │   ├── Http/           # Controllers & Middlewares
│   │   ├── Services/       # Business logic
│   │   └── Repositories/   # Data access layer
│   ├── config/             # Configuration files
│   ├── database/           # Migrations & seeders
│   └── routes/             # API routes
├── storage/                # File storage
├── tests/                  # Test files
├── docker/                 # Docker configuration
├── .env                    # Environment variables
└── artisan                 # Command-line tool
```

## Troubleshooting

### Common Issues

**Port already in use:**
```bash
# Find process using port 10000
lsof -i :10000
# Kill the process
kill -9 [PID]
```

**Database connection failed:**
```bash
# Verify database is running
mysql -u root -p
# Check connection details in .env
# Ensure database exists
```

**TypeScript compilation errors:**
```bash
# Clear build cache
rm -rf build/
# Rebuild project
npm run build
```

**Permission issues (macOS/Linux):**
```bash
# Make artisan executable
chmod +x artisan
```

### Getting Help

- 📚 **Documentation**: https://sosise.github.io/sosise-docs/
- 🐛 **Issues**: https://github.com/sosise/sosise/issues
- 💬 **Discussions**: https://github.com/sosise/sosise/discussions
- 📧 **Email**: support@sosise.com

## Next Steps

Now that your Sosise project is installed:

1. 🏗️ **Explore Architecture** - Learn about the [Directory Structure](getting-started/directory-structure.md)
2. 🗄️ **Setup Database** - Configure your [Database Connection](database/getting-started.md)
3. 🎯 **Create Your First API** - Build endpoints with [Controllers & Services](documentation/controllers.md)
4. 🔒 **Add Authentication** - Secure your API with [Session Management](documentation/session.md)
5. ✅ **Write Tests** - Ensure quality with [Testing Framework](documentation/testing.md)

Welcome to the Sosise ecosystem! Build amazing TypeScript APIs with confidence. 🎉