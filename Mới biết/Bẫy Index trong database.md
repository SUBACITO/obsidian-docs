# 🚨 SQL Index Pitfalls (Production Reality)

## 🧠 Bẫy #1 — Function trên column (Index bị “mù”)

### ❌ Sai

```sql
WHERE DATE(created_at) = '2026-05-10'
```

- Bọc column trong function → DB **không dùng được index**
    
- Vì index lưu **raw value**, không phải kết quả sau khi transform
    

### ✅ Đúng (Range Scan)

```sql
WHERE created_at >= '2026-05-10 00:00:00'
  AND created_at <  '2026-05-11 00:00:00'
```

- Giữ nguyên column → DB dùng **range scan**
    
- Complexity: từ `O(n)` → `O(log n)`
    

---

## 🧠 Bẫy #2 — Wildcard sai vị trí (`LIKE '%...'`)

### ❌ Sai

```sql
WHERE email LIKE '%gmail.com'
```

- `%` ở đầu → mất **start key**
    
- B-Tree không thể seek → **full table scan**
    

### ✅ Đúng (Prefix Search)

```sql
WHERE email LIKE 'user%'
```

### 🚀 Nếu cần substring thật

```sql
-- PostgreSQL
CREATE EXTENSION pg_trgm;

CREATE INDEX idx_email_trgm
ON users USING GIN (email gin_trgm_ops);

SELECT *
FROM users
WHERE email % 'gmail.com';
```

---

## 🧠 Bẫy #3 — Composite Index sai thứ tự

### 🎯 Index

```sql
INDEX(status, created_at)
```

### ❌ Sai

```sql
WHERE created_at = '2026-05-10'
```

- Bỏ qua `status` → vi phạm **leftmost prefix rule**
    
- DB không dùng index
    

### ✅ Đúng

```sql
WHERE status = 'active'
  AND created_at = '2026-05-10'
```

Hoặc:

```sql
WHERE status = 'active'
```

---

## 🔑 Rule quan trọng (Leftmost Prefix)

Index `(a, b, c)` chỉ dùng được khi:

|Query|Dùng index?|
|---|---|
|`a`|✅|
|`a, b`|✅|
|`a, b, c`|✅|
|`b`|❌|
|`b, c`|❌|

---

## ⚙️ Mental Model tổng hợp

- Index ≈ **sorted data structure**
    
- DB chỉ dùng index khi:
    
    - Có thể **seek (điểm bắt đầu rõ ràng)**
        
    - Không bị **transform column**
        
    - Tuân thủ **thứ tự index**
        

---

## ⚠️ Bonus Pitfalls (hay gặp ở production)

### 1. Function ngầm (hidden killer)

```sql
WHERE LOWER(email) = 'test@gmail.com'
```

→ Giống như bẫy #1

✔ Fix:

```sql
CREATE INDEX idx_email_lower ON users (LOWER(email));
```

---

### 2. Type mismatch

```sql
WHERE user_id = '123' -- string vs int
```

→ Có thể gây **implicit cast** → index không dùng

---

### 3. OR condition

```sql
WHERE status = 'active' OR created_at > NOW()
```

→ DB có thể bỏ index → chuyển sang scan

---

## 🚀 Advanced Insight

- B-Tree = **range-based index**, không phải search engine
    
- Trigram = **substring index (hack thông minh)**
    
- Full-text / Elastic = **semantic search layer**
    

---

## 📌 Kết luận

|Pattern|Index usable?|
|---|---|
|`column = value`|✅|
|`column BETWEEN ...`|✅|
|`LIKE 'abc%'`|✅|
|`LIKE '%abc%'`|❌|
|`FUNCTION(column)`|❌|
|Skip column trong composite index|❌|

---

👉 Nếu system của bạn:

- Có search text → dùng **trigram**
    
- Có filter + sort → optimize **composite index**
    
- Có nhiều query phức tạp → cần **query plan tuning**
    

---
