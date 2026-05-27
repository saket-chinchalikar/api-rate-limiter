# 🛡️ Smart API Rate Limiter

> A drop-in **Spring Boot** middleware library with three rate-limiting algorithms — Fixed Window, Token Bucket, and Sliding Window. Add a single `@RateLimit` annotation to any controller method to protect it, with Redis-backed persistence and standard HTTP response headers.

![Status](https://img.shields.io/badge/Status-In_Development-yellow?style=for-the-badge) <br>
[![Java](https://img.shields.io/badge/Java-21-ED8B00?style=for-the-badge&logo=openjdk&logoColor=white)](https://openjdk.org/projects/jdk/21/) <br>
[![Spring Boot](https://img.shields.io/badge/Spring_Boot-3.x-6DB33F?style=for-the-badge&logo=spring&logoColor=white)](https://spring.io/projects/spring-boot) <br>
[![Redis](https://img.shields.io/badge/Redis-DC382D?style=for-the-badge&logo=redis&logoColor=white)](https://redis.io) <br>
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](LICENSE)

---

## The problem

Every production API needs rate limiting — but implementing it correctly is surprisingly hard. Fixed windows have burst vulnerabilities. Token buckets need careful state management. Sliding windows are the most accurate but the trickiest to build. This project implements all three correctly, backed by Redis, and exposes them through a clean annotation API.

---

## Planned usage (3 lines to protect any endpoint)

```java
@RestController
public class MyController {

    @RateLimit(algorithm = Algorithm.TOKEN_BUCKET, requestsPerMinute = 20)
    @GetMapping("/api/data")
    public ResponseEntity<Data> getData() {
        return ResponseEntity.ok(service.getData());
    }
}
```

---

## Algorithm comparison

| Algorithm | Accuracy | Burst handling | Complexity | Best for |
|---|---|---|---|---|
| Fixed Window | Medium | ❌ Vulnerable at edges | Low | Simple APIs, low traffic |
| Token Bucket | High | ✅ Allows controlled bursts | Medium | Most production APIs |
| Sliding Window | Highest | ✅ Smoothest limiting | High | High-precision rate control |

---

## Planned features

- [ ] `@RateLimit` annotation via Spring AOP — zero boilerplate for route protection
- [ ] Fixed Window — Redis `INCR` + `EXPIRE` per user per window
- [ ] Token Bucket — Bucket4j + `RedisProxyManager` for persistent buckets
- [ ] Sliding Window — Redis sorted sets (`ZADD` / `ZREMRANGEBYSCORE`) for exact timestamps
- [ ] Per-user, per-IP, and per-endpoint keying strategies
- [ ] Standard response headers: `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `Retry-After`
- [ ] Admin endpoints: view and reset user limits
- [ ] JUnit 5 + Testcontainers test suite (real Redis in Docker)
- [ ] Load test results in README

---

## Tech stack

| Layer | Technology |
|---|---|
| Language | Java 21 |
| Framework | Spring Boot 3.x |
| Rate limiting | Bucket4j + Spring AOP |
| Persistence | Redis 7 via Spring Data Redis |
| Auth | Spring Security (admin endpoints) |
| Testing | JUnit 5 · Mockito · Testcontainers |
| Build | Maven 3.9 |

---

## Architecture

```
HTTP Request
    │
    └── OncePerRequestFilter (Spring)
          │
          ├── Extract user identity (JWT / IP)
          │
          ├── Select algorithm (Fixed / Token Bucket / Sliding Window)
          │
          ├── Check Redis ──► ALLOW ──► Controller
          │                │
          │                └── DENY ──► 429 Too Many Requests
          │                             + X-RateLimit-* headers
          │
          └── Admin: GET/DELETE /admin/limits/{userId}
```

---

## Getting started

> ⚠️ **This project is in active development.** Setup instructions will be added as the project is built.

```bash
git clone https://github.com/saket-chinchalikar/api-rate-limiter.git
cd api-rate-limiter
# Requires Java 21 + Docker (for Redis)
# Setup instructions coming soon
```

---

## Roadmap

- [ ] Publish as a Maven dependency on GitHub Packages
- [ ] Prometheus metrics for rate limit hit rate
- [ ] Dynamic limit configuration (change limits without restart)
- [ ] Distributed token bucket across multiple app instances

---

## License

[MIT](LICENSE) · Built by [@saket-chinchalikar](https://github.com/saket-chinchalikar)
