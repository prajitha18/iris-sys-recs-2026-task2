# Containerized Microservice Infrastructure
This project implements a production-style containerized infrastructure
using Docker Compose.

The system includes:

• NGINX reverse proxy
• Load balanced application replicas
• MySQL database
• Grafana + Prometheus monitoring stack
• NFS shared storage
• Automated backup service

All services follow the Principle of Least Privilege through network isolation.
Only NGINX is exposed to the host machine.
## System Architecture

The infrastructure consists of multiple containerized services
connected through isolated Docker networks.

External traffic enters through NGINX which acts as a reverse proxy
and load balancer.

## Network Architecture

The system is divided into multiple Docker networks
to follow the Principle of Least Privilege.

### Public Network
Accessible from the host machine.

Services:
• NGINX

### Application Network
Used for communication between the reverse proxy,
application replicas, and database.

Services:
• NGINX
• Application replicas
• MySQL database

### Monitoring Network
Used by monitoring services.

Services:
• Prometheus
• Grafana
• Exporters
NGINX is connected to both the public and application networks
so it can receive external traffic and forward requests internally.
## Reverse Proxy & Load Balancing

NGINX is configured as the single entry point to the system.

Features implemented:

• Load balancing across 3 application replicas
• Health checks with automatic failover
• Graceful reloads
• Rate limiting to prevent abuse
• Subdomain based routing
## Access Control

Sensitive internal services such as Grafana are protected
using HTTP Basic Authentication configured at the NGINX level.

Authentication is implemented using a .htpasswd file.
## Running the Project

Clone the repository:

git clone <repo-url>

Start all services:

docker compose up -d
Then access:

http://app.localhost
http://grafana.localhost