# Lab 2 — Containerization & Kubernetes Orchestration

**Student:** Amit Mohanty
**Roll No.:** 2301730325
**Tools used:** Docker Desktop, Docker CLI, Docker Compose, VSCode, Windows PowerShell / Git Bash, Python, Flask

This repository contains the work for Lab 2, covering four tasks on Docker containerization: building and running a containerized application, managing containers/images and using Docker Compose, working with Docker storage (volumes and bind mounts), and setting up Docker networking between containers.

---

## Repository Structure

```
.
├── app/
│   ├── app.py              # Flask application used across Tasks 1 and 2
│   ├── requirements.txt    # Python dependencies (Flask)
│   └── Dockerfile          # Image definition for the Flask app
├── k8s/
│   └── deployment.yaml     # Kubernetes Deployment + Service
├── docs/                   # Lab report documents (.docx)
├── docker-compose.yml      # Multi-container setup (Flask web + Redis) — Task 2
└── README.md
```

---

## Task 1: Containerize a Sample Application Using Docker

A simple Flask application was containerized from scratch.

- Wrote `app.py` (a Flask app with `/` and `/health` routes) and `requirements.txt`.
- Wrote a `Dockerfile` using `python:3.11-slim` as the base image, installing dependencies and running the app on port 5000.
- Built the image:
  ```bash
  docker build -t flask-docker-app:1.0 ./app
  ```
- Ran it as a container with port mapping:
  ```bash
  docker run -d --name flask-container -p 5050:5000 flask-docker-app:1.0
  ```
- Verified the app in the browser (`/` and `/health`), inspected logs (`docker logs`) and image layers (`docker history`), and tested the stop/start container lifecycle.

**Result:** Image `flask-docker-app:1.0` built and verified; container lifecycle (stop/start) confirmed working without rebuilding the image.

---

## Task 2: Manage Docker Containers and Images

Built on Task 1 by managing images/containers and introducing Docker Compose.

- Listed containers and images (`docker ps`, `docker ps -a`, `docker images`).
- Tagged the image for Docker Hub and pushed it:
  ```bash
  docker tag flask-docker-app:1.0 amitmiv/flask-docker-app:1.0
  docker push amitmiv/flask-docker-app:1.0
  ```
- Created `docker-compose.yml` defining two services — a Redis cache (`flask-redis`) and the Flask web app (`flask-web`), built from the existing `Dockerfile`.
- Brought up the multi-container setup:
  ```bash
  docker compose up -d
  docker compose ps
  ```
- Verified the app in the browser at `http://localhost:5001`.

**Result:** Image pushed to Docker Hub at [`amitmiv/flask-docker-app`](https://hub.docker.com/r/amitmiv/flask-docker-app); multi-container setup (web + redis) running via Docker Compose.

---

## Task 3: Docker Storage

Demonstrated the difference between ephemeral container storage and persistent storage options.

- **Container-layer data loss:** Wrote a file inside a plain container (`storage-test`), removed the container, and confirmed the data was gone when a new container was started from the same image.
- **Named volume:** Created a Docker volume (`app-data`), mounted it into a container, wrote data to it, removed the container, and confirmed the data persisted by mounting the same volume into a fresh container.
- **Bind mount:** Mounted a folder from the host machine into a container, wrote data from inside the container, and confirmed the file was visible directly on the host filesystem after the container was removed.

**Result:** Confirmed that container-layer writes are lost on removal, while both named volumes and bind mounts persist data independently of the container's lifecycle.

---

## Task 4: Docker Network

Demonstrated container-to-container communication over a custom Docker network.

- Created a user-defined bridge network:
  ```bash
  docker network create task4-network
  ```
- Ran an `nginx:alpine` container (`web-server`) on that network.
- Ran a second container (`network-client`) on the same network and reached the web server:
  - By container name: `wget -qO- http://web-server`
  - By container IP address (resolved via `docker inspect`): `wget -qO- http://<web-server-ip>`
- Inspected the network (`docker network inspect task4-network`) to confirm subnet, gateway, and connected containers.

**Result:** Verified that containers on the same user-defined network can communicate both by container name (via Docker's built-in DNS) and by IP address.

---

## Notes

- Commands were run on Windows using Git Bash (MINGW64); commands with path-like arguments were prefixed with `MSYS_NO_PATHCONV=1` to prevent Git Bash from rewriting Docker-internal paths (e.g. `/data`) as Windows paths.
- Full command logs and screenshots for each task are included in the accompanying lab report documents.
