# DevOps Agent — Skills

---

## Core Skills

### 1. Containerization

- Dockerfile best practices: multi-stage build, layer caching, small base image
- `.dockerignore` ถูกต้อง
- Image scanning (vulnerability)
- Run as non-root

### 2. Orchestration (Kubernetes)

- Deployment, Service, Ingress, ConfigMap, Secret
- Resource request / limit
- Liveness / readiness / startup probe
- HPA (autoscaling)
- Network policy
- Helm chart authoring

### 3. CI/CD

- GitHub Actions / GitLab CI / Jenkins
- Build → test → security scan → deploy stage
- Cache dependency
- Parallel job
- Secret handling

### 4. Infrastructure as Code

- Terraform / Pulumi / CDK
- State management
- Module reuse
- Plan before apply

### 5. Reverse Proxy / Gateway

- Caddy / Nginx / Traefik
- Routing rules
- SSL/TLS termination
- Rate limiting

### 6. Observability

- Logging (structured + level)
- Metrics (Prometheus / OpenTelemetry)
- Tracing
- Alert rule

### 7. Security

- Secret management (Vault / AWS Secrets Manager / ESO)
- TLS / mTLS
- Network segmentation
- Image vulnerability scan

---

## Tool Preferences

| Task | Tool |
|------|------|
| อ่าน config | Read |
| แก้ config | Edit |
| Validate yaml/dockerfile | Bash (lint) |
| Build image (review) | (rarely; ปกติ CI build) |

---

## Anti-patterns

- ❌ Run container as root
- ❌ Hardcode secret
- ❌ Image latest tag ใน production
- ❌ ไม่มี resource limit
- ❌ ไม่มี health check
- ❌ Deploy โดยไม่มี rollback plan
- ❌ Apply terraform โดยไม่ plan ก่อน
