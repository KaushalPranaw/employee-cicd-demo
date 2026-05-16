# 💻 Local Development & Testing Guide

## Quick Start

### 1. Clone & Setup
```bash
git clone <your-repo-url>
cd employee-service
```

### 2. Build & Run for DEV (Default)
```bash
# Build with DEV profile
mvn clean install -Pdev

# Run the application
mvn spring-boot:run

# Or run the JAR directly
java -jar target/employee-service-0.0.1-SNAPSHOT.jar
```

The app will use:
- **Properties File:** `application-dev.properties`
- **Database:** Local PostgreSQL (localhost:5432)
- **Port:** 8080
- **Logging Level:** DEBUG
- **URL:** http://localhost:8080

---

## Building for Different Environments

### Build for DEV
```bash
mvn clean package -Pdev
```
**Output:** `target/employee-service-0.0.1-SNAPSHOT.jar`
**Uses:** `application-dev.properties`

### Build for QA
```bash
mvn clean package -Pqa
```
**Output:** `target/employee-service-0.0.1-SNAPSHOT.jar`
**Uses:** `application-qa.properties`

### Build for PROD
```bash
mvn clean package -Pprod
```
**Output:** `target/employee-service-0.0.1-SNAPSHOT.jar`
**Uses:** `application-prod.properties`

---

## Running with Different Profiles

### Run with DEV Profile
```bash
mvn spring-boot:run -Pdev
```

### Run with QA Profile
```bash
mvn spring-boot:run -Pqa
```

### Run with PROD Profile
```bash
mvn spring-boot:run -Pprod
```

### Run JAR with Specific Profile
```bash
java -jar target/employee-service-0.0.1-SNAPSHOT.jar \
  --spring.profiles.active=qa
```

---

## Understanding Profiles

### application.properties (Default)
- Used when NO profile is specified
- Sets DEV defaults
- Fallback for all profiles

### application-dev.properties
- Activated by: `spring.profiles.active=dev`
- Overrides: application.properties
- Purpose: Development environment
- Database: Local PostgreSQL
- DDL: `update` (auto-creates/updates schema)
- Logging: DEBUG

### application-qa.properties
- Activated by: `spring.profiles.active=qa`
- Overrides: application.properties
- Purpose: Quality Assurance environment
- Database: QA server (via environment variable)
- DDL: `validate` (only validates schema)
- Logging: INFO

### application-prod.properties
- Activated by: `spring.profiles.active=prod`
- Overrides: application.properties
- Purpose: Production environment
- Database: PROD server (via environment variable)
- DDL: `validate` (strict - no changes)
- Logging: WARN

---

## Testing Before Commit

### 1. Build & Test (All Tests)
```bash
mvn clean test
```

### 2. Build & Package (Skip Tests)
```bash
mvn clean package -DskipTests
```

### 3. Full Build
```bash
mvn clean package
```

### 4. Check Compilation for All Profiles
```bash
mvn clean compile -Pdev
mvn clean compile -Pqa
mvn clean compile -Pprod
```

---

## Database Configuration for Testing

### Setup PostgreSQL Locally

#### Using Docker (Recommended)
```bash
# Start PostgreSQL container
docker run --name employee-db \
  -e POSTGRES_USER=employee_user_dev \
  -e POSTGRES_PASSWORD=dev_password_123 \
  -e POSTGRES_DB=employee_db_dev \
  -p 5432:5432 \
  -d postgres:15

# Verify it's running
docker ps
```

#### Manual Setup (macOS)
```bash
# Install PostgreSQL
brew install postgresql@15

# Start PostgreSQL
brew services start postgresql@15

# Create database
createdb employee_db_dev

# Create user
createuser employee_user_dev

# Set password
psql -d employee_db_dev -c "ALTER USER employee_user_dev WITH PASSWORD 'dev_password_123';"
```

#### Verify Connection
```bash
# Test connection
psql -h localhost -U employee_user_dev -d employee_db_dev

# Command prompt should show: employee_db_dev=#
# Type: \q to exit
```

---

## Checking Active Profile

### Method 1: Check Logs
```bash
mvn spring-boot:run 2>&1 | grep "profiles"
```

Look for output like:
```
The following profiles are active: dev
```

### Method 2: Application Startup Info
```bash
# After starting app, check console output
2024-05-16 10:30:45.123 INFO : The following profiles are active: dev
```

### Method 3: Check Properties
Add logging to a controller:
```java
@Value("${spring.profiles.active:default}")
private String activeProfile;

@GetMapping("/health")
public ResponseEntity<String> health() {
    return ResponseEntity.ok("Active Profile: " + activeProfile);
}
```

Then call:
```bash
curl http://localhost:8080/health
# Response: Active Profile: dev
```

---

## Environment Variables

### QA Environment Variables
```bash
# For local QA testing
export DB_PASSWORD_QA="your_qa_password"
export SPRING_PROFILES_ACTIVE=qa
mvn spring-boot:run
```

### PROD Environment Variables
```bash
# For local PROD simulation
export DB_PASSWORD_PROD="your_prod_password"
export SPRING_PROFILES_ACTIVE=prod
mvn spring-boot:run
```

---

## Maven Commands Reference

| Command | Purpose | Profile |
|---|---|---|
| `mvn clean install` | Full build & install | dev (default) |
| `mvn clean test` | Run all tests | dev (default) |
| `mvn clean package` | Build & package JAR | dev (default) |
| `mvn clean package -Pqa` | Build for QA | qa |
| `mvn clean package -Pprod` | Build for PROD | prod |
| `mvn spring-boot:run` | Run app locally | dev (default) |
| `mvn spring-boot:run -Pqa` | Run app with QA config | qa |
| `mvn compile` | Compile only | dev (default) |
| `mvn dependency:tree` | Show dependencies | dev (default) |

---

## Common Issues & Solutions

### Issue: Wrong Profile Being Used
```bash
# Verify active profile
mvn spring-boot:run -X 2>&1 | grep "profiles.active"

# Explicitly set profile
mvn spring-boot:run -Dspring.profiles.active=qa
```

### Issue: Database Connection Failed
```
Error: org.postgresql.util.PSQLException: 
  The connection attempt failed.
```

**Solution:**
1. Verify PostgreSQL is running: `docker ps` or `brew services list`
2. Check database connection string in `application-*.properties`
3. Verify username/password are correct
4. Ensure database exists: `psql -l`

### Issue: Build Fails with Compilation Errors
```bash
# Clean build
mvn clean

# Clear Maven cache
rm -rf ~/.m2/repository

# Rebuild
mvn clean install -Pdev
```

### Issue: Tests Failing Before Deployment
```bash
# Run tests with verbose output
mvn -X test

# Skip tests (use with caution!)
mvn package -DskipTests

# Run specific test
mvn test -Dtest=EmployeeServiceApplicationTests
```

---

## Verifying Configuration Files Are Correct

### Check Dev Config
```bash
grep -E "spring.datasource|spring.profiles" \
  src/main/resources/application-dev.properties
```

### Check QA Config
```bash
grep -E "spring.datasource|spring.profiles" \
  src/main/resources/application-qa.properties
```

### Check Prod Config
```bash
grep -E "spring.datasource|spring.profiles" \
  src/main/resources/application-prod.properties
```

---

## Pre-Deployment Checklist

Before pushing to GitHub:

- [ ] `mvn clean test` passes ✅
- [ ] `mvn clean package` succeeds ✅
- [ ] App runs locally with DEV profile ✅
- [ ] No compilation warnings ✅
- [ ] All properties files configured ✅
- [ ] Maven profiles defined in pom.xml ✅
- [ ] Git workflow file `.github/workflows/manual-deploy.yml` updated ✅
- [ ] GitHub environments configured (dev, qa, production) ✅
- [ ] GitHub secrets set (RENDER_*_HOOK, DB_PASSWORD_*) ✅

---

## Useful Links

- [Maven Documentation](https://maven.apache.org/what-is-maven.html)
- [Spring Boot Profiles](https://spring.io/blog/2015/02/17/spring-boot-management-endpoints-http-versus-jmx)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Docker PostgreSQL](https://hub.docker.com/_/postgres)

---

**Last Updated:** May 16, 2026

