---
description: Docker best practices optimized for AI Agents, ML services, and General Software Engineering. Covers security, multi-stage builds, GPU passthrough, large model handling, Python/C++ layer optimization, and production-ready patterns.
alwaysApply: false
---
# Docker Best Practices Rules for AI Agent

## I. Dockerfile Fundamentals

### 1.1 Base Image Selection (MANDATORY)

**General Rules:**
- **ALWAYS** use official, minimal base images from trusted sources.
- **AVOID** `latest` tag; use specific version pins (e.g., `3.11-slim-bookworm`).
- **VERIFY** base image security using `docker scout` or `trivy`.

**Stack-Specific Base Images:**
- **Go:** `golang:1.21-alpine` (build), `alpine:3.19` (runtime).
- **Python (General/AI):** **PREFER** `python:3.11-slim-bookworm` (Debian-based).
  - *Why:* Alpine (`musl`) breaks Python wheels for NumPy/Pandas/PyTorch, causing massive build times and bugs.
- **AI (GPU Enabled):** Use official NVIDIA runtime, e.g., `nvidia/cuda:12.1.0-runtime-ubuntu22.04`.
- **Node/React:** `node:20-alpine` (build), `nginx:alpine` (runtime).

### 1.2 Multi-Stage Builds (MANDATORY)

**Why:** Separate build dependencies (compilers, headers) from runtime.

**Rules:**
- **ALWAYS** use multi-stage for Compiled Languages (Go, Rust, C++).
- **ALWAYS** use multi-stage for Python if compiling C-extensions or using Poetry/Pipenv.
- **NEVER** include build tools (`gcc`, `g++`, `make`, `cuda-toolkit`) in the final production image; use the runtime variant.

### 1.3 Layer Optimization

**Minimize Layers:**
- **COMBINE** related `RUN` commands using `&&` and `\`.
- **CLEANUP** caches in the *same* `RUN` instruction.

**Layer Ordering (Cache Optimization):**
1. System deps (apt/apk).
2. Dependency files (`requirements.txt`, `pyproject.toml`, `go.mod`).
3. **Install dependencies** (Use BuildKit Cache Mounts).
4. Source code.

## II. Security Best Practices

### 2.1 Non-Root User (MANDATORY)

**ALWAYS** run containers as a non-root user.

**Python/AI Example:**
```dockerfile
# Create user with fixed UID
RUN groupadd -g 1000 appuser && \
    useradd -u 1000 -g appuser -s /bin/sh -m appuser

# Set permissions
CHOWN -R appuser:appuser /app
USER appuser
