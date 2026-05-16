# 🔌 Port Configuration Guide

## Summary: Port Configuration

| Environment | Local Port | Render Port | Reason |
|---|---|---|---|
| **DEV** | **8080** | 8080 | Default, first environment |
| **QA** | **8081** | 8080 | Avoid conflict with DEV locally |
| **PROD** | **8082** | 8080 | Avoid conflict with others locally |

---

## 📍 Why Different Ports Locally?

### **On Your Computer (Local Development):**

```
Your Machine (localhost)

❌ CONFLICT - Same Port:
  Service 1 listens on http://localhost:8080
  Service 2 tries to listen on http://localhost:8080 ❌ ERROR!

✅ SOLUTION - Different Ports:
  DEV:  http://localhost:8080
  QA:   http://localhost:8081  ← Different port
  PROD: http://localhost:8082  ← Different port
```

### **On Render (Cloud Servers):**

```
Render Cloud

✅ NO CONFLICT - Same Port (each separate server):
  dev-service   listens on http://dev-employee-service.render.com (port 8080 internally)
  qa-service    listens on http://qa-employee-service.render.com (port 8080 internally)
  prod-service  listens on http://employee-service.render.com (port 8080 internally)
  
  Each service has its own server, so port conflicts don't happen!
```

---

## 🚀 How to Run Locally

### **Run DEV Locally:**
```bash
mvn spring-boot:run -Pdev
# Listens on: http://localhost:8080
# Uses: application-dev.properties (port 8080)
```

### **Run QA Locally:**
```bash
# In different terminal
mvn spring-boot:run -Pqa
# Listens on: http://localhost:8081
# Uses: application-qa.properties (port 8081)
```

### **Run PROD Locally:**
```bash
# In another different terminal
mvn spring-boot:run -Pprod
# Listens on: http://localhost:8082
# Uses: application-prod.properties (port 8082)
```

### **All 3 Running at Once:**
```bash
Terminal 1: mvn spring-boot:run -Pdev   → http://localhost:8080
Terminal 2: mvn spring-boot:run -Pqa    → http://localhost:8081
Terminal 3: mvn spring-boot:run -Pprod  → http://localhost:8082

✅ No conflicts! All running simultaneously.
```

---

## 🐳 Docker Setup

### **Docker Compose (Recommended for Local)**

```yaml
# Create: docker-compose.yml
version: '3.8'

services:
  dev:
    build: .
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=dev
    
  qa:
    build: .
    ports:
      - "8081:8080"  # Host 8081 → Container 8080
    environment:
      - SPRING_PROFILES_ACTIVE=qa
    
  prod:
    build: .
    ports:
      - "8082:8080"  # Host 8082 → Container 8080
    environment:
      - SPRING_PROFILES_ACTIVE=prod
```

**Run all at once:**
```bash
docker-compose up

# Access:
# DEV:  http://localhost:8080
# QA:   http://localhost:8081
# PROD: http://localhost:8082
```

### **Individual Docker Commands:**

```bash
# Build image
docker build -t employee-service .

# Run DEV
docker run -p 8080:8080 \
  -e SPRING_PROFILES_ACTIVE=dev \
  employee-service

# Run QA (different terminal)
docker run -p 8081:8080 \
  -e SPRING_PROFILES_ACTIVE=qa \
  employee-service

# Run PROD (different terminal)
docker run -p 8082:8080 \
  -e SPRING_PROFILES_ACTIVE=prod \
  employee-service
```

---

## 🌐 On Render (No Port Issues)

### **Render Service Configuration:**

Each service on Render is **completely isolated**, so no port conflicts:

```
Render Dashboard → Web Service → Settings

For DEV Service:
  Environment Variables:
    SPRING_PROFILES_ACTIVE=dev
  Port (internal): 8080 ✅ (OK, no conflict)
  URL: dev-employee-service.render.com

For QA Service:
  Environment Variables:
    SPRING_PROFILES_ACTIVE=qa
  Port (internal): 8080 ✅ (OK, no conflict)
  URL: qa-employee-service.render.com

For PROD Service:
  Environment Variables:
    SPRING_PROFILES_ACTIVE=prod
  Port (internal): 8080 ✅ (OK, no conflict)
  URL: employee-service.render.com
```

**The key:** Each service runs on **different servers**, so port 8080 on one doesn't conflict with port 8080 on another.

---

## 📋 Property File Port Configuration

### **application-dev.properties:**
```ini
server.port=8080
# Local: http://localhost:8080
# Render: uses 8080 (no conflict, separate server)
```

### **application-qa.properties:**
```ini
server.port=8081
# Local: http://localhost:8081 (different from DEV)
# Render: uses 8080 (no conflict, separate server)
```

### **application-prod.properties:**
```ini
server.port=8082
# Local: http://localhost:8082 (different from others)
# Render: uses 8080 (no conflict, separate server)
```

---

## 🔄 Override Ports Dynamically

### **Override via Command Line:**

```bash
# Override default port from properties
mvn spring-boot:run -Dserver.port=9090

# Or with profile:
mvn spring-boot:run -Pqa -Dserver.port=9090
```

### **Override via Environment Variable:**

```bash
export SERVER_PORT=9090
mvn spring-boot:run -Pdev

# Or:
SERVER_PORT=9090 mvn spring-boot:run -Pdev
```

### **Override in Docker:**

```bash
docker run -p 9090:8080 \
  -e SERVER_PORT=8080 \
  -e SPRING_PROFILES_ACTIVE=qa \
  employee-service

# Access: http://localhost:9090
```

---

## ✅ Quick Verification

### **Check Which Port is in Use:**

```bash
# macOS/Linux
lsof -i :8080
lsof -i :8081
lsof -i :8082

# Windows
netstat -ano | findstr :8080

# Docker
docker ps

# Extract port info
docker port <container_id>
```

### **Kill Process on Specific Port:**

```bash
# macOS/Linux
lsof -i :8080 | grep -v COMMAND | awk '{print $2}' | xargs kill -9

# Or specify PID
kill -9 <PID>

# Windows
netstat -ano | findstr :8080
taskkill /PID <PID> /F
```

---

## 🎯 Summary for Different Scenarios

### **Scenario 1: Running Locally (All at Once)**
```
Terminal 1: mvn spring-boot:run -Pdev    → 8080 ✅
Terminal 2: mvn spring-boot:run -Pqa     → 8081 ✅
Terminal 3: mvn spring-boot:run -Pprod   → 8082 ✅

Access:
  Dev:  curl http://localhost:8080/health
  QA:   curl http://localhost:8081/health
  Prod: curl http://localhost:8082/health
```

### **Scenario 2: Running in Docker (All at Once)**
```
docker-compose up

Access:
  Dev:  curl http://localhost:8080/health
  QA:   curl http://localhost:8081/health
  Prod: curl http://localhost:8082/health
```

### **Scenario 3: Deployed on Render**
```
Services:
  dev-service.render.com → 8080 internally ✅
  qa-service.render.com  → 8080 internally ✅
  prod-service.render.com → 8080 internally ✅

Access:
  Dev:  https://dev-employee-service.render.com
  QA:   https://qa-employee-service.render.com
  Prod: https://employee-service.render.com
```

---

## ⚠️ Important Notes

1. **Local conflicts are only an issue when**:
   - Running multiple instances on same machine
   - Without Docker (direct Java execution)
   - OR mapping same host port in Docker

2. **Render has no port conflicts because**:
   - Each service runs on different infrastructure
   - Each gets isolated container/instance
   - The host port mapping only happens on YOUR machine

3. **Property files now have**:
   - DEV: 8080 (default)
   - QA: 8081 (local only)
   - PROD: 8082 (local only)

4. **Render configuration**:
   - Keep all services on 8080 internally
   - Render exposes them via separate URLs
   - No need to change ports on Render

---

## 🚀 Bottom Line

✅ **On Render:** All environments can use port 8080 (no issue)
✅ **Locally:** Use different ports (8080, 8081, 8082) to avoid conflicts
✅ **Updated:** Property files now have correct ports configured

**You're all set!** 🎉

