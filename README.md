# Cloud Technologies Project - Microservices Architecture

## Overview

This project is a fully containerized, three-tier Full-Stack application (MERN stack: MongoDB, Express.js, React, Node.js) built with a microservices architecture. It serves as a practical demonstration of cloud-native best practices, focusing on containerization, orchestration, security, and data persistence using Docker and Kubernetes.

## Application Flow & Architecture

### 1. User Interface (Frontend)

* The user accesses the application through a web browser.
* The **React** frontend (built with Vite) loads the user interface and serves as the client-side interaction point.

### 2. Data Retrieval (GET)

* The Frontend sends a `GET /api/items` request to the Backend.
* The **Node.js/Express** Backend receives the request and establishes a connection with the **MongoDB** database using secure credentials injected via Environment Variables / Kubernetes Secrets.
* MongoDB queries the database and returns the 10 most recent items (`Item.find().sort({ createdAt: -1 }).limit(10)`).
* The Backend formats this data into JSON and sends it back to the Frontend, which renders the list dynamically.

### 3. Data Submission (POST)

* The user fills out the form and clicks "Add Item".
* The Frontend sends a `POST /api/items` request with a JSON payload (e.g., `{ "name": "Example", "value": 42 }`).
* The Backend validates and saves the new record in MongoDB.
* The new item instantly appears on the Frontend list without requiring a page reload.

## Key Features & Cloud-Native Practices

* **Security:**
  * Configured CORS with a dynamic list of allowed origins.
  * Separation of concerns: Database credentials are never hardcoded. They are managed securely via a local `.env` file for Docker Compose and `Secret` objects in Kubernetes.


* **Monitoring & Resilience:**
  * Dedicated health check endpoint (`/api/health`) for readiness/liveness probes.
  * Database auto-retry connection logic implemented in the backend.
  * Comprehensive request logging.


* **Scalability & Orchestration:**
  * Kubernetes **Horizontal Pod Autoscaler (HPA)** configured to scale the backend dynamically based on CPU utilization (>80%).
  * Multiple pod replicas for high availability.
  * Frontend exposed via a `LoadBalancer` service, and internal routing handled by an **NGINX Ingress Controller**.


* **Data Persistence:**
  * Docker Volumes mapped for local development.
  * Kubernetes **Persistent Volume Claims (PVC)** ensure MongoDB data survives pod restarts.


* **Optimized Containerization:**
  * Multi-stage Docker builds utilizing lightweight Alpine Linux images to minimize attack surface and image size.



## API Endpoints

| Endpoint | Method | Description |
| --- | --- | --- |
| `/` | GET | Serves the React User Interface |
| `/api/health` | GET | Returns the service health and DB connection status |
| `/api/items` | GET | Fetches the list of recent items |
| `/api/items` | POST | Adds a new item to the database |

---

## Getting Started / Run Instructions

### Prerequisites

* [Docker Desktop](https://www.docker.com/products/docker-desktop/) (with Kubernetes enabled if running the K8s cluster locally)
* A [Docker Hub](https://hub.docker.com/) account
* `kubectl` CLI tool

### Step 1: Environment Preparation

1. **Clone the repository** to your local machine.
2. **Setup Environment Variables:** Create a `.env` file in the root directory (alongside `docker-compose.yaml`) with the following secure credentials:
```env
MONGO_INITDB_ROOT_USERNAME=admin
MONGO_INITDB_ROOT_PASSWORD=secret
DB_USER=admin
DB_PASS=secret
DB_HOST=mongo
DB_PORT=27017
DB_NAME=appdb
PORT=5000

```


3. **Build and Push Docker Images:**
*Navigate to the backend folder:*
```bash
cd backend
docker buildx build --platform linux/amd64,linux/arm64 -t <your-dockerhub-username>/microservices-backend:latest --push .

```


*Navigate to the frontend folder:*
```bash
cd ../frontend
docker buildx build --platform linux/amd64,linux/arm64 -t <your-dockerhub-username>/microservices-frontend:latest --push .

```



### Step 2: Running with Docker Compose (Local Development)

1. **Start the application:**
Navigate back to the root directory and run:
```bash
docker-compose up --build

```


2. **Access the application:**
* Frontend UI: [http://localhost:3000](https://www.google.com/search?q=http://localhost:3000)
* Backend API / Health: [http://localhost:5000/api/health](https://www.google.com/search?q=http://localhost:5000/api/health)


3. **Stop the application:**
```bash
docker-compose down -v

```



### Step 3: Running with Kubernetes

1. Ensure a local Kubernetes cluster (like the one built into Docker Desktop or Minikube) is running.
2. **Update Image References:** Edit `k8s/04-backend-deployment.yaml` and `k8s/07-frontend-deployment.yaml`. Replace the `image:` placeholders with your actual pushed Docker Hub image names.
3. **Apply the Kubernetes Manifests:**
Apply the entire configuration directory:
```bash
kubectl apply -f k8s/

```


4. **Verify the Deployment:**
Check if all pods, services, and the ingress controller are running correctly:
```bash
kubectl get all
kubectl get ingress
kubectl describe ingress app-ingress

```


5. **Access the application:**
Open your browser and navigate to: [http://localhost](https://www.google.com/search?q=http://localhost)
