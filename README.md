# Secure AI Proxy Gateway (Kong + FastAPI)

A standalone, enterprise-grade infrastructure pattern demonstrating how to decouple API traffic management, security, and rate-limiting from core AI Agent applications using **Kong API Gateway** and **FastAPI**.

## System Architecture

This project utilizes a completely decoupled, containerized edge gateway pattern:

1. **Client Layer:** Sends user prompt payloads via HTTP requests to the public edge gateway.
2. **Gateway Layer (Kong):** Operates in a lightweight, declarative (DB-less) Docker container on port `8000`. It actively checks incoming traffic using local plugins.
3. **Upstream AI Layer (FastAPI):** A Python application running locally on port `8005` that processes validated requests and executes mock agentic reasoning.

## Core Features Implemented

* **Declarative Configuration:** Managed entirely via a `kong.yml` manifest file without database overhead.
* **Infrastructure Rate-Limiting:** Attached a native Kong plugin restricting traffic to a maximum of **5 requests per minute** to protect downstream LLM compute and budgets.
* **Network Bridging:** Integrated container-to-host networking via `host.docker.internal` to route gateway traffic back safely to local application layers.

---

## Getting Started

### Prerequisites
* [Docker Desktop](https://www.docker.com/products/docker-desktop/) installed and running.
* Python 3.10+ with an active virtual environment (`venv`).

### 1. Start the Kong Gateway
Navigate to the root directory and boot up the containerized infrastructure:
```bash
docker compose up -d

```
Verify Kong is running successfully by checking the admin health status endpoint:
  http://localhost:8001/status

2. Launch the Upstream AI Service
Install your dependencies, ensure your virtual environment is active, and launch the FastAPI server:

```bash 
python app.py
```
The application will begin listening on http://127.0.0.1:8005.

Testing the Infrastructure
Happy Path (Successful Request)
Fire a valid prompt payload directly at the Kong Gateway (Port 8000) using PowerShell:

```powershell
Invoke-RestMethod -Method Post -Uri "http://localhost:8000/ai-agent" -ContentType "application/json" -Body '{"prompt": "Hello AI Agent, check my system health."}'
```
Expected Response: A clean JSON output echoing back your validated agent response.

Throttling Path (Rate Limit Exceeded)
Execute the above command rapidly 6 or more times within one minute.

Expected Behavior: On the 6th attempt, Kong blocks the request at the gateway boundary, completely insulating your Python backend. The terminal will immediately receive:

JSON
{ "message": "API rate limit exceeded" }
