# URL Shortener — High-Level Design (HLD)

This document describes the **high-level architecture** of a URL Shortener system. The goal is to map requirements to components and understand **data flow, scalability, and trade-offs** — without going into low-level implementation details.

---

## 1. Requirements Recap

### Functional

- Create a short URL from a long URL
- Redirect short URL to original URL

### Non-Functional

- Read-heavy system (redirects >> writes)
- Redirect latency < 100ms
- 99.99% availability
- Horizontally scalable
- Strong consistency for URL creation
- Cost-efficient reads

---

## 2. High-Level Components

The system consists of the following major components:

1. Client (Browser / Mobile App)
2. CDN
3. Load Balancer
4. URL Service (API Layer)
5. Cache (Redis)
6. Persistent Storage (Database)

---

## 3. Architecture Overview

### Component Responsibilities

#### Client

- Sends request to shorten a URL
- Uses short URL for redirection

#### CDN

- Caches redirect responses for hot URLs
- Reduces latency for global users
- Offloads traffic from origin servers

#### Load Balancer

- Distributes traffic across API servers
- Enables horizontal scaling
- Removes unhealthy instances

#### URL Service (Stateless)

- Handles URL creation and redirection
- Generates unique short codes
- Reads from cache / database

#### Cache (Redis)

- Stores shortCode → longURL mappings
- Optimized for fast reads
- Primary optimization for read-heavy traffic

#### Database

- Persistent storage for URL mappings
- Source of truth
- Ensures durability and correctness

---

## 4. Data Flow

### 4.1 URL Creation Flow (Write Path)

1. Client sends long URL to URL Service
2. URL Service generates a unique short code
3. Mapping is stored in the database
4. Cache is updated (write-through or async)
5. Short URL is returned to the client

**Key Points:**

- Writes are less frequent
- Strong consistency is required
- Database is the source of truth

---

### 4.2 URL Redirection Flow (Read Path)

1. Client hits short URL
2. Request goes through CDN
3. If cached → immediate redirect
4. If cache miss:

   - Load Balancer routes to URL Service
   - URL Service checks Redis cache
   - If cache hit → redirect
   - If cache miss → fetch from database
   - Update cache
   - Redirect client

**Key Points:**

- Cache-first approach
- Database accessed only on cache miss
- Designed for extremely low latency

---

## 5. Scalability Strategy

### Application Layer

- Stateless services
- Horizontal scaling behind load balancer

### Cache Layer

- Redis cluster
- Sharded by short code

### Database Layer

- Primary DB for writes
- Read replicas for scale
- Future sharding based on short code range

---

## 6. Availability & Fault Tolerance

- Multiple API instances across zones
- Cache failures fall back to database
- Database replication for durability
- CDN absorbs traffic spikes

No single point of failure exists in the request path.

---

## 7. Trade-Offs & Decisions

### Why Cache?

- Redirect traffic is extremely read-heavy
- Database-only reads would not scale cost-effectively

### Why Stateless Services?

- Enables easy horizontal scaling
- Simplifies load balancing

### Why Database as Source of Truth?

- Ensures durability
- Cache can be rebuilt

---

## 8. What We Are Not Solving Yet

- Custom aliases
- URL expiration
- Analytics
- Abuse detection
- Authentication

These will be layered on top later without changing the core architecture.

---

## 9. Summary

This architecture prioritizes:

- Fast redirects
- Horizontal scalability
- High availability
- Cost-efficient reads
