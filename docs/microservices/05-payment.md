# Service 05 — payment

## What is it?

The payment service is a simple **Go** microservice that simulates payment authorisation. It has **no database** — it is completely stateless. Each authorisation request is independent.

## Technical Details

| Property | Value |
|----------|-------|
| Language | Go |
| Framework | Go standard library |
| Database | None (stateless) |
| Internal Port | 80 |
| Docker Image | `weaveworksdemos/payment:0.4.3` |

## API

```json
// Request
POST /paymentAuth
{ "amount": 19.99 }

// Response — approved
{ "authorised": true, "message": "Payment authorised" }

// Response — declined (amount too high)
{ "authorised": false, "message": "Payment declined: amount exceeds 100.00" }
```

## Key Concepts

- **Stateless = easy to scale**: No database, no session. Run 10 replicas — any one handles any request.
- **Fault isolation**: If payment goes down, only checkout breaks. Browsing, login, and order history all continue.

## Hands-On Practice

```bash
# Scale to 3 replicas
docker service scale sockshop_payment=3

# Kill payment — demonstrate fault isolation
docker service scale sockshop_payment=0
# Browse catalogue — still works ✅
# Try checkout — fails ❌
docker service scale sockshop_payment=1
```
