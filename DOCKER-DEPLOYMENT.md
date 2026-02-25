# Spring Boot Application - Docker Deployment Guide

## Files Created

### 1. **Dockerfile**
- Multi-stage build for optimized image size
- Build stage: Uses `eclipse-temurin:17-jdk` to compile the application
- Runtime stage: Uses `eclipse-temurin:17-jre` (lightweight)
- Exposes port **8888**

### 2. **docker-compose.yml**
Includes two services:
- **database**: PostgreSQL 15 Alpine image
  - Port: 5433 (mapped to 5432 inside container)
  - Database: `lms_db`
  - User: `dev_user`
  - Password: `123456`
  - Health check enabled
  - Data persisted in volume: `postgres_data`

- **backend-app**: Your Spring Boot application
  - Port: 8888
  - Environment variables configured for database connection
  - Depends on database being healthy
  - Restart policy: unless-stopped

### 3. **.dockerignore**
- Excludes unnecessary files from Docker build context
- Reduces build time and image size

## Quick Start

### Prerequisites
- Docker Desktop installed and running

### Deploy
```bash
cd /Users/heybaby/demo1
docker compose up -d
```

### View Logs
```bash
docker compose logs -f backend-app
```

### Check Status
```bash
docker compose ps
```

### Stop Services
```bash
docker compose down
```

### Remove Everything (including data)
```bash
docker compose down -v
```

## Application Behavior

**Note:** Your current Spring Boot application exits cleanly after initialization because it has no active components (no web server, no message listeners, etc.). This is normal behavior.

### If you see: "Exited (0) X seconds ago"
This means:
- ✅ The application started successfully
- ✅ Spring Boot initialization completed
- ✅ The application exited cleanly (exit code 0 = success)

To keep the application running continuously, you have two options:

### **Option 1: Add Spring Web (REST API)**
If you want to expose HTTP endpoints, add to `build.gradle`:

```gradle
dependencies {
    // ...existing dependencies...
    implementation 'org.springframework.boot:spring-boot-starter-web'
}
```

Then rebuild:
```bash
docker compose down -v
docker compose up -d --build
```

This will make your app a web server listening on port 8888.

### **Option 2: Add Other Active Components**
- Message Queue Listener (RabbitMQ, Kafka)
- Scheduled Tasks
- Background Jobs
- Any other component that keeps threads alive

### **Option 3: Keep Current Behavior**
If your app is designed to:
- Run batch jobs
- Process data once on startup
- Perform scheduled tasks

Then the current behavior is correct. You can:
1. Run it on-demand: `docker compose up`
2. Check logs: `docker compose logs backend-app`
3. Stop when done: `docker compose down`

## Accessing Services

- **Spring Boot App**: http://localhost:8888
- **PostgreSQL Database**: localhost:5433
  - Username: `dev_user`
  - Password: `123456`
  - Database: `lms_db`

## Network Configuration
- Services communicate through `app-network` bridge network
- Database hostname inside app: `database`
- Connection string: `jdbc:postgresql://database:5432/lms_db`

## Environment Variables
The backend app receives these environment variables:
- `SPRING_PROFILES_ACTIVE=prod`
- `DB_HOST=database`
- `DB_PORT=5432`
- `SPRING_DATASOURCE_URL=jdbc:postgresql://database:5432/lms_db`
- `SPRING_DATASOURCE_USERNAME=dev_user`
- `SPRING_DATASOURCE_PASSWORD=123456`

## Database Persistence
PostgreSQL data is stored in the `postgres_data` volume. This ensures data persists even when containers are stopped.

## Troubleshooting

### Container Shows "Exited (0)"
This is normal! The app started successfully and exited cleanly. See "Application Behavior" section above.

### Port Already in Use
If you get "port already in use" error, either:
1. Stop other services using the port
2. Change the port mapping in `docker-compose.yml`
3. Check with: `lsof -i :5435` (macOS/Linux)

### Container Won't Stay Running (Keeps Exiting)
Your app exits because it has no active components. This is expected behavior for non-web applications. See "Application Behavior" section for solutions.

To verify it's working, check the logs:
```bash
docker compose logs backend-app
# Look for: "Started Demo1Application in X seconds"
```

### Container Won't Start at All
Check logs with:
```bash
docker compose logs backend-app
docker compose logs database
```

### Rebuild Application
To rebuild without cache:
```bash
docker compose down -v
docker compose up -d --build --force-recreate
```

### Clean Up Everything
```bash
docker compose down -v
docker system prune -a
```

## Adding Web Dependencies (Optional)
If you want to make your app a REST API, add to `build.gradle`:
```gradle
implementation 'org.springframework.boot:spring-boot-starter-web'
```

Then rebuild:
```bash
docker compose up -d --build
```

