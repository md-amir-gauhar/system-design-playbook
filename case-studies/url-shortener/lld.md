# URL Shortener — Low-Level Design (LLD)

This document dives into **implementation-level details** of the URL Shortener system. It focuses on **APIs, data models, algorithms, and failure handling**, while staying aligned with the HLD.

---

## 1. API Design

### 1.1 Create Short URL

**Endpoint**

```
POST /api/v1/shorten
```

**Request Body**

```json
{
  "longUrl": "https://example.com/some/very/long/url",
  "customAlias": "ex123", // optional
  "expiresAt": "2026-01-01T00:00:00Z" // optional
}
```

**Response**

```json
{
  "shortUrl": "https://sho.rt/abC12",
  "code": "abC12"
}
```

**Notes**

- `customAlias` must be unique
- Validation required for URL format
- Idempotency can be supported via idempotency-key header (optional)

---

### 1.2 Redirect Short URL

**Endpoint**

```
GET /{code}
```

**Flow**

1. Check cache for `code → longUrl`
2. If cache miss → DB lookup
3. If found → redirect (HTTP 302)
4. Async event for analytics

---

## 2. Database Design

### 2.1 URL Mapping Table

```
urls
-----
id (UUID, PK)
code (varchar, unique, indexed)
long_url (text)
created_at (timestamp)
expires_at (timestamp, nullable)
```

**Indexes**

- Unique index on `code`

**Why relational?**

- Strong consistency for writes
- Simple lookup pattern

---

## 3. Short Code Generation Strategy

### Option Chosen: Base62 Encoding

**Why?**

- Compact
- URL-safe
- Large keyspace

**Flow**

1. Generate unique numeric ID (DB sequence / Snowflake)
2. Encode ID using Base62
3. Store mapping in DB

**Example**

```
ID: 125789 → Base62 → abC12
```

---

## 4. Caching Strategy

### Cache Type

- Redis / in-memory cache

### Cache Key

```
key: code
value: longUrl
```

### TTL

- Long TTL (e.g., 24h)
- Invalidate on expiration

### Cache Miss Handling

- Fallback to DB
- Populate cache after DB hit

---

## 5. Expiration Handling

### Strategy

- Check `expires_at` during redirect
- If expired → return 404
- Optional background cleanup job

---

## 6. Analytics (Async)

Captured events:

- code
- timestamp
- IP / user-agent (optional)

**Why async?**

- Redirect path must stay fast

---

## 7. Failure Scenarios

### Cache Down

- Redirect via DB
- Higher latency but system works

### DB Read Replica Lag

- Acceptable (redirects are tolerant)

### Duplicate Code Collision

- Retry generation
- Enforced by DB unique constraint

---

## 8. Security Considerations

- Rate limiting on create API
- Input validation
- Prevent open redirect abuse

---

## 9. What We Intentionally Skip

- Auth & user accounts
- Abuse / malware detection
- Billing

---

## 10. Mapping to HLD

| HLD Component | LLD Detail        |
| ------------- | ----------------- |
| API Service   | REST endpoints    |
| Cache         | Redis key-value   |
| Database      | URL mapping table |
| Async Queue   | Analytics events  |
