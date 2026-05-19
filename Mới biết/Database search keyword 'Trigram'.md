### 1. TL;DR

Trigram trong SQL dùng để **so khớp chuỗi gần đúng (fuzzy search)** và **tăng tốc tìm kiếm text**.  
Nó chia chuỗi thành các nhóm 3 ký tự → so sánh độ giống nhau thay vì match exact.

### 2. Mental Model

Đừng nghĩ search text là `"LIKE '%abc%'"`.  
Hãy nghĩ: **biến text thành vector các “mảnh nhỏ” (trigram) → so sánh overlap giữa các vector**.

=> Search becomes **similarity problem**, không còn là string matching thuần nữa.

### 3. Core Concepts

**Trigram là gì?**  
Chuỗi `"hello"` → trigram:

```
" he", "hel", "ell", "llo", "lo "
```

**Similarity**

- So sánh 2 chuỗi bằng số trigram chung
- Thường dùng hàm:
    - `similarity(a, b)`
    - `%` operator (Postgres)

**Index hỗ trợ**

- GIN / GiST index với trigram → search cực nhanh
- Không còn scan full table như `LIKE '%abc%'`

### 4. Practical Usage

#### Khi nào nên dùng

- Search tên user (typo tolerant)
- Autocomplete / suggestion
- Full-text search nhẹ (không cần Elasticsearch)
- Matching gần đúng:  
    `"nguyen"` vs `"nguyenn"`

#### Khi KHÔNG nên dùng

- Exact match → dùng index BTREE
- Structured data (id, email exact)
- Dataset cực lớn cần ranking phức tạp → dùng search engine (Elastic)