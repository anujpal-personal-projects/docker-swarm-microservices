# Deployment Guide — Docker Swarm on AWS

## Prerequisites

- Docker Swarm cluster initialised (1 manager + 2 workers recommended)
- All nodes running Docker Engine 20.x or later
- Minimum **6 GB total RAM** across the cluster (Java Spring Boot services use ~512 MB each)
- Port **80** open in AWS Security Group
- RHEL 9, Ubuntu 22.04, or Amazon Linux 2023

## AWS Security Group Ports Required

| Port | Protocol | Purpose |
|------|----------|---------|
| 22 | TCP | SSH access |
| 80 | TCP | Sock Shop web UI |
| 2377 | TCP | Swarm cluster management |
| 7946 | TCP/UDP | Node communication |
| 4789 | UDP | Overlay network (VXLAN) |

## Step-by-Step Deployment

### Step 1 — Clone this repository on the manager node

```bash
git clone https://github.com/anujpal-personal-projects/docker-swarm-microservices.git
cd docker-swarm-microservices
```

### Step 2 — Deploy the stack

```bash
docker stack deploy -c docker-compose.yml sockshop
```

> ✅ This `docker-compose.yml` has all RHEL 9 fixes already applied — no manual patching needed.

### Step 3 — Watch services come up

```bash
watch docker stack services sockshop
```

Expected output after 2-3 minutes:

```
NAME                    REPLICAS   IMAGE
sockshop_carts          1/1        weaveworksdemos/carts:0.4.8
sockshop_carts-db       1/1        mongo:4.4
sockshop_catalogue      1/1        weaveworksdemos/catalogue:0.3.5
sockshop_catalogue-db   1/1        weaveworksdemos/catalogue-db:0.3.0
sockshop_edge-router    1/1        weaveworksdemos/edge-router:0.1.1
sockshop_front-end      1/1        weaveworksdemos/front-end:0.3.12
sockshop_orders         1/1        weaveworksdemos/orders:0.4.7
sockshop_orders-db      1/1        mongo:4.4
sockshop_payment        1/1        weaveworksdemos/payment:0.4.3
sockshop_queue-master   1/1        weaveworksdemos/queue-master:0.3.1
sockshop_rabbitmq       1/1        rabbitmq:3.6.8
sockshop_shipping       1/1        weaveworksdemos/shipping:0.4.8
sockshop_user           1/1        weaveworksdemos/user:0.4.4
sockshop_user-db        1/1        weaveworksdemos/user-db:0.4.0
sockshop_user-sim       1/1        weaveworksdemos/load-test:0.1.1
```

### Step 4 — Access the application

```
http://<any-node-public-ip>:80
```

- Default test user: **user / password**
- Add socks to cart → Place an order

### Step 5 — Remove the stack

```bash
docker stack rm sockshop
```

## Useful Commands During Training

```bash
# See all 15 services and their status
docker stack services sockshop

# See which node each container is running on
docker stack ps sockshop

# Scale a service
docker service scale sockshop_front-end=3

# View logs of a service
docker service logs sockshop_orders --tail 30

# Force restart a service
docker service update --force sockshop_orders
```
