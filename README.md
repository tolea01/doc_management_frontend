# Document Management Frontend

Frontend application for a role-based document management platform, built with **Next.js 14 (App Router)** and **TypeScript**.

It supports authentication, role-specific dashboards, document flows (entry/exit/internal), and real-time chat updates.

## Table of Contents
- [Overview](#overview)
- [Core Features](#core-features)
- [Architecture](#architecture)
- [Project Structure](#project-structure)
- [Routing & Access Control](#routing--access-control)
- [API & Auth Flow](#api--auth-flow)
- [Environment Variables](#environment-variables)
- [Run Locally](#run-locally)
- [Docker Setup](#docker-setup)
- [CI/CD](#cicd)
- [Troubleshooting](#troubleshooting)

## Overview

This app is the UI layer for managing organizational documents and operational workflows. It uses reusable UI components, typed service layers, and route protection via middleware.

## Core Features

- **Authentication & token refresh** using Axios interceptors and persisted access token.
- **Role-aware navigation and protected dashboards** (Admin, Director, Head of Direction, Secretary).
- **Document domain services** for entry, exit, and internal documents.
- **Person and user management services**.
- **Realtime chat integration** via Socket.IO client.
- **Reusable UI kit**: tables, pagination, modal, badges, form fields, loader, sidebar.

## Architecture

- **Framework:** Next.js 14 (App Router)
- **Language:** TypeScript
- **Data fetching/cache:** @tanstack/react-query
- **HTTP client:** Axios
- **Forms:** react-hook-form
- **Charts:** recharts
- **Date handling:** date-fns, react-datepicker, react-day-picker
- **Styling:** TailwindCSS + SCSS + component CSS
- **Realtime:** socket.io-client

## Project Structure

```text
src/
  app/                     # Pages, dashboards, auth screens, global providers
  api/                     # Axios setup, request/response interceptors
  components/ui/           # Reusable UI components
  components/chat/         # Chat UI
  services/                # Domain service layer (auth, documents, users, chat)
  config/                  # Routes and page URLs
  hooks/                   # Custom hooks (e.g., useAuth)
  middleware.ts            # Route protection logic
  types/                   # Shared TypeScript models
  enums/                   # Domain enums (roles, statuses, token keys)
  utils/                   # Formatting / mapping helpers
```

## Routing & Access Control

- `/` redirects to `/auth/login`.
- Public routes are defined in `src/config/routes.config.ts`.
- Protected routes are role-gated using `UserRole` values.
- Middleware enforces access and redirects unauthorized users.

## API & Auth Flow

- Base URL is read from `API_url`.
- Two Axios clients are used:
  - `axiosClassic`: unauthenticated requests.
  - `axiosWithAuth`: adds bearer token and handles 401 retry.
- On 401, the interceptor attempts token refresh; if refresh fails, stored token is removed.

## Environment Variables

Create `.env.local` from `.env.example`:

```bash
cp .env.example .env.local
```

| Variable | Required | Example | Purpose |
|---|---|---|---|
| `API_url` | Yes | `http://localhost:4200` | Backend API base URL used by Axios clients. |

## Run Locally

Prerequisites:
- Node.js 20+
- npm 10+

Install and start:

```bash
npm ci
npm run dev
```

Build for production:

```bash
npm run build
npm run start
```

## Docker Setup

### Development image
```bash
docker build -f Dockerfile.dev -t doc-management-frontend:dev .
docker run --rm -it -p 3000:3000 -v "$(pwd)":/app -v /app/node_modules doc-management-frontend:dev
```

### Production image
```bash
docker build -t doc-management-frontend:latest .
docker run --rm -p 3000:3000 -e API_url="https://api.example.com" doc-management-frontend:latest
```

### Docker Compose (development)
```bash
docker compose up --build
```

## CI/CD

Workflow file: `.github/workflows/ci-cd.yml`

### CI (push/PR to `main` or `develop`)
1. Checkout source
2. Setup Node 20 + npm cache
3. `npm ci`
4. `npm run lint`
5. `npm run build`
6. Docker build for validation

### CD (push to `main`)
- Logs in to GHCR
- Builds and pushes image tags:
  - `latest`
  - `${{ github.sha }}`

Required GitHub secrets:
- `GHCR_USERNAME`
- `GHCR_TOKEN` (`write:packages`)

## Troubleshooting

- If lint fails with `Failed to load plugin 'prettier'`, add:
  ```bash
  npm i -D eslint-plugin-prettier
  ```
- Ensure backend CORS allows frontend origin and credentials.
- Verify `API_url` is defined in runtime environment for container deployments.
