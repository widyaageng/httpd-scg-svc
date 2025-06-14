# SCG Compose: Full-Stack Reverse Proxy, API Gateway, and Streamlit App with Docker Compose

## Overview

This repository provides a complete, production-grade example of integrating a Python Streamlit web application, a Spring Cloud Gateway (Java) API gateway, and an Apache HTTPD reverse proxy, all orchestrated via Docker Compose. The setup demonstrates advanced reverse proxying, WebSocket support, and service-to-service communication using Docker networking.

---

## Architecture

```
[Client]
   |
   v
[Apache HTTPD (httpd)]
   |
   +-- /streamlit/*, WebSocket --> [Spring Cloud Gateway (gateway)] --> [Streamlit App (streamlit)]
   +-- / (other HTTP) ----------> [Spring Cloud Gateway (gateway)] --> [Streamlit App (streamlit)]
```

- **httpd**: Acts as the public-facing reverse proxy, handling HTTP and WebSocket traffic, and forwarding requests to the gateway or directly to Streamlit as needed.
- **gateway**: A Spring Cloud Gateway (Java 17, Spring Boot 3.x) that provides API gateway features, path rewriting, and service routing.
- **streamlit**: A Python 3.11 Streamlit application, demonstrating a simple interactive UI and WebSocket support.

---

## Directory Structure

```
.
├── docker-compose.yaml
├── gateway/
│   ├── Dockerfile
│   ├── pom.xml
│   ├── src/
│   │   └── main/
│   │       ├── java/com/example/gateway/GatewayApplication.java
│   │       └── resources/application.yaml
│   └── target/gateway-0.0.1-SNAPSHOT.jar
├── httpd/
│   ├── httpd.conf
│   └── httpd-docker.conf
├── streamlit/
│   ├── Dockerfile
│   ├── app.py
│   ├── index.html
│   └── requirements.txt
└── README.md
```

---

## Service Details

### 1. Apache HTTPD (`httpd`)

- **Config:** `httpd/httpd-docker.conf`
- **Role:** Reverse proxy for HTTP and WebSocket traffic.
- **Key Features:**
  - Uses Docker service hostnames (`gateway`, `streamlit`) for backend routing.
  - Handles WebSocket upgrade and forwards `/streamlit/_stcore/stream` to the gateway.
  - Uses balancer and rewrite rules for advanced proxying.
  - Logs to stdout/stderr for Docker compatibility.
- **Docker Image:** `httpd:2.4`

### 2. Spring Cloud Gateway (`gateway`)

- **Config:** `gateway/src/main/resources/application.yaml`
- **Role:** API gateway, path rewriting, and service routing.
- **Key Features:**
  - Java 17, Spring Boot 3.x, Spring Cloud Gateway.
  - Forwards `/streamlit/**` to the Streamlit service, strips and rewrites paths as needed.
  - Configurable via externalized YAML (mounted in Docker).
- **Build:** Uses Maven (`pom.xml`), produces `gateway-0.0.1-SNAPSHOT.jar`.
- **Dockerfile:** Multi-stage, copies built JAR and config.

### 3. Streamlit App (`streamlit`)

- **Main App:** `streamlit/app.py`
- **Role:** Python web application with WebSocket and interactive UI.
- **Key Features:**
  - Simple demo: text input and greeting.
  - All dependencies in `requirements.txt`.
  - Custom `index.html` for static content.
- **Dockerfile:** Based on `python:3.11-slim`, installs Streamlit and runs the app.

---

## Running the Stack

### Prerequisites

- Docker and Docker Compose installed.
- (Optional) Java 17 and Maven for local gateway development.

### Build and Start All Services

```bash
docker-compose up --build
```

- HTTPD will be available on [http://localhost](http://localhost)
- Gateway on [http://localhost:8080](http://localhost:8080)
- Streamlit on [http://localhost:8501](http://localhost:8501)

### Service Endpoints

- **Public (via HTTPD):**
  - `http://localhost/streamlit/` → Streamlit app (proxied)
  - WebSocket: `ws://localhost/streamlit/_stcore/stream`
- **Direct (for debugging):**
  - `http://localhost:8080/streamlit/` → Gateway to Streamlit
  - `http://localhost:8501/` → Streamlit direct

---

## Development Notes

- **Spring Cloud Gateway**: Edit `application.yaml` for route changes. Rebuild the JAR and restart the container for changes to take effect.
- **Streamlit**: Edit `app.py` and `requirements.txt` as needed. Rebuild the image for dependency changes.
- **HTTPD**: Edit `httpd-docker.conf` for proxy or balancer changes. The config is mounted read-only in the container.

---

## Advanced Features

- **WebSocket Proxying**: Fully supported from HTTPD through the gateway to Streamlit.
- **Path Rewriting**: Gateway rewrites `/streamlit/**` to `/` for Streamlit compatibility.
- **Docker Networking**: All services communicate via Docker Compose service names.
- **Production-Ready Logging**: HTTPD logs to Docker stdout/stderr.

---

## Troubleshooting

- **HTTPD Fails to Start**: Check for missing modules in `httpd-docker.conf`. The official image requires `LoadModule` lines with `modules/` prefix.
- **Gateway 404s**: Ensure `application.yaml` is mounted and the route is correct.
- **WebSocket 400/200 Errors**: Ensure correct `ProxyPass` order and that the backend supports WebSocket at the target path.
- **Permission Issues**: All paths and log files are Docker-compatible.

---

## License

- Streamlit and all code in this repo are under the Apache 2.0 License.

---

## Authors

- [![WA](https://img.shields.io/badge/GitHub-@widya-black?logo=github)](https://github.com/widyaageng)
- [LELEM](https://en.wikipedia.org/wiki/GPT-4) 

---

Feel free to copy, modify, and extend this stack for your own production or development needs!
