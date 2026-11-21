# Service 03 — carts

## What is it?

The carts service is a **Java Spring Boot** microservice that manages each user's shopping cart. It stores cart items in **MongoDB** because cart data is flexible — different users can have different types of items with varying attributes.

## Technical Details

| Property | Value |
|----------|-------|
| Language | Java |
| Framework | Spring Boot 1.4 |
| Database | MongoDB 4.4 (carts-db) |
| Internal Port | 80 |
| Docker Image | `weaveworksdemos/carts:0.4.8` |
| DB Image | `mongo:4.4` |

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/carts/{customerId}` | Get cart for a user |
| DELETE | `/carts/{customerId}` | Empty the cart |
| POST | `/carts/{customerId}/items` | Add item to cart |
| DELETE | `/carts/{customerId}/items/{itemId}` | Remove one item |
| PATCH | `/carts/{customerId}/items` | Update item quantity |

## ⚠️ RHEL 9 Fixes Applied

- `mongo:3.4` → `mongo:4.4` (3.4 is incompatible with RHEL 9 kernel 5.x+)
- `read_only: true` removed (MongoDB writes `/tmp/mongodb-27017.sock` on startup; Spring Boot writes to `/tmp` for Tomcat temp files)

## Hands-On Practice

```bash
# Add item to cart
curl -X POST http://<manager-ip>/carts/test123/items \
  -H 'Content-Type: application/json' \
  -d '{"id":"3395a43e-2d88-40de-b95f-e00e1502085b","quantity":2}'

# View cart
curl http://<manager-ip>/carts/test123/items

# Kill carts-db — cart operations fail, browsing still works
docker service scale sockshop_carts-db=0
docker service scale sockshop_carts-db=1
```
