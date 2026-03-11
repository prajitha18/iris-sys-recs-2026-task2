# Containerized Microservice Infrastructure (Task-2)

## Overview
This project implements a production-style containerized infrastructure using Docker Compose.  
The system demonstrates reverse proxy routing, load balancing, network isolation, monitoring, and access control.

The infrastructure includes:

- NGINX Reverse Proxy
- Application service (3 replicas)
- MySQL database
- NFS shared storage
- Monitoring stack (Prometheus + Grafana)
- Automated backup service

The architecture follows the **Principle of Least Privilege**, ensuring that each service only has access to the networks and resources it strictly requires.

---

# System Architecture

External traffic enters the system through the reverse proxy and is routed internally to the appropriate service.

```
                Client Browser
                       │
                       ▼
                NGINX Reverse Proxy
                   (Public Network)
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
      App1           App2           App3
        │              │              │
        └──────────────┴──────────────┘
                     │
                  MySQL
                     │
                 NFS Storage
                     │
                Backup Service

Monitoring Stack
Prometheus  →  Grafana
```

---

# Network Architecture

The infrastructure uses multiple Docker networks to isolate services and enforce security.

## Public Network
Purpose: Entry point for external traffic.

Services:
- NGINX

Only NGINX exposes ports **80/443** to the host machine.

All other services remain private.

---

## Application Network
Purpose: Communication between the reverse proxy, application replicas, and database.

Services:
- NGINX
- Application replicas
- MySQL

This network ensures that application services can communicate internally without being exposed externally.

---

## Storage Network
Purpose: Shared storage access.

Services:
- Application replicas
- NFS server
- Backup service

This network allows the application containers and backup service to access persistent storage.

---

## Monitoring Network
Purpose: Monitoring and observability.

Services:
- Prometheus
- Grafana
- exporters

This network collects metrics and visualizes them using dashboards.

---

# Multi-Network Attachments

Some services must connect to multiple networks to perform their roles.

### NGINX
Connected to:
- Public network
- Application network

Reason:  
Receives external traffic from the public network and forwards requests to internal services.

---

### Application Replicas
Connected to:
- Application network
- Storage network

Reason:  
Application containers need to access both the database and shared storage.

---

### Backup Service
Connected to:
- Storage network

Reason:  
Allows periodic backups of shared storage and database data.

---

# Reverse Proxy and Load Balancing

The system uses NGINX as a reverse proxy and load balancer.

Features implemented:

- Load balancing across **3 application replicas**
- Automatic failover
- Graceful configuration reload
- Rate limiting (returns HTTP 429 if exceeded)
- Subdomain-based routing

Example routes:

```
app.localhost      → Application service
grafana.localhost  → Grafana dashboard
```

Requests are distributed between the three application replicas using an upstream configuration.

---

# Access Control

Monitoring dashboards are protected using **HTTP Basic Authentication** configured at the NGINX level.

Authentication is implemented using a `.htpasswd` file.

This ensures that internal tools like Grafana are not publicly accessible.

---

# Backup Strategy

A dedicated backup service periodically creates backups of:

- MySQL database
- Shared storage

Backups are stored on the **NFS shared storage server**, ensuring persistence even if containers restart.

---

# Monitoring and Observability

The monitoring stack includes:

- Prometheus for metrics collection
- Grafana for visualization

Prometheus scrapes metrics from exporters and internal services.  
Grafana connects to Prometheus and provides dashboards for system monitoring.

---

# Security and Production Practices

Several best practices were followed:

- Only NGINX is exposed to the host
- Internal services remain isolated within Docker networks
- Principle of Least Privilege applied to networking
- Monitoring included for observability
- Access control implemented for internal services
- Secrets are not hardcoded in configuration files

---

# Running the Project

Clone the repository:

```
git clone <repository-url>
cd iris-sys-recs-2026-task2
```

Start all services:

```
docker compose up -d
```

Verify running containers:

```
docker ps
```

---

# Accessing Services

Application service:

```
http://app.localhost
```

Monitoring dashboard:

```
http://grafana.localhost
```


---

# Conclusion

This project demonstrates a production-style containerized infrastructure with reverse proxy routing, load balancing, monitoring, and secure network design.

The architecture prioritizes **security, scalability, and maintainability**, similar to real-world DevOps environments.