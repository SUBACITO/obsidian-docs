# 🚀 Cache System Design — From Basics to Production

---

## 🧠 Bẫy #4 — Không phải data nào cũng nên cache

### ✅ Nên cache (Read-heavy, ít thay đổi)

- Danh sách trending (video, sản phẩm)
    
- Static assets (JS, CSS, images)
    
- Public profile
    
- Query nặng (join nhiều, aggregate)
    

👉 Pattern: **Read >> Write**

---

### ❌ Không nên cache (Consistency critical)

- Số dư tài khoản
    
- Quyền truy cập (authorization)
    
- Inventory realtime
    
- Payment / financial data
    

👉 Pattern: **Sai 1 lần = toang hệ thống**

---

### ⚠️ Core Insight

> Cache không phải để “tăng tốc mọi thứ”  
> → mà là **đánh đổi consistency để lấy performance**

---

## 🧠 Bẫy #5 — Cache Invalidation (bài toán khó nhất)

> “There are only two hard things in Computer Science: cache invalidation…”

### Vấn đề:

- Khi nào data trong cache **không còn đúng**?
    

---

### 3 chiến lược chính

#### 1. TTL (Time-based)

```ts
set(key, value, ttl = 60)
```

- Đơn giản
    
- Nhưng có thể stale
    

---

#### 2. Write-through

```ts
updateDB(data)
updateCache(data)
```

- Strong consistency hơn
    
- Tốn effort
    

---

#### 3. Cache-aside (phổ biến nhất)

```ts
const data = cache.get(key)
if (!data) {
  const fresh = db.query()
  cache.set(key, fresh)
}
```

- Linh hoạt
    
- Nhưng dễ stale nếu quên invalidate
    

---

## 🧠 Bẫy #6 — Cache nhiều tầng (Multi-layer cache)

### Flow thực tế (request đi qua):

```
Browser Cache
   ↓
CDN (Cloudflare)
   ↓
Reverse Proxy (Nginx / Varnish)
   ↓
App Cache (in-memory)
   ↓
Redis (distributed)
   ↓
DB Cache (query cache)
   ↓
Database (source of truth)
```

---

### ⚠️ Nguy hiểm

> Chỉ cần **1 tầng trả sai data** → toàn hệ thống sai

---

### 🧠 Mental Model

- Cache ≠ 1 layer
    
- Cache = **stack nhiều layer chồng lên nhau**
    

---

## 🧠 Bẫy #7 — Cache Stampede (Thảm họa production)

### 💥 Scenario

- Cache hết hạn cùng lúc
    
- 1000 request → miss cache
    
- → ALL hit DB
    

```
CACHE EXPIRED → DB OVERLOADED → SYSTEM DOWN
```

---

### 🔥 Solutions

#### 1. Random TTL (jitter)

```ts
ttl = baseTTL + random(0, 30)
```

---

#### 2. Distributed Lock

```ts
if (acquireLock(key)) {
  rebuildCache()
}
```

- Chỉ 1 request rebuild cache
    

---

#### 3. Request Collapsing

- N request → 1 request fetch data
    
- Những request khác wait
    

---

#### 4. Preload / Warmup Cache

- Cron job load sẵn data
    

---

#### 5. Stale-while-revalidate (🔥 best practice)

```ts
return staleData
asyncRefreshCache()
```

- User luôn có response
    
- Cache update async
    

---

## ⚙️ Practical Usage

### Khi nên dùng cache mạnh

- Feed (Facebook, TikTok)
    
- Chat list (last message)
    
- Dashboard analytics
    

---

### Khi nên cẩn thận

- Banking
    
- Inventory
    
- Auth system
    

---

## ⚠️ Pitfalls / Gotchas

- ❌ Cache key design kém → collision
    
- ❌ Không invalidate → data sai lâu dài
    
- ❌ Cache quá nhiều → memory pressure
    
- ❌ Không monitor hit rate
    

---

## 📊 Advanced Notes

### Cache hierarchy mindset

|Layer|Latency|Scope|
|---|---|---|
|Browser|~0ms|per user|
|CDN|~10ms|global|
|Redis|~1ms|backend|
|DB|~10–100ms|source|

---

### Trade-off lớn nhất

|Factor|Cache|No Cache|
|---|---|---|
|Speed|⚡|🐢|
|Consistency|❌|✅|
|Complexity|🔥|😌|

---

## 📌 Kết luận

- Cache là **performance multiplier**, không phải silver bullet
    
- Sai cache → **bug khó debug nhất system**
    
- System lớn = **multi-layer cache + invalidation strategy**
    

---

👉 Nếu bạn đang build chat system:

- Cache conversation list (OK)
    
- Cache message realtime (cẩn thận)
    
- Presence / online status → **KHÔNG cache lâu**
    

---
