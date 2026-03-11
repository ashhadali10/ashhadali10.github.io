```markdown
---
title: "Securing Docker Containers — Practical Hands-On Guide"
date: 2025-11-23
tags: [docker, security, devsecops, containers, hardening]
---

Securing Docker Containers — Practical Hands-On Guide

**Learn production-grade techniques to secure Docker containers:** non-root users, Docker Content Trust, Docker Scout scanning, capability limiting, read-only filesystems, and a full hardened compose example.

---

Hook — A Hard Truth Every Senior Engineer Knows

If a container in your fleet is running as root, it's not a "sandbox" — it's a live bridge to the host. In real incidents, attackers have used unscanned images, writable containers, or excessive Linux capabilities to escalate from an app process to host compromise.

This lab-based guide converts hands-on lab experience into a mentor-style article you can use in production. No fluff — just step-by-step explanation, why each step matters, how it works, and copy-paste commands you can run locally or in a lab environment.

---

Table of Contents

1. [Objectives & Prerequisites](#objectives--prerequisites)
2. [Lab Environment Overview](#lab-environment-overview)
3. [Task 1 — Run Containers as Non-Root Users](#task-1--run-containers-as-non-root-users)
4. [Task 2 — Docker Content Trust](#task-2--docker-content-trust)
5. [Task 3 — Security Scanning with Docker Scout](#task-3--security-scanning-with-docker-scout)
6. [Task 4 — Limit Container Privileges](#task-4--limit-container-privileges)
7. [Task 5 — Read-Only Filesystems](#task-5--read-only-filesystems)
8. [Task 6 — Comprehensive Secure Docker Compose](#task-6--comprehensive-secure-docker-compose)
9. [Troubleshooting & Common Mistakes](#troubleshooting--common-mistakes)
10. [Lab Cleanup](#lab-cleanup)
11. [Key Takeaways & Next Steps](#key-takeaways--next-steps)

---

 Objectives & Prerequisites

### What You'll Achieve

By the end of this article you will be able to:

- Implement Docker security best practices for containerized apps
- Run containers as non-root users using `USER` in Dockerfile
- Set up Docker Content Trust and sign images
- Use Docker Scout to scan images for CVEs and compare bases
- Limit container privileges by dropping capabilities and applying security options
- Make containers read-only and use tmpfs for writable runtime storage
- Combine all controls in a production-grade docker-compose deployment

### Prerequisites

- Basic Docker knowledge (images, containers, Dockerfile)
- Linux CLI familiarity
- Understanding of user permissions and filesystems in Linux

### Lab Environment

The lab assumes a preconfigured Linux VM (Ubuntu 20.04) with Docker and Docker Compose available. You can run everything on a local machine, VM, or cloud lab.

---

## Task 1 — Run Containers as Non-Root Users

### What It Is

Run your application inside the container as a non-privileged user instead of root.

### Why It Matters

If the container process is root and an attacker escapes the container, they can often perform host-level actions. Running as non-root reduces blast radius and follows the principle of least privilege.

**Analogy:** Running a container as root is like giving a contractor full keys to your house vs. restricting them to a room while work is done.

### How It Works

1. Create a system user and group inside the image (`useradd`, `groupadd` or `adduser` on Alpine)
2. Make sure application files/directories are owned by that user
3. Use the `USER` directive in the Dockerfile so the runtime PID runs under non-root UID/GID

### Implementation

**Create a secure app directory and minimal Flask app:**

```bash
mkdir secure-app
cd secure-app

cat > app.py <<'EOF'
from flask import Flask
import os

app = Flask(__name__)

@app.route('/')
def hello():
    return f"Hello from secure container! Running as user: {os.getuid()}"

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=8080)
EOF

cat > requirements.txt <<'EOF'
Flask==2.3.3
EOF
```

**Create a secure Dockerfile:**

```dockerfile
FROM python:3.9-slim

# Create a non-root user
RUN groupadd -r appuser && useradd -r -g appuser appuser

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY app.py .
RUN chown -R appuser:appuser /app

USER appuser

EXPOSE 8080
CMD ["python", "app.py"]
```

**Build and run:**

```bash
docker build -t secure-app:v1 .
docker run -d -p 8080:8080 --name secure-app-container secure-app:v1

# Verify non-root
docker exec secure-app-container ps aux
curl http://localhost:8080
docker exec secure-app-container id   # should show uid != 0
```

### Common Mistakes

| Mistake | Fix |
|---------|-----|
| Forgetting `chown` → app files owned by root cause permission errors | `chown -R appuser:appuser /app` before `USER` |
| Using `USER` but still exposing privileged ports (<1024) without CAP | Use non-root ports (e.g., 8080) or add minimal capability `NET_BIND_SERVICE` only |

---

## Task 2 — Implement Docker Content Trust

### What It Is

Docker Content Trust (DCT) uses cryptographic signatures (Notary) to ensure image integrity and authenticity.

### Why It Matters

Prevents pulling tampered or malicious images — essential for supply-chain security and compliance.

**Analogy:** DCT is a tamper-evident seal on a package: if it's broken, you don't accept the package.

### How It Works

- `DOCKER_CONTENT_TRUST=1` forces Docker to require signatures on pulls/pushes
- You generate signing keys, assign signer roles to repositories, sign images, and push trust metadata alongside the image

### Implementation

**Enable trust:**

```bash
export DOCKER_CONTENT_TRUST=1
echo $DOCKER_CONTENT_TRUST   # should output 1
```

**Initialize keys and add signer:**

```bash
mkdir -p ~/.docker/trust
docker trust key generate mykey
docker trust signer add --key mykey.pub mykey secure-app
```

**Start a local registry and push signed image:**

```bash
docker run -d -p 5000:5000 --name registry registry:2

docker tag secure-app:v1 localhost:5000/secure-app:signed
docker push localhost:5000/secure-app:signed

docker trust inspect localhost:5000/secure-app:signed
```

**Test enforcement (unsigned pull should fail with DCT on):**

```bash
# This should fail when DOCKER_CONTENT_TRUST=1
docker pull hello-world

# To pull an unsigned image temporarily:
export DOCKER_CONTENT_TRUST=0
docker pull hello-world
export DOCKER_CONTENT_TRUST=1
```

### Common Mistakes

| Mistake | Fix |
|---------|-----|
| Storing private keys in repo — never commit `~/.docker/trust` | Add to `.gitignore`, use secret management |
| Using root key online — keep root key offline | Use repository signing keys for daily ops |
| Not enabling DCT in CI | Set `DOCKER_CONTENT_TRUST=1` in pipeline env |

---

## Task 3 — Security Scanning with Docker Scout

### What It Is

Docker Scout scans images against vulnerability databases and provides CVE reports and recommendations.

### Why It Matters

Detects known vulnerabilities before images are deployed; informs decisions about base images and fixes.

### How It Works

- `docker scout cves <image>` queries vulnerability DBs and returns findings
- You can export results (SARIF) for integration with tooling or CI

### Implementation

**Check Scout availability and install if necessary:**

```bash
docker scout version

# If not available:
curl -sSfL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sh -s --
```

**Scan images:**

```bash
docker scout cves secure-app:v1
docker scout cves --format sarif --output secure-app-scan.json secure-app:v1

# Baseline scan
docker scout cves python:3.9-slim
```

**Compare and get recommendations:**

```bash
docker scout compare secure-app:v1 --to python:3.9-alpine
docker scout recommendations secure-app:v1
```

### Image Hardening Example

Create `Dockerfile.secure` with smaller, updated base:

```dockerfile
FROM python:3.9-alpine

RUN apk update && apk upgrade && rm -rf /var/cache/apk/*

RUN addgroup -g 1001 -S appuser && adduser -u 1001 -S appuser -G appuser

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

COPY app.py .
RUN chown -R appuser:appuser /app

USER appuser

EXPOSE 8080

HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \
  CMD curl -f http://localhost:8080/ || exit 1

CMD ["python", "app.py"]
```

**Build and rescan:**

```bash
docker build -f Dockerfile.secure -t secure-app:v2 .
docker scout cves secure-app:v2
```

### Common Mistakes

| Mistake | Fix |
|---------|-----|
| Only scanning top image but not base image | Always scan base images and layers |
| Ignoring recommendations | Some findings may be low severity, but cumulative risk matters |

---

## Task 4 — Limit Container Privileges

### What It Is

Reduce Linux capabilities and enforce runtime security options so container processes can't perform risky host-level operations.

### Why It Matters

Default container privileges include many Linux capabilities. Dropping unnecessary ones reduces attack surface and prevents privilege escalation.

**Analogy:** Capabilities are like special passes at a building — only give the passes necessary for the job.

### How It Works

| Option | Effect |
|--------|--------|
| `--cap-drop=ALL` | Removes all capabilities |
| `--cap-add=NET_BIND_SERVICE` | Selectively adds required capability (bind low ports) |
| `--security-opt=no-new-privileges:true` | Blocks privilege escalation via setuid binaries |
| AppArmor/Seccomp profiles | Restrict syscalls and device access |

### Implementation

**Check default capabilities:**

```bash
docker run --rm alpine:latest sh -c "apk add --no-cache libcap && capsh --print"
```

**Run with dropped caps and only NET_BIND_SERVICE:**

```bash
docker run -d --name limited-container \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  -p 8081:8080 \
  secure-app:v2
```

**Verify inside container:**

```bash
docker exec limited-container sh -c "apk add --no-cache libcap && capsh --print"
```

**Use additional security opts:**

```bash
docker run -d --name secure-container \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --security-opt=no-new-privileges:true \
  --security-opt=apparmor:docker-default \
  -p 8082:8080 \
  secure-app:v2
```

**Inspect security options:**

```bash
docker inspect secure-container | grep -A 10 "SecurityOpt"
```

**Test forbidden privileged operation (should fail):**

```bash
# Should fail
docker exec limited-container mount -t tmpfs tmpfs /tmp/test

# Contrast: privileged container succeeds
docker run --rm --privileged alpine:latest mount -t tmpfs tmpfs /tmp/test
```

### Common Mistakes

| Mistake | Fix |
|---------|-----|
| Granting `--privileged` unnecessarily | It gives uncontrolled host access — avoid it |
| Adding too many capabilities | Only add what app strictly needs. Use `strace` to identify necessary calls |

---

## Task 5 — Use Read-Only File Systems

### What It Is

Run containers with root filesystem mounted read-only and provide ephemeral writable areas (`/tmp`) with tmpfs.

### Why It Matters

Prevents code or malware inside container from writing to the image, making persistence and tampering harder.

**Analogy:** Making the container filesystem read-only is like locking the doors of a shop after hours — intruders can't rearrange stock permanently.

### How It Works

- `--read-only` mounts container root as `ro`
- Use `--tmpfs /tmp:rw,noexec,nosuid` for temporary writable directories
- App must avoid writing to `/app` and use permitted locations like `/tmp` or mounted volumes

### Implementation

**Create `app_readonly.py` that writes only to `/tmp`:**

```python
from flask import Flask, request, jsonify
import os
import tempfile

app = Flask(__name__)

@app.route('/')
def hello():
    return f"Hello from read-only container! Running as user: {os.getuid()}"

@app.route('/write-test', methods=['POST'])
def write_test():
    try:
        with tempfile.NamedTemporaryFile(mode='w', delete=False, dir='/tmp') as f:
            f.write("Temporary data")
            temp_file = f.name
        return jsonify({"status": "success", "message": f"Successfully wrote to {temp_file}"})
    except Exception as e:
        return jsonify({"status": "error", "message": str(e)})

@app.route('/illegal-write', methods=['POST'])
def illegal_write():
    try:
        with open('/app/illegal_file.txt', 'w') as f:
            f.write("This should not work")
        return jsonify({"status": "success", "message": "File written"})
    except Exception as e:
        return jsonify({"status": "error", "message": str(e)})
```

**Dockerfile for read-only image (`Dockerfile.readonly`):**

```dockerfile
FROM python:3.9-alpine

RUN apk update && apk upgrade && \
    addgroup -g 1001 -S appuser && \
    adduser -u 1001 -S appuser -G appuser

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

COPY app_readonly.py .
RUN mkdir -p /tmp/app-data && \
    chown -R appuser:appuser /app /tmp/app-data

USER appuser

EXPOSE 8080
CMD ["python", "app_readonly.py"]
```

**Build and run read-only container:**

```bash
docker build -f Dockerfile.readonly -t readonly-app:v1 .

docker run -d --name readonly-container \
  --read-only \
  --tmpfs /tmp:rw,noexec,nosuid,size=100m \
  --cap-drop=ALL \
  --cap-add=NET_BIND_SERVICE \
  --security-opt=no-new-privileges:true \
  -p 8083:8080 \
  readonly-app:v1
```

**Test:**

```bash
# Should succeed
curl http://localhost:8083/
curl -X POST http://localhost:8083/write-test

# Should fail
curl -X POST http://localhost:8083/illegal-write

# Filesystem tests
docker exec readonly-container touch /app/test-file.txt      # fails
docker exec readonly-container touch /tmp/test-file.txt      # succeeds
docker exec readonly-container df -h
docker exec readonly-container mount | grep -E "(tmpfs|ro)"
```

### Common Mistakes

| Mistake | Fix |
|---------|-----|
| App writes to `/var/tmp` or `/app` | Ensure app uses exclusive temp directories |
| Forgetting to `chown` directories mounted as writable | Non-root user must own writable dirs |

---

## Task 6 — Comprehensive Secure Docker Compose

### What It Is

Bundle everything — non-root, read-only root, tmpfs, dropped caps, AppArmor, healthchecks, and logging — into a production-grade docker-compose service.

### Why It Matters

A single, reproducible config enforces consistent runtime security and is ready for CI/CD and ops automation.

### Production-Grade Compose File

**`docker-compose.secure.yml`:**

```yaml
version: '3.8'

services:
  secure-web-app:
    build:
      context: .
      dockerfile: Dockerfile.readonly
    ports:
      - "8084:8080"
    read_only: true
    tmpfs:
      - /tmp:rw,noexec,nosuid,size=100m
    cap_drop:
      - ALL
    cap_add:
      - NET_BIND_SERVICE
    security_opt:
      - no-new-privileges:true
      - apparmor:docker-default
    user: "1001:1001"
    environment:
      - PYTHONUNBUFFERED=1
    healthcheck:
      test: ["CMD", "wget", "--no-verbose", "--tries=1", "--spider", "http://localhost:8080/"]
      interval: 30s
      timeout: 10s
      retries: 3
      start_period: 40s
    restart: unless-stopped
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

**Deploy & test:**

```bash
docker compose -f docker-compose.secure.yml up -d
docker compose -f docker-compose.secure.yml ps

curl http://localhost:8084/
curl -X POST http://localhost:8084/write-test
curl -X POST http://localhost:8084/illegal-write

# Verify security options
docker inspect $(docker compose -f docker-compose.secure.yml ps -q) | \
  grep -A 20 "SecurityOpt"
```

### Security Audit Checklist (Post-Deployment)

| Check | Command |
|-------|---------|
| Final CVE scan | `docker scout cves readonly-app:v1` |
| Running processes | `docker exec <container> ps aux` |
| File ownership & perms | `docker exec <container> ls -la /` |
| Privilege escalation should fail | `docker exec <container> su -` |

---

## Troubleshooting & Common Mistakes

### Issue 1 — Docker Content Trust Errors

**Solution:**

```bash
rm -rf ~/.docker/trust
docker trust key generate mykey
```

### Issue 2 — Permission Denied in Read-Only Containers

**Solution:** Ensure proper tmpfs mounts and permissive writable dirs:

```bash
--tmpfs /tmp:rw,noexec,nosuid,size=100m
--tmpfs /var/tmp:rw,noexec,nosuid,size=50m
```

### Issue 3 — Security Scanning Tool Missing

**Solution:** Install Docker Scout CLI or use Trivy as alternative:

```bash
# Docker Scout
curl -sSfL https://raw.githubusercontent.com/docker/scout-cli/main/install.sh | sh -s --

# Or Trivy
docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
  aquasec/trivy image secure-app:v1
```

### Issue 4 — Application Needs Additional Capabilities

**Solution:** Trace capability use and add only required capabilities:

```bash
strace -e trace=capget,capset docker run --rm your-image

# Then add only what's needed:
--cap-add=NET_ADMIN  # only if justified
```

---

## Lab Cleanup

Run these commands after completing tests:

```bash
docker compose -f docker-compose.secure.yml down

docker stop $(docker ps -aq) 2>/dev/null || true
docker rm $(docker ps -aq) 2>/dev/null || true

docker rmi secure-app:v1 secure-app:v2 readonly-app:v1 2>/dev/null || true

docker stop registry && docker rm registry 2>/dev/null || true

rm -rf secure-app/
rm -f *.json *.pub mykey

unset DOCKER_CONTENT_TRUST
```

---

## Key Takeaways & Next Steps

### What You Should Remember

| Control | Impact |
|---------|--------|
| **Run containers as non-root** | Biggest single improvement in risk reduction |
| **Sign images (Docker Content Trust)** | Prevents supply-chain tampering |
| **Scan images (Docker Scout / Trivy)** | Catch CVEs before runtime |
| **Drop capabilities & use security options** | Reduce host-level risks |
| **Run containers read-only with tmpfs** | Eliminate persistence inside images |
| **Combine controls** | Defense-in-depth is stronger than individual hardening |

### Actions to Take Now

1. **Audit your images** with `docker scout cves` or `trivy`
2. **Search for containers running as root** in your environments and change Dockerfiles to use `USER`
3. **Enable signing in CI** (set `DOCKER_CONTENT_TRUST=1`) for image promotion
4. **Add runtime controls** in your orchestration (Pod Security Standards in Kubernetes)
5. **Run kube-bench** and other CIS checks on clusters; include RBAC hardening and network policies
6. **Automate image scans in CI** and block promotions on high/critical CVEs

### Final Words

Security is never "done." These lab steps form a practical, repeatable baseline for production-grade container security. Start by removing root where you can, sign the images you promote, scan continuously, and enforce minimal runtime privileges.

Treat containers like short-lived systems: **immutable, monitored, and replaceable** — and you'll drastically reduce the chance of a small vulnerability turning into a full-blown breach.

---

*Have questions or want to share your hardening results? [Reach out on Twitter](https://twitter.com/Kashhad10) or [connect on LinkedIn](https://linkedin.com/in/ashhadali10).*
```
