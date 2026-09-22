# Multi-Tier Containerized MERN Architecture on AWS EC2

An end-to-end containerized production deployment of a 3-tier MERN stack (MongoDB, Express, React, Node.js) orchestrated using Docker Compose on an AWS EC2 host.

This repository highlights hands-on infrastructure design, isolated bridge networking, stateful volume management, and resolution of real-world distributed networking challenges (client-side cross-origin routing and container engine build dependencies).

---

 🏗 System Architecture

```text
+-------------------------------------------------------+
|                Remote Client Browser                  |
+-------------------------------------------------------+
       |                                   |
       | HTTP (Port 5173)                  | HTTP (Port 5050)
       v                                   v
+-------------------------------------------------------+
|                  AWS EC2 Instance                     |
|                                                       |
|  [ Docker Host - Custom Network: mern_network ]       |
|                                                       |
|  +-------------------+       +---------------------+  |
|  |     Frontend      |       |       Backend       |  |
|  |   (React / Vite)  |       |  (Express / Node)   |  |
|  +-------------------+       +----------+----------+  |
|                                         |             |
|                                         | mongodb:    |
|                                         | 27017       |
|                                         v             |
|                              +---------------------+  |
|                              |       MongoDB       |  |
|                              |       (v7.0+)       |  |
|                              +----------+----------+  |
|                                         |             |
|                                         v (Persists)  |
|                              +---------------------+  |
|                              | Named Volume        |  |
|                              | (mongo-data)        |  |
|                              +---------------------+  |
+-------------------------------------------------------+


⚙️ Key Technical Highlights

 Container Orchestration: Single-command lifecycle management (docker compose) linking multi-service microcomponents.

 Isolated Networking: Custom bridge network (mern_network) isolating database traffic. The database container interacts only within the Docker internal DNS space (mongodb:27017) and does not rely on public network exposure.

 Persistent Storage: Stateful MongoDB data persistence via Docker named volumes (mongo-data:/data/db), ensuring zero data loss during container updates or host rebuilds.

 Service Lifecycle Ordering: Explicit startup dependencies via depends_on ensuring reliable database readiness before backend initialization.


🛠 Engineering Challenges & Troubleshooting

1. Client-Side API Resolution (Browser Execution vs Host Execution)
   Issue: After spinning up containers on AWS EC2, employee record submissions failed. Frontend React code executes directly inside the end-user's remote browser, not on the server. Hardcoded localhost:5050 requests were reaching the client's local machine rather than the EC2 host.

   Resolution: Decoupled endpoints from localhost, updated API targets in Record.jsx and RecordList.jsx to dynamically target the public host interface, rebuilt the frontend image, and updated AWS Security Group ingress rules to allow traffic on port 5050.

2. MongoDB v7+ Socket Inspection vs HTTP Deprecation
   Issue: Probing curl -I http://localhost:27017 returned Empty reply from server, leading to false negatives during container health verification.

   Resolution: Identified that modern MongoDB versions have completely deprecated legacy HTTP status servers. Switched verification to verbose socket connection checks (curl -v http://localhost:27017) and backend driver connection status logs.

3. Missing Buildx CLI Runtime in Cloud Linux
   Issue: Running docker compose up --build failed immediately with compose build requires buildx 0.17.0 or later on minimal cloud Linux AMIs.

   Resolution: Configured the missing Docker CLI plugins directory and installed the official Buildx binary (~/.docker/cli-plugins/docker-buildx), restoring native multi-image compilation through Docker Compose v2.

 🚀 Quickstart:

  Prerequisites

   Docker Engine & Docker Compose v2

   Open EC2 Inbound Ports: 22 (SSH), 5173 (Frontend), 5050 (Backend API)

Deployment Steps 

Bash
# 1. Clone repository
  git clone [https://github.com/pkp0023/mern-docker-compose-deployment.git](https://github.com/pkp0023/mern-docker-compose-deployment.git)
  cd mern-docker-compose-deployment

# 2. Deploy multi-container stack
  docker compose up -d --build

# 3. Verify running services
  docker compose ps

Access the UI at: http://<EC2-PUBLIC-IP>:5173
