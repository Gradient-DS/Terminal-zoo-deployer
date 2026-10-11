# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Kubernetes deployment management platform with two main components:
- **Nuxt 4 Frontend** - Vue 3 application with PrimeVue UI components and Pinia state management
- **Go Backend** - HTTP API server that manages Kubernetes deployments via client-go

The frontend proxies requests through Nuxt server routes (`server/api/`) to the Go backend, which communicates with Kubernetes clusters.

## Common Commands

### Frontend (Nuxt)
```bash
npm install          # Install dependencies
npm run dev          # Start dev server on localhost:3000
npm run build        # Production build
npm run preview      # Preview production build
npm run lint         # Run linter
npm run test         # Run tests
```

### Go Backend
```bash
cd go
make build           # Build binary (linux/amd64)
make run             # Build and run locally
make docker-build    # Build Docker image
```

### Docker Compose
```bash
make build           # Build with docker-compose
make run             # Run containers detached
make stop            # Stop containers
make logs            # View container logs
make clean           # Remove containers, volumes, node_modules, .nuxt, .output
```

## Architecture

### Frontend Structure
- `app/pages/` - Nuxt file-based routing (apps, auth, dashboard, deploy, community)
- `app/stores/` - Pinia stores for state (deploy, user, deployments, appCatalog, criteria)
- `app/composables/API/` - API client composables (useClusters, useDeployments, useConfigSecrets)
- `app/layouts/` - Page layouts (default, dashboard, auth, landing)
- `server/api/` - Nuxt server routes that proxy to Go backend

### Backend Structure (go/)
- `cmd/server/main.go` - Entry point with signal handling and HTTP server setup
- `internal/server/server.go` - HTTP handler with all Kubernetes API endpoints
- `pkg/kube/client/` - Kubernetes client creation (in-cluster and out-of-cluster)
- `pkg/log/` - Logging abstraction

### API Endpoints (Go backend)
The Go server exposes REST endpoints for:
- `/deployments` - CRUD for Kubernetes deployments (creates Deployment + Service + Ingress)
- `/pods/{namespace}` - List pods and logs
- `/clusters` - Multi-cluster management (onboard external clusters via SA token)
- `/secrets`, `/configmap` - Kubernetes secrets and configmaps

Multi-cluster support uses the `cluster-name` header to switch between registered Kubernetes clients.

## Key Patterns

### Deployment Flow
1. Frontend submits via `/api/deployments` server route
2. Nuxt server route validates and forwards to Go backend (`NUXT_EXTERNAL_API_BASE`)
3. Go backend creates: Namespace (if needed) -> TLS Secret -> Deployment -> Service -> Traefik Middleware -> Ingress

### State Management
- `useDeployStore` - Draft deployment specs and deployment list
- `useUserStore` - Authentication state
- `useDeploymentsStore` - Kubernetes deployments from API

### Configuration
- `nuxt.config.ts` - Contains `runtimeConfig.externalApiBase` for Go backend URL
- `.env` / `.env.example` - Environment variables for HTTP timeouts and listen address
- Go uses `envconfig` with `AICO_` prefix for configuration
