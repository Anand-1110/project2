# 🚀 React App — Full DevOps CI/CD Pipeline

> A production-grade React application demonstrating an end-to-end DevOps workflow: containerized with Docker using multi-stage builds, served via Nginx with gzip compression, automated through GitHub Actions CI/CD, and deployed to Kubernetes.

---

## 📌 Project Overview

This project showcases a complete **DevOps lifecycle** for a modern React (Vite) frontend application — from source code to a production-ready deployment. The emphasis is on infrastructure, automation, and container orchestration rather than the application logic itself.

**Key DevOps concepts demonstrated:**
- Multi-stage Docker builds for minimal production image size
- Nginx as a production-grade static file server with performance tuning
- GitHub Actions for automated CI/CD pipeline (build → push → deploy)
- Kubernetes manifests for scalable, declarative container orchestration
- SWC-powered Vite builds for fast, optimized frontend bundling

---

## 🏗️ Architecture & Pipeline

```
Developer Push
      │
      ▼
┌─────────────────────┐
│   GitHub Actions     │  ← Triggered on push to main
│   CI/CD Pipeline     │
└────────┬────────────┘
         │
    ┌────▼────┐      ┌──────────────┐
    │  Build   │─────▶  Container   │  ← Push Docker image
    │  & Test  │      │  Registry   │
    └─────────┘      └──────┬───────┘
                            │
                     ┌──────▼───────┐
                     │  Kubernetes  │  ← Apply k8s manifests
                     │   Cluster    │
                     └──────┬───────┘
                            │
                     ┌──────▼───────┐
                     │  Nginx Pod   │  ← Serves React SPA
                     │  (Container) │
                     └─────────────┘
```

---

## 🛠️ Tech Stack

| Layer | Technology | Purpose |
|---|---|---|
| **Frontend** | React 19 + Vite 7 (SWC) | UI framework with fast HMR builds |
| **Web Server** | Nginx 1.17 Alpine | Serve static files, gzip, SPA routing |
| **Containerization** | Docker (multi-stage) | Lightweight, reproducible builds |
| **Orchestration** | Kubernetes (k8s) | Scalable, self-healing deployments |
| **CI/CD** | GitHub Actions | Automated build-test-deploy pipeline |
| **Linting** | ESLint 9 | Code quality enforcement |

---

## 📂 Project Structure

```
project2/
│
├── .github/
│   └── workflows/
│       └── *.yml              # GitHub Actions CI/CD pipeline definitions
│
├── k8s/
│   ├── deployment.yml         # Kubernetes Deployment manifest
│   ├── service.yml            # Kubernetes Service (LoadBalancer/ClusterIP)
│   └── ...                    # Additional k8s resources
│
├── public/                    # Static public assets
├── src/                       # React application source code
│
├── Dockerfile                 # Multi-stage Docker build
├── nginx.conf                 # Custom Nginx configuration
├── index.html                 # Vite HTML entry point
├── vite.config.js             # Vite bundler configuration
├── eslint.config.js           # ESLint rules
└── package.json               # Dependencies and npm scripts
```

---

## 🐳 Docker — Multi-Stage Build

The Dockerfile uses a **two-stage build** to keep the final image as small as possible.

**Stage 1 — Build** (`node:22-alpine`):
- Installs npm dependencies
- Runs `vite build` to produce optimized static files in `/app/dist`

**Stage 2 — Serve** (`nginx:1.17.1-alpine`):
- Copies only the compiled `dist/` output — no Node.js, no source code
- Uses a custom `nginx.conf` for production-grade serving

This results in a **minimal final image** (~25 MB) compared to shipping a full Node environment (~300 MB+).

```dockerfile
# Stage 1: Build
FROM node:22-alpine AS build
WORKDIR /app
COPY package*.json .
RUN npm install
COPY . ./
RUN npm run build

# Stage 2: Serve
FROM nginx:1.17.1-alpine
COPY nginx.conf /etc/nginx/nginx.conf
COPY --from=build /app/dist /usr/share/nginx/html
```

### Build & Run Locally

```bash
# Build the image
docker build -t project2:latest .

# Run the container
docker run -d -p 80:80 --name project2 project2:latest

# Access the app
open http://localhost
```

---

## ⚙️ Nginx Configuration

The custom `nginx.conf` is tuned for production SPA hosting:

| Feature | Detail |
|---|---|
| **Gzip compression** | Enabled for HTML, CSS, JS, XML — reduces transfer size ~70% |
| **SPA routing** | `try_files $uri $uri/ /index.html` handles React Router client-side navigation |
| **MIME types** | Correct `Content-Type` headers for all asset types |
| **Legacy browser** | Gzip disabled for MSIE 1–6 to prevent rendering issues |
| **Port** | Listens on port `80` |

---

## ☸️ Kubernetes Deployment

Kubernetes manifests are in the `k8s/` directory. Apply all resources with:

```bash
# Deploy to cluster
kubectl apply -f k8s/

# Verify deployment
kubectl get deployments
kubectl get pods
kubectl get services

# Watch rollout progress
kubectl rollout status deployment/project2
```

### Resources Defined

- **Deployment** — Declares desired pod count, container image, resource limits, and update strategy
- **Service** — Exposes the deployment (LoadBalancer for cloud clusters, NodePort for local)

---

## 🔄 CI/CD Pipeline — GitHub Actions

The pipeline in `.github/workflows/` automates the full delivery process on every push to `main`:

```
┌──────────┐    ┌──────────┐    ┌─────────────┐    ┌────────────────┐
│ Checkout  │───>│  Install  │───>│  Build &    │───>│ Docker Build   │
│   Code    │    │  & Lint   │    │  npm build  │    │   & Push       │
└──────────┘    └──────────┘    └─────────────┘    └───────┬────────┘
                                                            │
                                                   ┌────────▼────────┐
                                                   │ kubectl apply   │
                                                   │ (Deploy to k8s) │
                                                   └─────────────────┘
```

**Pipeline stages:**

1. **Checkout** — Pull the latest source code
2. **Install & Lint** — `npm install` + ESLint code quality check
3. **Build** — `npm run build` (Vite produces optimized `dist/`)
4. **Docker Build & Push** — Build image, tag with commit SHA, push to registry
5. **Deploy** — `kubectl apply -f k8s/` updates the running cluster

---

## 💻 Local Development

### Prerequisites

- [Node.js](https://nodejs.org/) v18+
- [Docker](https://www.docker.com/) (for container testing)
- [kubectl](https://kubernetes.io/docs/tasks/tools/) + cluster access (for k8s)

### Setup

```bash
# Clone the repository
git clone https://github.com/Anand-1110/project2.git
cd project2

# Install dependencies
npm install

# Start dev server with Hot Module Replacement
npm run dev
# → http://localhost:5173
```

### Available Scripts

| Command | Description |
|---|---|
| `npm run dev` | Start Vite dev server with HMR |
| `npm run build` | Production build — outputs to `dist/` |
| `npm run preview` | Serve the production build locally |
| `npm run lint` | Run ESLint across the codebase |

---

## 🔑 Key DevOps Concepts Demonstrated

**1. Immutable Infrastructure**
Docker images are tagged by commit SHA. Every deployment is fully traceable back to an exact code state — no surprises, no manual changes on servers.

**2. Separation of Build and Runtime**
The build environment (Node.js) is completely discarded after compilation. The production image contains only Nginx and the compiled static files — zero build tools in production.

**3. Declarative Deployments**
Kubernetes manifests describe the *desired state*. The cluster continuously reconciles towards it, providing self-healing if pods crash or nodes fail.

**4. Pipeline as Code**
The entire CI/CD process lives in version-controlled YAML. There are no manual deployment steps and no snowflake servers — the pipeline is reproducible by anyone on the team.

**5. Twelve-Factor App Principles**
Config is separated from code, containers are stateless, and a single codebase is tracked in Git — aligned with cloud-native best practices.

---

## 📋 Prerequisites Summary

| Tool | Version | Purpose |
|---|---|---|
| Node.js | ≥ 18 | Local development |
| Docker | ≥ 20 | Build and run containers |
| kubectl | ≥ 1.25 | Interact with Kubernetes |
| Kubernetes cluster | — | Minikube / kind / EKS / GKE / AKS |

---

## 👤 Author

**Anand** — [GitHub @Anand-1110](https://github.com/Anand-1110)

---

> ⭐ If you found this project useful, feel free to star the repository!
