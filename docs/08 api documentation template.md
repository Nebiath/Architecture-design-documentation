# [Nombre del Servicio] - API Documentation

> **Version:** 1.0.0  
> **Fecha:** YYYY-MM-DD  
> **API Type:** REST / GraphQL / gRPC  
> **Base URL:** https://api.company.com/v1  
> **Status:** Active | Beta | Deprecated

---

## 1. API Overview

### 1.1 Description

[Brief description of what this API does and its purpose]

Example: "The User Management API provides endpoints for creating, updating, and retrieving user profiles. It supports authentication, role management, and user preferences."

### 1.2 Key Features

- ✅ Feature 1 (e.g., User authentication)
- ✅ Feature 2 (e.g., Profile management)
- ✅ Feature 3 (e.g., Role-based access control)

### 1.3 API Characteristics

| Attribute | Value |
|-----------|-------|
| **Protocol** | HTTPS |
| **Format** | JSON |
| **Authentication** | OAuth 2.0, API Keys |
| **Rate Limiting** | 1000 requests/minute |
| **Versioning** | URI-based (/v1, /v2) |
| **SLA** | 99.9% uptime |

---

## 2. Getting Started

### 2.1 Prerequisites

- Valid API credentials (see Authentication section)
- HTTPS-capable client
- JSON parser

### 2.2 Base URLs

| Environment | URL | Purpose |
|-------------|-----|---------|
| Production | https://api.company.com | Live data |
| Staging | https://api-staging.company.com | Testing |
| Sandbox | https://api-sandbox.company.com | Development |

### 2.3 Quick Start Example

```bash
# Get an access token
curl -X POST https://api.company.com/v1/auth/token \
  -H "Content-Type: application/json" \
  -d '{
    "client_id": "your_client_id",
    "client_secret": "your_client_secret",
    "grant_type": "client_credentials"
  }'

# Make an authenticated request
curl -X GET https://api.company.com/v1/users/me \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

---

## 3. Authentication

### 3.1 Authentication Methods

#### OAuth 2.0 (Recommended)

**Grant Types Supported:**
- Client Credentials (service-to-service)
- Authorization Code (user authorization)
- Refresh Token (token renewal)

**Token Endpoint:**
```
POST /v1/auth/token
```

**Request:**
```json
{
  "grant_type": "client_credentials",
  "client_id": "your_client_id",
  "client_secret": "your_client_secret",
  "scope": "read:users write:users"
}
```

**Response:**
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 3600,
  "scope": "read:users write:users"
}
```

**Token Expiration:** 1 hour (3600 seconds)

#### API Keys

**For simpler integrations:**

```bash
curl -X GET https://api.company.com/v1/users \
  -H "X-API-Key: your_api_key"
```

**Obtaining API Key:**
1. Log into developer portal: https://developers.company.com
2. Navigate to "API Keys"
3. Click "Generate New Key"
4. Copy and store securely

**Security:** Treat API keys like passwords - never commit to git

### 3.2 Scopes

| Scope | Description | Permissions |
|-------|-------------|-------------|
| `read:users` | Read user data | GET /users, GET /users/{id} |
| `write:users` | Modify users | POST /users, PUT /users/{id} |
| `delete:users` | Delete users | DELETE /users/{id} |
| `admin:all` | Full access | All endpoints |

---

## 4. Rate Limiting

### 4.1 Rate Limit Rules

| Plan | Rate Limit | Burst | Period |
|------|------------|-------|--------|
| Free | 100 req/min | 120 | 1 minute |
| Basic | 1,000 req/min | 1,200 | 1 minute |
| Pro | 10,000 req/min | 12,000 | 1 minute |
| Enterprise | Custom | Custom | Negotiated |

### 4.2 Rate Limit Headers

**Every response includes:**

```http
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1679068800
```

### 4.3 Rate Limit Exceeded Response

**HTTP Status:** 429 Too Many Requests

```json
{
  "error": {
    "code": "rate_limit_exceeded",
    "message": "Rate limit exceeded. Retry after 60 seconds.",
    "retry_after": 60
  }
}
```

**Best Practice:** Implement exponential backoff

---

## 5. API Endpoints

### 5.1 Users

#### GET /v1/users

**Description:** List all users (paginated)

**Authentication:** Required (Bearer token or API key)

**Scopes:** `read:users`

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| page | integer | No | 1 | Page number |
| per_page | integer | No | 20 | Items per page (max: 100) |
| sort | string | No | created_at | Sort field (created_at, name, email) |
| order | string | No | desc | Sort order (asc, desc) |
| filter | string | No | - | Filter by status (active, inactive) |

**Example Request:**

```bash
curl -X GET "https://api.company.com/v1/users?page=1&per_page=20&sort=name&order=asc" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

**Example Response:**

```json
{
  "data": [
    {
      "id": "usr_1234567890",
      "email": "user@example.com",
      "name": "John Doe",
      "status": "active",
      "role": "user",
      "created_at": "2024-01-15T10:30:00Z",
      "updated_at": "2024-03-15T14:20:00Z"
    }
  ],
  "meta": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 5,
    "total_count": 100
  },
  "links": {
    "self": "https://api.company.com/v1/users?page=1",
    "next": "https://api.company.com/v1/users?page=2",
    "last": "https://api.company.com/v1/users?page=5"
  }
}
```

**HTTP Status Codes:**

| Code | Description |
|------|-------------|
| 200 | Success |
| 401 | Unauthorized (missing or invalid token) |
| 403 | Forbidden (insufficient permissions) |
| 429 | Rate limit exceeded |

---

#### GET /v1/users/{id}

**Description:** Retrieve a specific user by ID

**Authentication:** Required

**Scopes:** `read:users`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | User ID (format: usr_XXXXXXXXXX) |

**Example Request:**

```bash
curl -X GET "https://api.company.com/v1/users/usr_1234567890" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

**Example Response:**

```json
{
  "id": "usr_1234567890",
  "email": "user@example.com",
  "name": "John Doe",
  "phone": "+1-555-0100",
  "status": "active",
  "role": "user",
  "preferences": {
    "language": "en",
    "timezone": "America/New_York",
    "notifications": {
      "email": true,
      "sms": false
    }
  },
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-03-15T14:20:00Z",
  "metadata": {
    "custom_field_1": "value1"
  }
}
```

**HTTP Status Codes:**

| Code | Description |
|------|-------------|
| 200 | Success |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | User not found |

---

#### POST /v1/users

**Description:** Create a new user

**Authentication:** Required

**Scopes:** `write:users`

**Request Body:**

```json
{
  "email": "newuser@example.com",
  "name": "Jane Smith",
  "phone": "+1-555-0101",
  "role": "user",
  "preferences": {
    "language": "en",
    "timezone": "UTC"
  }
}
```

**Request Schema:**

| Field | Type | Required | Validation | Description |
|-------|------|----------|------------|-------------|
| email | string | Yes | Valid email format | User's email |
| name | string | Yes | 2-100 characters | User's full name |
| phone | string | No | E.164 format | Phone number |
| role | string | No | Enum: user, admin | User role (default: user) |
| preferences | object | No | - | User preferences |

**Example Request:**

```bash
curl -X POST "https://api.company.com/v1/users" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "newuser@example.com",
    "name": "Jane Smith",
    "role": "user"
  }'
```

**Example Response:**

```json
{
  "id": "usr_0987654321",
  "email": "newuser@example.com",
  "name": "Jane Smith",
  "status": "active",
  "role": "user",
  "created_at": "2024-03-15T16:00:00Z",
  "updated_at": "2024-03-15T16:00:00Z"
}
```

**HTTP Status Codes:**

| Code | Description |
|------|-------------|
| 201 | Created successfully |
| 400 | Bad request (validation error) |
| 401 | Unauthorized |
| 403 | Forbidden |
| 409 | Conflict (email already exists) |
| 422 | Unprocessable entity |

---

#### PUT /v1/users/{id}

**Description:** Update an existing user

**Authentication:** Required

**Scopes:** `write:users`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | User ID |

**Request Body:** (All fields optional)

```json
{
  "name": "John Updated Doe",
  "phone": "+1-555-0199",
  "preferences": {
    "language": "es"
  }
}
```

**Example Response:**

```json
{
  "id": "usr_1234567890",
  "email": "user@example.com",
  "name": "John Updated Doe",
  "phone": "+1-555-0199",
  "status": "active",
  "role": "user",
  "created_at": "2024-01-15T10:30:00Z",
  "updated_at": "2024-03-15T16:30:00Z"
}
```

**HTTP Status Codes:**

| Code | Description |
|------|-------------|
| 200 | Updated successfully |
| 400 | Bad request |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | User not found |
| 422 | Unprocessable entity |

---

#### DELETE /v1/users/{id}

**Description:** Delete a user (soft delete)

**Authentication:** Required

**Scopes:** `delete:users`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | string | Yes | User ID |

**Example Request:**

```bash
curl -X DELETE "https://api.company.com/v1/users/usr_1234567890" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN"
```

**Example Response:**

```json
{
  "id": "usr_1234567890",
  "deleted": true,
  "deleted_at": "2024-03-15T17:00:00Z"
}
```

**HTTP Status Codes:**

| Code | Description |
|------|-------------|
| 200 | Deleted successfully |
| 401 | Unauthorized |
| 403 | Forbidden |
| 404 | User not found |

---

## 6. Common Response Patterns

### 6.1 Success Response

```json
{
  "data": { ... },
  "meta": { ... },
  "links": { ... }
}
```

### 6.2 Error Response

**Standard Error Format:**

```json
{
  "error": {
    "code": "error_code",
    "message": "Human-readable error message",
    "details": [
      {
        "field": "email",
        "issue": "Email format is invalid"
      }
    ],
    "request_id": "req_abc123def456"
  }
}
```

### 6.3 Validation Error (422)

```json
{
  "error": {
    "code": "validation_error",
    "message": "Request validation failed",
    "details": [
      {
        "field": "email",
        "issue": "must be a valid email address"
      },
      {
        "field": "name",
        "issue": "must be at least 2 characters"
      }
    ]
  }
}
```

---

## 7. HTTP Status Codes

| Code | Name | Description |
|------|------|-------------|
| 200 | OK | Request successful |
| 201 | Created | Resource created |
| 204 | No Content | Request successful, no content returned |
| 400 | Bad Request | Malformed request syntax |
| 401 | Unauthorized | Missing or invalid authentication |
| 403 | Forbidden | Authenticated but insufficient permissions |
| 404 | Not Found | Resource doesn't exist |
| 409 | Conflict | Request conflicts with current state |
| 422 | Unprocessable Entity | Validation error |
| 429 | Too Many Requests | Rate limit exceeded |
| 500 | Internal Server Error | Server error |
| 503 | Service Unavailable | Temporary server issue |

---

## 8. Pagination

**Query Parameters:**

| Parameter | Type | Default | Max | Description |
|-----------|------|---------|-----|-------------|
| page | integer | 1 | - | Page number |
| per_page | integer | 20 | 100 | Items per page |

**Response Meta:**

```json
{
  "meta": {
    "current_page": 1,
    "per_page": 20,
    "total_pages": 5,
    "total_count": 100,
    "has_next": true,
    "has_prev": false
  },
  "links": {
    "self": "https://api.company.com/v1/users?page=1",
    "first": "https://api.company.com/v1/users?page=1",
    "prev": null,
    "next": "https://api.company.com/v1/users?page=2",
    "last": "https://api.company.com/v1/users?page=5"
  }
}
```

---

## 9. Filtering & Sorting

### 9.1 Filtering

**Syntax:** `?filter[field]=value`

**Example:**
```
GET /v1/users?filter[status]=active&filter[role]=admin
```

**Supported Operators:**

| Operator | Syntax | Example |
|----------|--------|---------|
| Equals | `filter[field]=value` | `filter[status]=active` |
| Not equals | `filter[field][ne]=value` | `filter[status][ne]=inactive` |
| Greater than | `filter[field][gt]=value` | `filter[created_at][gt]=2024-01-01` |
| Less than | `filter[field][lt]=value` | `filter[age][lt]=30` |
| In list | `filter[field][in]=val1,val2` | `filter[role][in]=admin,moderator` |

### 9.2 Sorting

**Syntax:** `?sort=field&order=asc|desc`

**Example:**
```
GET /v1/users?sort=created_at&order=desc
```

**Multiple Fields:**
```
GET /v1/users?sort=status,created_at&order=asc,desc
```

---

## 10. Webhooks

### 10.1 Overview

**Purpose:** Receive real-time notifications when events occur

**Supported Events:**

| Event | Description | Payload |
|-------|-------------|---------|
| user.created | New user created | User object |
| user.updated | User modified | User object |
| user.deleted | User deleted | User ID |
| order.completed | Order finalized | Order object |

### 10.2 Setup

1. Register webhook endpoint in developer portal
2. Verify endpoint with challenge
3. Start receiving events

**Webhook Endpoint Requirements:**
- Must use HTTPS
- Must respond within 5 seconds
- Must return 200 status code

### 10.3 Webhook Payload

```json
{
  "event": "user.created",
  "timestamp": "2024-03-15T16:00:00Z",
  "data": {
    "id": "usr_0987654321",
    "email": "newuser@example.com",
    "name": "Jane Smith"
  },
  "signature": "sha256=abc123..."
}
```

### 10.4 Signature Verification

**Verify webhook authenticity:**

```python
import hmac
import hashlib

def verify_signature(payload, signature, secret):
    expected = hmac.new(
        secret.encode(),
        payload.encode(),
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(f"sha256={expected}", signature)
```

### 10.5 Retry Policy

**Failed webhook delivery:**
- Retry 1: After 5 seconds
- Retry 2: After 1 minute
- Retry 3: After 10 minutes
- Retry 4: After 1 hour

**After 4 failures:** Webhook disabled, requires manual re-enable

---

## 11. Idempotency

### 11.1 Idempotency Keys

**For POST/PUT/DELETE requests:**

```bash
curl -X POST "https://api.company.com/v1/users" \
  -H "Authorization: Bearer YOUR_ACCESS_TOKEN" \
  -H "Idempotency-Key: unique-key-12345" \
  -H "Content-Type: application/json" \
  -d '{ "email": "user@example.com", "name": "John Doe" }'
```

**Behavior:**
- Same key within 24 hours → Returns original response (cached)
- Different key → Creates new resource
- Key expires after 24 hours

**Use Case:** Prevent duplicate operations on network retries

---

## 12. Versioning

### 12.1 Version Strategy

**URI-based versioning:**
```
https://api.company.com/v1/users
https://api.company.com/v2/users
```

### 12.2 Current Versions

| Version | Status | Released | EOL | Notes |
|---------|--------|----------|-----|-------|
| v3 | Beta | 2024-04 | TBD | New features |
| v2 | Current | 2023-01 | 2025-01 | Recommended |
| v1 | Deprecated | 2022-01 | 2024-06 | Use v2 |

### 12.3 Deprecation Policy

1. **Announcement:** 6 months before EOL
2. **Warning headers:** 3 months before EOL
3. **End of Life:** Version removed

**Migration Guide:** https://docs.company.com/api/v1-to-v2

---

## 13. SDKs & Libraries

### 13.1 Official SDKs

| Language | Package | Documentation |
|----------|---------|---------------|
| Python | `pip install company-api` | [Docs](https://github.com/company/python-sdk) |
| JavaScript | `npm install @company/api` | [Docs](https://github.com/company/js-sdk) |
| Ruby | `gem install company-api` | [Docs](https://github.com/company/ruby-sdk) |
| Java | `maven: com.company:api` | [Docs](https://github.com/company/java-sdk) |
| Go | `go get github.com/company/go-sdk` | [Docs](https://github.com/company/go-sdk) |

### 13.2 SDK Example (Python)

```python
from company_api import Client

# Initialize client
client = Client(api_key="your_api_key")

# List users
users = client.users.list(page=1, per_page=20)

# Get specific user
user = client.users.get("usr_1234567890")

# Create user
new_user = client.users.create(
    email="newuser@example.com",
    name="Jane Smith"
)

# Update user
updated_user = client.users.update(
    "usr_1234567890",
    name="John Updated"
)

# Delete user
client.users.delete("usr_1234567890")
```

---

## 14. Testing

### 14.1 Sandbox Environment

**URL:** https://api-sandbox.company.com

**Features:**
- Test data only
- No rate limits
- Reset data daily
- Free to use

**Test Credentials:**
```
Client ID: test_client_id
Client Secret: test_client_secret
API Key: test_api_key_12345
```

### 14.2 Test Data

**Pre-populated test users:**

| ID | Email | Role |
|----|-------|------|
| usr_test_001 | test1@example.com | user |
| usr_test_002 | admin@example.com | admin |
| usr_test_003 | inactive@example.com | user (inactive) |

### 14.3 Postman Collection

**Import collection:**
```
https://api.company.com/v1/postman-collection.json
```

---

## 15. Best Practices

### 15.1 Error Handling

```python
import requests

try:
    response = requests.get(
        "https://api.company.com/v1/users",
        headers={"Authorization": f"Bearer {token}"}
    )
    response.raise_for_status()
    data = response.json()
except requests.exceptions.HTTPError as e:
    if e.response.status_code == 401:
        # Refresh token
        pass
    elif e.response.status_code == 429:
        # Exponential backoff
        pass
    else:
        # Log error
        pass
```

### 15.2 Rate Limit Handling

```python
import time

def api_call_with_retry(url, max_retries=3):
    for attempt in range(max_retries):
        response = requests.get(url)
        
        if response.status_code == 429:
            retry_after = int(response.headers.get('X-RateLimit-Reset', 60))
            time.sleep(retry_after)
            continue
            
        return response
    
    raise Exception("Max retries exceeded")
```

### 15.3 Pagination

```python
def fetch_all_users():
    all_users = []
    page = 1
    
    while True:
        response = client.users.list(page=page, per_page=100)
        all_users.extend(response['data'])
        
        if not response['meta']['has_next']:
            break
            
        page += 1
    
    return all_users
```

---

## 16. Support & Resources

### 16.1 Documentation

- **API Reference:** https://api.company.com/docs
- **Developer Portal:** https://developers.company.com
- **Status Page:** https://status.company.com
- **Changelog:** https://docs.company.com/changelog

### 16.2 Support Channels

| Channel | Response Time | Purpose |
|---------|---------------|---------|
| Email | 24 hours | General questions (api-support@company.com) |
| Slack Community | Best effort | Community help |
| Priority Support | 1 hour | Enterprise customers |
| GitHub Issues | Best effort | SDK bugs |

### 16.3 SLA

**For Enterprise customers:**
- Uptime: 99.9%
- Response time: < 500ms (p95)
- Support: 24/7 with 1-hour response

---

## 17. Security

### 17.1 HTTPS Only

All API requests must use HTTPS. HTTP requests will be rejected.

### 17.2 Token Security

- ✅ Store tokens securely (never in code)
- ✅ Use environment variables
- ✅ Rotate tokens regularly
- ✅ Use short-lived tokens when possible
- ❌ Never log tokens
- ❌ Never commit tokens to git

### 17.3 IP Whitelisting

**Enterprise feature:** Restrict API access to specific IP ranges

**Setup:**
1. Contact support
2. Provide IP ranges (CIDR notation)
3. Test in sandbox
4. Enable in production

### 17.4 Vulnerability Reporting

**Found a security issue?**

Email: security@company.com  
PGP Key: https://company.com/security/pgp-key

**Do NOT:**
- Post publicly
- Test on production
- Access data you don't own

---

## 18. Changelog

### Version 1.0.0 (2024-03-15)

**Added:**
- Initial API release
- User management endpoints
- OAuth 2.0 authentication
- Rate limiting
- Webhooks

**Changed:**
- N/A

**Deprecated:**
- N/A

**Removed:**
- N/A

---

**Document Owner:** API Team  
**Last Updated:** YYYY-MM-DD  
**Next Review:** YYYY-MM-DD
