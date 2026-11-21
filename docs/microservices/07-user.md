# Service 07 — user

## What is it?

The user service is a **Go** microservice that handles all user account management — registration, login, profile data, and delivery addresses. It uses **MongoDB** to store user accounts as flexible JSON documents.

## Technical Details

| Property | Value |
|----------|-------|
| Language | Go |
| Framework | Go standard library |
| Database | MongoDB (weaveworksdemos/user-db:0.4.0) |
| Internal Port | 80 |
| Docker Image | `weaveworksdemos/user:0.4.4` |

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/register` | Create new account |
| GET | `/login` | Authenticate (HTTP Basic Auth) |
| GET | `/customers/{id}` | Get user profile |
| GET | `/customers/{id}/addresses` | Get delivery addresses |
| POST | `/customers/{id}/addresses` | Add address |

## Default Test Accounts

| Username | Password |
|----------|----------|
| `user` | `password` |
| `Eve_Berger` | `eve` |

## Hands-On Practice

```bash
# Register a new user
curl -X POST http://<manager-ip>/register \
  -H 'Content-Type: application/json' \
  -d '{"username":"testuser","password":"testpass","email":"test@example.com"}'

# Login
curl -u user:password http://<manager-ip>/login

# Kill user — login breaks, but catalogue and existing cart still work
docker service scale sockshop_user=0
docker service scale sockshop_user=1
```
