# Troubleshooting Guide

## The 3-Step Method

```bash
# Step 1 — Find the broken service
docker stack services sockshop
# Look for 0/1 in REPLICAS column

# Step 2 — See the error code and node
docker stack ps sockshop --no-trunc | grep -v "Running\|Complete"

# Step 3 — Read the actual error message
docker service logs sockshop_<service-name> --tail 30
```

---

## Issue 1 — MongoDB fails: "Read-only file system"

**Services affected**: `carts-db`, `orders-db`, `user-db`

**Log error**:
```
Failed to unlink socket file /tmp/mongodb-27017.sock: Read-only file system
Fatal assertion: aborting after fassert() failure
```

**Root cause**: Original compose file sets `read_only: true`. MongoDB writes a Unix socket to `/tmp` on startup — blocked by read-only filesystem.

**Fix** (already applied in this repo):
```bash
python3 -c "
import yaml
with open('docker-compose.yml', 'r') as f:
    data = yaml.safe_load(f)
for svc in data['services'].values():
    svc.pop('read_only', None)
with open('docker-compose.yml', 'w') as f:
    yaml.dump(data, f, default_flow_style=False)
"
```

---

## Issue 2 — MongoDB fails on RHEL 9 (mongo:3.4)

**Log error**: `task: non-zero exit (14)`

**Root cause**: `mongo:3.4` is built for Ubuntu 16.04 and is incompatible with Linux kernel 5.x+ used by RHEL 9.

**Fix** (already applied in this repo):
```bash
sed -i 's/mongo:3.4/mongo:4.4/g' docker-compose.yml
```

---

## Issue 3 — Java services fail: "Unable to create tempDir"

**Services affected**: `orders`, `carts`, `shipping`, `queue-master`

**Log error**:
```
Unable to create tempDir. java.io.tmpdir is set to /tmp
Caused by: java.io.IOException: Read-only file system
```

**Root cause**: Spring Boot's embedded Tomcat writes temporary files to `/tmp` during startup. `read_only: true` blocks this.

**Fix**: Same as Issue 1 — remove `read_only: true` from all services.

---

## Issue 4 — Wrong path after git clone

**Error**:
```
bash: cd: microservices-demo/deploy/docker-swarm: No such file or directory
```

**Root cause**: The upstream Weaveworks repository changed its folder structure. The Docker Swarm compose file moved to `deploy/docker-compose/`.

**Fix**: Use this repository instead — the fixed `docker-compose.yml` is at the root:
```bash
git clone https://github.com/anujpal-personal-projects/docker-swarm-microservices.git
cd docker-swarm-microservices
docker stack deploy -c docker-compose.yml sockshop
```

---

## Quick Diagnostic Commands

```bash
# All tasks with errors
docker stack ps sockshop --no-trunc | grep -v "Running\|Complete"

# Service logs
docker service logs sockshop_orders --tail 30

# Node health
docker node ls

# Resource usage
docker stats --no-stream

# Force restart
docker service update --force sockshop_orders
```
