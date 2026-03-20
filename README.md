# DOCKER-WEBAPP

Containerized Web Application with PostgreSQL using Docker Compose and IPvlan

Project Assignment 1 — Containerization and DevOps

---

## Project Description

This project demonstrates the deployment of a containerized web application using modern DevOps practices.

The application consists of:

* FastAPI Backend for handling API requests
* PostgreSQL Database for persistent data storage
* Docker and Docker Compose for container orchestration
* IPvlan Networking for static IP-based communication

Each service runs in an isolated container and communicates through a custom-defined IPvlan network.

---

## Project Architecture

```
Client (Browser/Postman)
        │
        │ HTTP Request
        ▼
Backend Container (FastAPI)
IP: 172.22.208.11
        │
        │ SQL Query
        ▼
PostgreSQL Container
IP: 172.22.208.10
        │
        ▼
Docker Volume (pgdata)
```

---

## Prerequisites

Ensure Docker and Docker Compose are installed:

```bash
docker --version
docker compose version
```

### Output

```
bhargav@DESKTOP-QK0KB4I:/mnt/c/Users/...$ docker --version
Docker version 29.2.1, build a5c7197

bhargav@DESKTOP-QK0KB4I:/mnt/c/Users/...$ docker compose version
Docker Compose version v5.0.2
```

---

## Step 1 — Create Project Directory

```bash
mkdir docker-webapp
cd docker-webapp
```

---

## Step 2 — Create Required Directories

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

## Step 3 — Backend Structure

```
backend/
│
├── app.py              # FastAPI application entry point
├── requirements.txt   # Python dependencies
├── Dockerfile        # Backend container build file
```

### Output

```
bhargav@DESKTOP-QK0KB4I:/mnt/c/.../backend$ ls
app.py
requirements.txt
Dockerfile
```

---

## Step 4 — Database Structure

```
database/
│
├── Dockerfile        # PostgreSQL setup
└── init.sql          # Schema initialization script
```

### Output

```
bhargav@DESKTOP-QK0KB4I:/mnt/c/.../database$ ls
Dockerfile
init.sql
```

---

## Step 5 — Create IPvlan Network

```bash
docker network create -d ipvlan \
--subnet=172.22.208.0/24 \
--gateway=172.22.208.1 \
-o parent=eth0 mynetwork
```

### Explanation

* ipvlan: Lightweight network driver
* subnet: Defines container IP range
* gateway: Network gateway
* parent=eth0: Host network interface

---

## Step 6 — Verify Network

```bash
docker network ls
```

### Output

```
NETWORK ID     NAME        DRIVER    SCOPE
xxxxxx         mynetwork   ipvlan    local
```

---

## Step 7 — Inspect Network

```bash
docker network inspect mynetwork
```

### Output

```
{
  "Driver": "ipvlan",
  "Subnet": "172.22.208.0/24",
  "Gateway": "172.22.208.1"
}
```

---

## Step 8 — Build and Start Containers

```bash
docker compose up --build
```

### Internal Workflow

* Backend image is built using a multi-stage Dockerfile
* PostgreSQL container initializes database schema
* Containers connect to the IPvlan network
* Static IP addresses are assigned

---

## Step 9 — Verify Running Containers

```bash
docker ps
```

### Output

```
backend_api   Up
postgres_db   Up
```

---

## Step 10 — Verify Container IP Addresses

```bash
docker inspect backend_api | grep IPAddress
docker inspect postgres_db | grep IPAddress
```

### Output

```
Backend → 172.22.208.11
Database → 172.22.208.10
```

---

## Step 11 — API Testing

### Health Check

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

## Step 12 — Insert Record

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

## Step 13 — Fetch Records

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

## Step 14 — Volume Persistence Test

```bash
docker compose down
docker compose up
```

### Output

```
Data persists successfully
```

Data remains intact due to Docker volume.

---

## Step 15 — Volume Verification

```bash
docker volume ls
```

### Output

```
docker-webapp_pgdata
```

---

## Build Optimization Techniques

* Multi-stage builds reduce final image size
* Slim base images reduce unnecessary dependencies
* .dockerignore avoids copying unnecessary files
* Layer caching improves rebuild speed
* Non-root execution improves container security

---

## Image Size Comparison

| Image            | Approx Size |
| ---------------- | ----------- |
| python:3.11      | ~1GB        |
| python:3.11-slim | ~150MB      |

---

## IPvlan Networking Details

* Containers share the host MAC address
* Static IP addresses are manually assigned
* No automatic DNS resolution like bridge networks
* Suitable for advanced networking and production environments

### Communication Flow

```
Backend → 172.22.208.10 → PostgreSQL
```

---

## Macvlan vs IPvlan

| Feature      | Macvlan              | IPvlan |
| ------------ | -------------------- | ------ |
| MAC Address  | Unique per container | Shared |
| Performance  | Medium               | High   |
| Scalability  | Limited              | High   |
| Network Load | Higher               | Lower  |

---

## Key Learning Outcomes

* Containerization using Docker
* Service orchestration using Docker Compose
* Advanced networking using IPvlan
* Persistent storage using Docker volumes
* API development using FastAPI
* DevOps best practices

---

## Conclusion

This project demonstrates a complete containerized application deployment using Docker and IPvlan networking.

It ensures scalability, portability, and reliable data persistence while following modern DevOps practices.

---
