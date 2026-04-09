---
name: caching-strategies
description: >-
  Caching design across HTTP, CDN, application, and client layers. Use when:
  adding cache headers, configuring CloudFront or CDN behavior, implementing
  Redis or in-memory caching, designing IndexedDB or service worker caches,
  choosing cache invalidation strategies, auditing an existing application for
  missing or misconfigured caches, or optimizing API response times. Covers
  HTTP cache-control, CDN edge caching, application-level caching, client-side
  storage, cache invalidation patterns, and cache warming.
tags:
  - developer
  - operations
---

# Caching Strategies

## When to Use

- Adding HTTP cache headers to API responses or static assets
- Configuring CDN (CloudFront, Cloudflare) cache behavior and TTLs
- Implementing Redis, Memcached, or in-memory caching for API responses or computed data
- Designing client-side caching with IndexedDB, localStorage, or service workers
- Choosing a cache invalidation strategy for a specific use case
- Auditing an existing application for missing, stale, or misconfigured caches
- Reducing database load or API latency with read-through caching
- Warming caches on deploy or after invalidation

---

## Core Principle: Cache Close to the Consumer

**The best cache is the one closest to the requester.** Cache at the edge before the origin, at the application before the database, at the client before the network. Each cache layer reduces load on the layers behind it.

---

## Cache Layers (Outer to Inner)

| Layer | Where | TTL range | Invalidation | Example |
|-------|-------|-----------|-------------|---------|
| **Browser** | Client | Seconds to years | Cache-Control headers, versioned URLs | Static assets, API responses |
| **Service Worker** | Client | App-controlled | Programmatic via Cache API | Offline-first PWA data |
| **CDN / Edge** | Edge PoP | Minutes to days | Purge API, TTL expiry | CloudFront, Cloudflare |
| **API Gateway** | Origin edge | Seconds to minutes | TTL expiry | API Gateway response cache |
| **Application** | Server memory or Redis | Seconds to hours | Explicit delete, TTL | Computed aggregations, session data |
| **Database** | Query layer | Automatic | Query plan cache, materialized views | PostgreSQL query cache |

### Rule: Never Cache Authentication or User-Specific Data at Shared Layers

Shared caches (CDN, API Gateway) must never serve one user's data to another. Mark user-specific responses as `private` or `no-store`.

---

## HTTP Cache-Control Headers

### Static Assets (CSS, JS, images, fonts)

```
Cache-Control: public, max-age=31536000, immutable
```

Use content-hashed filenames (`app.a1b2c3.js`) so the URL changes when content changes. Set `max-age` to 1 year. The `immutable` directive tells browsers not to revalidate.

### API Responses (Public, Cacheable)

```
Cache-Control: public, max-age=60, stale-while-revalidate=300
```

- `max-age=60`: Fresh for 60 seconds.
- `stale-while-revalidate=300`: Serve stale for up to 5 minutes while fetching fresh copy in background.

### API Responses (Private, User-Specific)

```
Cache-Control: private, max-age=0, must-revalidate
```

Prevents CDN/proxy caching. Browser may cache but must revalidate on every request.

### No Caching

```
Cache-Control: no-store
```

Use for sensitive data (auth tokens, PII, financial data). `no-store` is stronger than `no-cache` — it prevents storage entirely.

### ETag and Conditional Requests

```python
# FastAPI example
from hashlib import sha256

@app.get("/api/config")
def get_config(request: Request) -> Response:
    data = load_config()
    body = json.dumps(data)
    etag = sha256(body.encode()).hexdigest()[:16]

    if request.headers.get("if-none-match") == etag:
        return Response(status_code=304)

    return Response(
        content=body,
        headers={"ETag": etag, "Cache-Control": "private, max-age=0, must-revalidate"},
    )
```

---

## CDN / Edge Caching

### CloudFront Cache Behavior

| Path pattern | TTL | Origin | Cache policy |
|-------------|-----|--------|-------------|
| `/static/*` | 1 year | S3 | `CachingOptimized` (query strings ignored) |
| `/api/public/*` | 60s | ALB/Lambda | Forward `Accept`, `Accept-Encoding` |
| `/api/private/*` | 0 | ALB/Lambda | `CachingDisabled` |
| `Default (*)` | 1 day | S3 | `CachingOptimized` |

### Cache Key Design

Include only what differentiates responses:

- **Good**: Path + `Accept-Language` header (for localized content).
- **Bad**: Path + `Authorization` header (every user gets a different cache entry — defeats purpose).
- **Rule**: Fewer cache key components = higher hit rate.

### Invalidation

```bash
# Invalidate specific paths
aws cloudfront create-invalidation \
  --distribution-id E1234567890 \
  --paths "/api/public/products" "/api/public/categories"

# Invalidate everything (expensive — use sparingly)
aws cloudfront create-invalidation \
  --distribution-id E1234567890 \
  --paths "/*"
```

**Prefer versioned URLs over invalidation.** Invalidation is slow (minutes), costly at scale, and cannot be undone.

---

## Application-Level Caching

### Redis / ElastiCache

```python
import json
import hashlib
from collections.abc import Callable
from typing import Any

import redis

cache = redis.Redis(host="cache.example.com", port=6379, decode_responses=True)


def cached(prefix: str, ttl_seconds: int = 300) -> Callable[..., Any]:
    """Decorator for caching function results in Redis."""
    def decorator(func: Callable[..., Any]) -> Callable[..., Any]:
        def wrapper(*args: Any, **kwargs: Any) -> Any:
            key_data = json.dumps({"args": args, "kwargs": kwargs}, sort_keys=True)
            cache_key = f"{prefix}:{hashlib.sha256(key_data.encode()).hexdigest()[:16]}"

            cached_value = cache.get(cache_key)
            if cached_value is not None:
                return json.loads(cached_value)

            result = func(*args, **kwargs)
            cache.setex(cache_key, ttl_seconds, json.dumps(result))
            return result
        return wrapper
    return decorator


@cached("products", ttl_seconds=60)
def get_products(category: str) -> list[dict[str, Any]]:
    return db.query_products(category)
```

### In-Memory Caching (Python)

```python
from functools import lru_cache


@lru_cache(maxsize=256)
def compute_expensive_result(input_hash: str) -> dict[str, Any]:
    """Cache in process memory. Use for immutable inputs only."""
    return heavy_computation(input_hash)
```

**Warning**: In-memory caches are per-process. In Lambda, each concurrent execution has its own cache. Use Redis for shared state.

### Cache-Aside (Read-Through) Pattern

1. Check cache.
2. On miss: query database, write to cache, return result.
3. On hit: return cached value.

This is the most common pattern. The application manages the cache explicitly.

### Write-Through Pattern

1. Write to cache AND database on every update.
2. Reads always hit cache.

Use when read-after-write consistency is required. More complex, but eliminates stale reads.

---

## Client-Side Caching

### React Query / TanStack Query

```typescript
const { data } = useQuery({
  queryKey: ["products", categoryId],
  queryFn: () => fetchProducts(categoryId),
  staleTime: 5 * 60 * 1000,      // Fresh for 5 minutes
  gcTime: 30 * 60 * 1000,         // Keep in memory for 30 minutes
  refetchOnWindowFocus: false,     // Don't refetch on tab switch
});
```

### IndexedDB for Offline Data

```typescript
async function getCachedOrFetch<T>(
  storeName: string,
  key: string,
  fetcher: () => Promise<T>,
  maxAgeMs: number = 5 * 60 * 1000,
): Promise<T> {
  const cached = await idb.get(storeName, key);
  if (cached && Date.now() - cached.timestamp < maxAgeMs) {
    return cached.data as T;
  }

  const fresh = await fetcher();
  await idb.put(storeName, { key, data: fresh, timestamp: Date.now() });
  return fresh;
}
```

### Service Worker Cache

```typescript
// Cache-first for static assets, network-first for API
self.addEventListener("fetch", (event: FetchEvent) => {
  const url = new URL(event.request.url);

  if (url.pathname.startsWith("/static/")) {
    event.respondWith(caches.match(event.request).then((r) => r ?? fetch(event.request)));
  } else if (url.pathname.startsWith("/api/")) {
    event.respondWith(
      fetch(event.request)
        .then((response) => {
          const clone = response.clone();
          caches.open("api-cache").then((cache) => cache.put(event.request, clone));
          return response;
        })
        .catch(() => caches.match(event.request).then((r) => r ?? new Response("Offline", { status: 503 }))),
    );
  }
});
```

---

## Cache Invalidation Strategies

| Strategy | How it works | Best for |
|----------|-------------|----------|
| **TTL expiry** | Cache entry expires after fixed duration | Tolerant of slight staleness (product listings, feeds) |
| **Event-driven purge** | Publish event on write → subscriber deletes cache key | Strong consistency needs (user profile, permissions) |
| **Versioned keys** | Include version/hash in cache key; new version = new key | Configuration, feature flags |
| **Write-through** | Update cache on every write | Read-heavy with frequent writes |
| **Cache stampede prevention** | Lock during recomputation; others wait or serve stale | Expensive computations with high concurrency |

### Cache Stampede Prevention

```python
import time

LOCK_TTL = 10  # seconds


def get_with_lock(key: str, compute_fn: Callable[[], Any], ttl: int = 300) -> Any:
    """Prevent thundering herd on cache miss."""
    value = cache.get(key)
    if value is not None:
        return json.loads(value)

    lock_key = f"lock:{key}"
    if cache.set(lock_key, "1", nx=True, ex=LOCK_TTL):
        try:
            result = compute_fn()
            cache.setex(key, ttl, json.dumps(result))
            return result
        finally:
            cache.delete(lock_key)
    else:
        # Another process is computing — wait briefly and retry
        time.sleep(0.1)
        value = cache.get(key)
        return json.loads(value) if value else compute_fn()
```

---

## Cache Warming

Warm caches on deploy to avoid cold-start latency:

```python
def warm_cache() -> None:
    """Call after deployment to pre-populate critical caches."""
    popular_categories = ["electronics", "clothing", "home"]
    for category in popular_categories:
        get_products(category)  # Triggers cache-aside population
```

**When to warm**: After deploys, after cache flushes, for predictably popular content.
**When not to warm**: For long-tail content (millions of unique keys) — let demand drive caching.

---

## Anti-Patterns

| Anti-Pattern | Problem | Fix |
|-------------|---------|-----|
| Cache everything forever | Stale data served indefinitely | Set explicit TTLs on every cache entry |
| No cache at all | Every request hits origin | Add Cache-Control headers at minimum |
| Caching user-specific data in shared CDN | Data leaks between users | Use `Cache-Control: private` or `no-store` |
| Cache key includes irrelevant parameters | Low hit rate, wasted memory | Minimize cache key components |
| No invalidation strategy | Stale data until TTL expires | Use event-driven purge for mutable data |
| Cache stampede on popular keys | Origin overwhelmed on expiry | Use locking or stale-while-revalidate |
| Caching errors | Error responses cached and served to all | Never cache 5xx; cache 404 briefly if intentional |
| Using cache as primary data store | Data loss on eviction or restart | Cache is ephemeral — always have a source of truth |

---

## Audit Checklist

When auditing an existing application for caching:

- [ ] Static assets have long `max-age` with content-hashed filenames
- [ ] API responses have appropriate `Cache-Control` headers (not missing entirely)
- [ ] User-specific responses use `private` or `no-store`
- [ ] Sensitive data (auth, PII) uses `no-store`
- [ ] CDN cache key includes only differentiating parameters
- [ ] Application cache has TTLs on every key (no indefinite caching)
- [ ] Cache invalidation strategy exists for mutable data
- [ ] Cache stampede prevention exists for expensive computations
- [ ] Error responses are not cached (or cached only briefly and intentionally)
- [ ] Cache warming runs on deploy for critical paths
- [ ] Client-side `staleTime` / `gcTime` configured for data fetching library
- [ ] Service worker cache strategy matches data freshness requirements
