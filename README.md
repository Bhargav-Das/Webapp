# Containerized Web Application with PostgreSQL using Docker Compose & IPvlan

**Project Assignment 1 — Containerization and DevOps**

---

##  Overview

This project demonstrates a **containerized web application** built using:

*  **FastAPI** (Backend API)
*  **PostgreSQL** (Database)
*  **Docker & Docker Compose** (Container orchestration)
*  **IPvlan Networking** (Static IP-based communication)

Each service runs in an isolated container while communicating through a custom IPvlan network.

---

##  Architecture

```
Client (Browser/Postman)
        │
        │ HTTP Request
        ▼
Backend Container (FastAPI)
IP: 172.22.208.11
        │
        │ SQL Queries
        ▼
PostgreSQL Container
IP: 172.22.208.10
        │
        ▼
Docker Volume (pgdata)
```

---

##  Prerequisites

Ensure the following are installed:

```bash
docker --version
docker compose version
```

###  Output

```
bhargav@DESKTOP-QK0KB4I:/mnt/c/Users/...$ docker --version
Docker version 29.2.1, build a5c7197

bhargav@DESKTOP-QK0KB4I:/mnt/c/Users/...$ docker compose version
Docker Compose version v5.0.2
```

---

##  Project Setup

### Step 1 — Create Project Directory

```bash
mkdir docker-webapp
cd docker-webapp
```

---

### Step 2 — Create Required Directories

```bash
mkdir backend
mkdir database
ls
```

### Output

```
backend  database
```

---

##  Backend Structure

```
backend/
│
├── app.py                # FastAPI application
├── requirements.txt     # Python dependencies
├── Dockerfile          # Backend container build file
```

---

##  Database Structure

```
database/
│
├── Dockerfile          # PostgreSQL container setup
└── init.sql            # Initial schema & table creation
```

---

##  Step 3 — Create IPvlan Network

```bash
docker network create -d ipvlan \
--subnet=172.22.208.0/24 \
--gateway=172.22.208.1 \
-o parent=eth0 mynetwork
```

###  Explanation

* `ipvlan` → Lightweight network driver
* `subnet` → Defines IP range for containers
* `gateway` → Network gateway
* `parent=eth0` → Host network interface

---

##  Step 4 — Verify Network

```bash
docker network ls
docker network inspect mynetwork
```

---

##  Step 5 — Build & Run Containers

```bash
docker compose up --build
```

### What Happens Internally:

* Backend image is built using Python slim image
* PostgreSQL container initializes database
* Containers are attached to IPvlan network
* Static IPs are assigned

---

##  Step 6 — Verify Containers

```bash
docker ps
```

---

##  Step 7 — Check Container IPs

```bash
docker inspect backend_api | grep IPAddress
docker inspect postgres_db | grep IPAddress
```

### Expected Output

```
Backend: 172.22.208.11
Database: 172.22.208.10
```

---

##  API Testing

###  Health Check

```
GET http://172.22.208.11:8000/health
```

Response:

```json
{
  "status": "healthy"
}
```

---

###  Insert Data

```
POST /items?name=test
```

Response:

```json
{
  "message": "Item inserted successfully",
  "name": "test"
}
```

---

###  Fetch Data

```
GET /items
```

Response:

```json
[
  {
    "id": 1,
    "name": "test"
  }
]
```

---

##  Volume Persistence

### Steps:

```bash
docker compose down
docker compose up
```

✔ Data remains intact due to Docker volume.

---

##  Volume Verification

```bash
docker volume ls
```

Output:

```
docker-webapp_pgdata
```

---

##  Build Optimization Techniques

###  Multi-Stage Builds

* Reduces final image size
* Separates build and runtime layers

###  Slim Base Images

* `python:3.11-slim`
* Smaller attack surface
* Faster startup time

###  .dockerignore

* Excludes unnecessary files (logs, cache)

###  Non-root User

* Improves container security

### Layer Caching

* Faster rebuilds when dependencies unchanged

---

##  Image Size Comparison

| Image            | Size   |
| ---------------- | ------ |
| python:3.11      | ~1GB   |
| python:3.11-slim | ~150MB |

---

##  Macvlan vs IPvlan

| Feature     | Macvlan              | IPvlan             |
| ----------- | -------------------- | ------------------ |
| MAC Address | Unique per container | Shared             |
| Performance | Medium               | High               |
| Scalability | Limited              | High               |
| Use Case    | Small LAN            | Cloud / Production |

---

##  Key Learning Outcomes

* Docker containerization fundamentals
* Microservices communication using networking
* IPvlan configuration with static IPs
* Persistent storage with volumes
* API development using FastAPI
* DevOps best practices

---

##  Conclusion

This project successfully demonstrates:

* Fully containerized microservice architecture
* Efficient networking using IPvlan
* Scalable and portable deployment
* Persistent and reliable database storage

This setup is production-oriented and follows modern DevOps practices.

---


