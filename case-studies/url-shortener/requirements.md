# 🔗 URL Shortener

## 1. ✨ Functional Requirements

**🎯 Core Features:**

- URL shortening
- URL redirections

**⭐ Optional Features:**

- User defined short URL
- Analytics
- URL Expiry
- User Authentication

## 2. ⚡ Non-Functional Requirements

- **Latency**: < 100ms redirect time
- **Scale**: 100M DAU, 1B reads/day (~10K QPS)
- **Storage**: 1-5B total URLs
- **Availability**: 99.99% uptime
- **Optimization**: Read-heavy (optimize for fast reads)
- **Storage**: Compact and efficient

## 3. 🚫 Out of Scope (Important to State Explicitly)

We will **NOT** handle:

- Abuse detection
- Malware scanning
- Authentication
- Payments

(These are real problems but distract from core design)

## 4. 💭 Assumptions

- URLs are primarily short-lived (90% expire within 1 year)
- Read:Write ratio is approximately 100:1
- Average URL length is ~200 characters
- Peak traffic is 2x average traffic
- Users are distributed globally
- Most traffic comes from popular URLs (80-20 rule applies)
- No GDPR or data privacy requirements for MVP
- Short codes are case-sensitive
- URLs don't need to be human-readable
- System starts with single region deployment

## 5. Capacity Estimation

### Traffic

- **Daily Active Users**: 100M
- **Reads per day**: 1B → **~12K QPS** (peak ~24K QPS)
- **Writes per day**: 10M → **~120 QPS** (peak ~240 QPS)
- **Read:Write ratio**: **100:1** (read-heavy)

### Storage

- **URL record size**: ~500 bytes (original URL + metadata)
- **Total URLs**: 5B over 10 years
- **Storage needed**: 5B × 500 bytes = **2.5 TB**

### Bandwidth

- **Read bandwidth**: 12K QPS × 500 bytes = **6 MB/s**
- **Write bandwidth**: 120 QPS × 500 bytes = **60 KB/s**

### Cache (80-20 rule)

- 20% of URLs generate 80% of traffic
- Cache size: 20% of daily reads × 500 bytes
- **Cache needed**: ~100 GB (for hot URLs)
