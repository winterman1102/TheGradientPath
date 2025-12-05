# 📚 SemanticText2SQL - Tài Liệu Luồng Hoạt Động Hệ Thống (Tiếng Việt)

## 🎯 Tổng Quan Hệ Thống

**SemanticText2SQL** là một hệ thống chuyển đổi ngôn ngữ tự nhiên sang SQL được hỗ trợ bởi AI, kết hợp ba công nghệ bổ trợ để tạo nên khả năng truy vấn cơ sở dữ liệu thông minh:

1. **Bộ Lọc SQL Truyền Thống** - Độ chính xác cao & tốc độ nhanh cho dữ liệu có cấu trúc
2. **Fuzzy Matching với Levenshtein** - Chấp nhận lỗi gõ phím trong các trường văn bản
3. **Vector Embeddings** - Hiểu ngữ nghĩa và khái niệm

---

## 🏗️ Kiến Trúc Hệ Thống

```
┌─────────────────────────────────────────────────────────────────────┐
│                    HỆ THỐNG SEMANTIC TEXT2SQL                        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  Câu Hỏi Ngôn Ngữ Tự Nhiên (Bất Kỳ Ngôn Ngữ Nào)                   │
│            ↓                                                          │
│  ┌─────────────────────────────────────────────────────────┐        │
│  │  AGENT (text_to_sql_agent.py)                            │        │
│  │  ├── Sinh SQL (LLM)                                      │        │
│  │  ├── Sinh Embedding (OpenAI)                             │        │
│  │  ├── Xác Thực Truy Vấn (sqlglot)                         │        │
│  │  ├── Thực Thi Truy Vấn (psycopg2)                        │        │
│  │  └── Sinh Câu Trả Lời (LLM)                              │        │
│  └─────────────────────────────────────────────────────────┘        │
│            ↓                                                          │
│  ┌─────────────────────────────────────────────────────────┐        │
│  │  CƠ SỞ DỮ LIỆU POSTGRESQL (books_db)                     │        │
│  │  ├── 5 Bảng (authors, books, publishers, v.v.)           │        │
│  │  ├── Các Trường Văn Bản + Vector Embeddings              │        │
│  │  ├── Extension pgvector (tìm kiếm tương đồng)            │        │
│  │  └── Extension fuzzystrmatch (Levenshtein)               │        │
│  └─────────────────────────────────────────────────────────┘        │
│            ↓                                                          │
│  Câu Trả Lời Ngôn Ngữ Tự Nhiên với Kết Quả                          │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Luồng Hoạt Động Đầy Đủ (5 Bước)

### **Bước 1: Sinh SQL 🧠**

**Đầu Vào**: Câu hỏi ngôn ngữ tự nhiên của người dùng  
**Xử Lý**: LLM phân tích ý định và tạo chiến lược SQL  
**Đầu Ra**: Câu truy vấn SQL + tham số embedding (nếu cần)

```
Câu Hỏi Người Dùng
    ↓
System Prompt (với schema DB đầy đủ)
    ↓
Phân Tích LLM
    ├── Xác định loại truy vấn
    ├── Quyết định chiến lược tìm kiếm
    │   ├── Bộ Lọc SQL (ngày tháng, giá cả, ID)
    │   ├── Fuzzy Matching (tên có lỗi gõ)
    │   ├── Vector Embeddings (khái niệm/chủ đề)
    │   └── Kết Hợp (nhiều chiến lược)
    └── Sinh SQL với placeholder
    ↓
JSON Response
{
  "sql_query": "SELECT ... WHERE ...",
  "need_embedding": true/false,
  "embedding_params": [
    {
      "placeholder": "embedding_1",
      "text_to_embed": "từ khóa tìm kiếm",
      "table_field": "books.description_embed"
    }
  ]
}
```

**Thành Phần Chính**:
- `text_to_sql_agent.py::generate_sql()`
- `prompt.py::create_text_to_sql_prompt()`
- Sử dụng GPT-4.1 mặc định (có thể cấu hình)
- Temperature: 0.1 (để đảm bảo tính nhất quán)
- Định dạng phản hồi: JSON

**Giải Thích Temperature**:
- Temperature là tham số điều chỉnh độ ngẫu nhiên của LLM
- Giá trị thấp (0.0-0.2): Output ổn định, deterministic → dùng cho sinh SQL
- Giá trị cao (0.6-0.9): Output đa dạng, sáng tạo → dùng cho sinh văn bản
- Ở đây dùng 0.1 để SQL luôn nhất quán và ít lỗi

---

### **Bước 2: Sinh Embedding 🔮**

**Kích Hoạt Khi**: `need_embedding = true`  
**Xử Lý**: Chuyển đổi từ khóa tìm kiếm thành vector 1536 chiều  
**Đầu Ra**: Chuỗi vector định dạng PostgreSQL

```
Tham Số Embedding
    ↓
Với mỗi tham số:
    ├── Trích xuất text_to_embed
    ├── Gọi OpenAI Embedding API
    │   ├── Model: text-embedding-3-small
    │   └── Số chiều: 1536
    ├── Nhận vector [0.123, -0.456, ...]
    └── Định dạng cho PostgreSQL: '[0.123,-0.456,...]'
    ↓
Thay thế placeholder trong SQL
    ↓
Câu truy vấn SQL cuối cùng có thể thực thi
```

**Cách Triển Khai**:
```python
# Từ text_to_sql_agent.py
def _generate_embeddings(self, embedding_params):
    embeddings = {}
    for param in embedding_params:
        response = self.client.embeddings.create(
            model=EMBEDDING_MODEL,
            input=param["text_to_embed"]
        )
        vector = response.data[0].embedding
        embeddings[param["placeholder"]] = str(vector)
    return embeddings
```

**Tại Sao Cần Embeddings?**
- Hiểu **ý nghĩa**, không chỉ từ khóa
- Tìm "toàn trị" khi tìm kiếm "giám sát"
- Tìm "tự do" khi tìm kiếm "quyền lợi"
- Hoạt động với đồng nghĩa và khái niệm

---

### **Bước 3: Xác Thực SQL 🛡️**

**Mục Đích**: Xác thực bảo mật và cấu trúc  
**Xử Lý**: Xác thực đa lớp sử dụng sqlglot  
**Đầu Ra**: Pass/Fail kèm thông báo lỗi chi tiết

```
Câu Truy Vấn SQL Đã Sinh
    ↓
┌─────────────────────────────────────┐
│  XÁC THỰC BẢO MẬT                   │
├─────────────────────────────────────┤
│  ✓ Chỉ cho phép SELECT              │
│  ✓ Không có INSERT/UPDATE/DELETE    │
│  ✓ Không có pattern SQL injection   │
│  ✓ Không có từ khóa nguy hiểm       │
│  ✓ Không có truy vấn lồng nhau      │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  XÁC THỰC CẤU TRÚC                  │
├─────────────────────────────────────┤
│  ✓ Cú pháp SQL hợp lệ (parse được)  │
│  ✓ Không có cột vector trong GROUP  │
│  ✓ Sử dụng aggregate đúng cách      │
│  ✓ Cú pháp JOIN chính xác           │
└─────────────────────────────────────┘
    ↓
Kết Quả: PASS → Thực Thi | FAIL → Thử Lại/Hủy
```

**Quy Tắc Xác Thực**:
- **Vi Phạm Bảo Mật**: Hủy ngay lập tức (không thử lại)
- **Lỗi Có Thể Sửa**: Kích hoạt cơ chế thử lại
- Sử dụng `sqlglot` để parse và xác thực

---

### **Bước 4: Thực Thi Truy Vấn 🗄️**

**Cơ Sở Dữ Liệu**: PostgreSQL với pgvector & fuzzystrmatch  
**Xử Lý**: Thực thi SQL đã xác thực và lấy kết quả  
**Đầu Ra**: Các hàng dưới dạng dictionary + metadata

```
SQL Đã Xác Thực
    ↓
Kết Nối Database (psycopg2)
    ├── Host: localhost:5432
    ├── Database: books_db
    └── User: bookadmin
    ↓
Thay Thế Tham Số
    ├── Thay thế thủ công cho truy vấn phức tạp
    └── Xử lý định dạng vector đúng cách
    ↓
Thực Thi Truy Vấn
    ├── cursor.execute(sql)
    └── cursor.fetchall()
    ↓
Xử Lý Kết Quả
    ├── Chuyển thành dictionary
    ├── Trích xuất tên cột
    ├── Đếm số hàng
    └── Lọc bỏ các cột embedding
    ↓
Thành Công: Kết Quả Truy Vấn
Thất Bại: Chi Tiết Lỗi → Cơ Chế Thử Lại
```

**Xử Lý Lỗi**:
```python
try:
    cursor.execute(final_sql)
    results = cursor.fetchall()
    return {
        "success": True,
        "rows": results,
        "column_names": [desc[0] for desc in cursor.description],
        "row_count": len(results)
    }
except psycopg2.Error as e:
    return {
        "success": False,
        "error": str(e),
        "sql_query": final_sql
    }
```

---

### **Bước 5: Sinh Câu Trả Lời 💬**

**Mục Đích**: Chuyển đổi kết quả SQL sang ngôn ngữ tự nhiên  
**Xử Lý**: LLM định dạng kết quả theo phong cách hội thoại  
**Đầu Ra**: Câu trả lời thân thiện với người dùng

```
Kết Quả Truy Vấn
    ↓
Định Dạng Kết Quả
    ├── Giới hạn 20 hàng đầu tiên
    ├── Loại bỏ các cột embedding
    └── Tạo cấu trúc dễ đọc
    ↓
LLM Prompt
    ├── Câu hỏi gốc của người dùng
    ├── Kết quả đã định dạng
    └── Hướng dẫn sinh ngôn ngữ tự nhiên
    ↓
Câu Trả Lời Được Sinh
    ├── Giọng điệu hội thoại
    ├── Nhấn mạnh phát hiện chính
    ├── Tóm tắt nếu có nhiều kết quả
    └── Xử lý kết quả rỗng một cách khéo léo
    ↓
Câu Trả Lời Ngôn Ngữ Tự Nhiên Cuối Cùng
```

**Ví Dụ**:
```
Truy Vấn: "Tìm sách dystopian dưới $20"
Kết Quả: Tìm thấy 3 cuốn sách

Câu Trả Lời Được Sinh:
"Tôi tìm thấy 3 cuốn sách dystopian dưới $20:
1. '1984' của George Orwell - $15.99
2. 'Brave New World' của Aldous Huxley - $14.50
3. 'Fahrenheit 451' của Ray Bradbury - $13.99

Cả ba đều là kiệt tác văn học dystopian!"
```

---

## 🔄 Cơ Chế Thử Lại (Khôi Phục Lỗi Thông Minh)

**Kích Hoạt**: Thất bại khi thực thi SQL (trừ vi phạm bảo mật)  
**Số Lần Tối Đa**: 4 lần (1 lần ban đầu + 3 lần thử lại)  
**Chiến Lược**: Gửi ngữ cảnh lỗi về LLM

```
Thực Thi SQL Thất Bại
    ↓
Thu Thập Chi Tiết Lỗi
    ├── Thông báo lỗi
    ├── Câu SQL thất bại
    ├── Tham số đã sử dụng
    └── Số lần thử
    ↓
Thêm Vào Lịch Sử Thất Bại
    ↓
Sinh Prompt Thử Lại
    ├── Câu hỏi gốc
    ├── Tất cả lần thử trước đó
    ├── Tất cả thông báo lỗi
    └── Ngữ cảnh về lỗi
    ↓
LLM Sinh Lại SQL
    ├── Học từ sai lầm
    ├── Thử cách tiếp cận khác
    └── Sửa lỗi cú pháp/logic
    ↓
Lặp Lại Bước 2-4
    ↓
Số Lần Thử < 4 → Tiếp Tục
Số Lần Thử >= 4 → Trả về thất bại với lịch sử đầy đủ
```

**Ví Dụ Prompt Thử Lại**:
```python
f"""Truy vấn SQL trước đó thất bại với lỗi: {error_message}

SQL Thất Bại:
{failed_sql}

Các lần thử trước:
{json.dumps(failed_attempts, indent=2)}

Vui lòng sinh câu truy vấn SQL đã sửa lỗi.
"""
```

---

## 🎯 Ba Chiến Lược Tìm Kiếm Chi Tiết

### **1. Bộ Lọc SQL Truyền Thống 📊**

**Tốt Nhất Cho**: Dữ liệu có cấu trúc, chính xác (giá, ngày, ID, boolean)

**Ví Dụ Truy Vấn**:
```
Người Dùng: "Sách xuất bản sau 2010 giá dưới $20"

SQL Được Sinh:
SELECT * FROM books
WHERE publication_date > '2010-01-01'
  AND retail_price < 20
ORDER BY publication_date DESC;
```

**Đặc Điểm**:
- Tra cứu nhanh có index
- So sánh chính xác
- Logic boolean phức tạp (AND/OR/NOT)
- Truy vấn khoảng số/ngày
- JOIN giữa các bảng

---

### **2. Fuzzy Matching (Levenshtein) 🔤**

**Tốt Nhất Cho**: Các trường văn bản có khả năng lỗi gõ (tên, tiêu đề, danh mục)

**Ví Dụ Truy Vấn**:
```
Người Dùng: "Sách của George Orrwell" (lỗi gõ)

SQL Được Sinh:
SELECT b.*, a.first_name, a.last_name
FROM books b
JOIN authors a ON b.author_id = a.author_id
WHERE levenshtein(LOWER(a.last_name), LOWER('Orrwell')) <= 2
ORDER BY levenshtein(LOWER(a.last_name), LOWER('Orrwell'));
```

**Cách Hoạt Động**:
- `levenshtein('Orrwell', 'Orwell')` = 1 (1 ký tự khác biệt)
- Ngưỡng: 1-3 phép chỉnh sửa ký tự
- Không phân biệt hoa thường (`LOWER()`)
- Xếp hạng theo độ tương đồng (gần nhất trước)
- Xử lý: chèn, xóa, thay thế ký tự

**Giải Thích Levenshtein Distance**:
- **Levenshtein Distance** là khoảng cách giữa hai chuỗi ký tự
- Đo số phép chỉnh sửa tối thiểu để biến chuỗi này thành chuỗi kia
- Ba loại phép chỉnh sửa: chèn, xóa, thay thế
- Càng nhỏ → hai chuỗi càng giống nhau

**Ví Dụ Khoảng Cách Levenshtein**:
- "Orwell" → "Orrwell" = 1 (chèn 1 ký tự 'r')
- "Stephen" → "Stephan" = 1 (thay thế 'e' bằng 'a')
- "Penguin" → "Penguen" = 1 (thay thế 'i' bằng 'e')
- "Science" → "Sciance" = 2 (2 lần thay thế)

**Lưu Ý Quan Trọng**:
- Chọn ngưỡng phù hợp: tên ngắn dùng ngưỡng nhỏ (1-2), tên dài có thể dùng 2-3
- Kết hợp với LOWER() để không phân biệt hoa thường
- Có thể kết hợp với LIKE hoặc embedding để nâng cao độ chính xác

---

### **3. Vector Embeddings (Tìm Kiếm Ngữ Nghĩa) 🎯**

**Tốt Nhất Cho**: Tìm kiếm khái niệm, chủ đề, tương đồng, đề xuất

**Ví Dụ Truy Vấn**:
```
Người Dùng: "Sách về chủ đề giám sát dystopian"

SQL Được Sinh:
SELECT b.*, 
       (b.description_embed <-> %s::vector) AS distance
FROM books b
WHERE (b.description_embed <-> %s::vector) < 0.5
ORDER BY distance
LIMIT 10;

Tham Số:
- %s = embedding("giám sát dystopian toàn trị kiểm soát")
```

**Cách Hoạt Động**:
1. **Text sang Vector**: "giám sát dystopian" → [0.123, -0.456, ..., 0.789] (1536 chiều)
2. **Tính Khoảng Cách**: toán tử `<->` (khoảng cách cosine trong pgvector)
3. **Ngưỡng**: < 0.5 nghĩa là tương đồng về mặt ngữ nghĩa
4. **Kết Quả**: Sách về giám sát, ngay cả khi không dùng từ chính xác đó

**Tại Sao Mạnh Mẽ**:
- Hiểu **khái niệm**, không chỉ từ khóa
- Tìm "toàn trị" khi tìm kiếm "giám sát"
- Tìm "tự do" khi tìm kiếm "quyền lợi"
- Hiểu đa ngôn ngữ (nếu embedding hỗ trợ)
- Nhận biết đồng nghĩa ("xe hơi" tìm "ô tô")

**Ví Dụ Khớp**:
```
Tìm Kiếm: "chủ đề dystopian"

Tìm Thấy:
✓ "1984" - mô tả chứa "giám sát toàn trị"
✓ "Brave New World" - mô tả chứa "xã hội kiểm soát"
✓ "Fahrenheit 451" - mô tả chứa "kiểm duyệt áp bức"

Không Tìm Thấy:
✗ Tiểu thuyết lãng mạn (tương đồng ngữ nghĩa thấp)
✗ Sách nấu ăn (không gian khái niệm hoàn toàn khác)
```

---

## 🔗 Chiến Lược Kết Hợp (Sức Mạnh Thực Sự)

**Ví Dụ: Truy Vấn Phức Tạp Đa Chiến Lược**:

```
Câu Hỏi Người Dùng:
"Tìm sách dystopian tương tự 1984 của tác giả có tên 
kết thúc bằng 'well', xuất bản sau 2000, giá $12-$18, 
với đánh giá nhắc đến tự do"

Xử Lý Hệ Thống:
┌──────────────────────────────────────────────────┐
│ PHÂN TÁCH CHIẾN LƯỢC                             │
├──────────────────────────────────────────────────┤
│ 1. TÌM KIẾM NGỮ NGHĨA                            │
│    - "tương tự 1984"                             │
│    → embedding("giám sát toàn trị")              │
│                                                  │
│ 2. FUZZY MATCHING                                │
│    - "tên kết thúc bằng 'well'"                  │
│    → LIKE '%well' (chấp nhận lỗi gõ)             │
│                                                  │
│ 3. BỘ LỌC SQL                                    │
│    - xuất bản sau 2000                           │
│    → publication_date > '2000-01-01'             │
│    - giá $12-$18                                 │
│    → retail_price BETWEEN 12 AND 18              │
│                                                  │
│ 4. TÌM KIẾM NGỮ NGHĨA (đánh giá)                 │
│    - "đánh giá nhắc đến tự do"                   │
│    → review_embed <-> embedding("tự do")         │
└──────────────────────────────────────────────────┘

SQL Được Sinh:
SELECT 
    b.book_id,
    b.title,
    a.first_name || ' ' || a.last_name AS author,
    b.retail_price,
    b.publication_date,
    (b.description_embed <-> %s::vector) AS desc_similarity,
    MIN(r.review_embed <-> %s::vector) AS review_similarity
FROM books b
JOIN authors a ON b.author_id = a.author_id
LEFT JOIN reviews r ON b.book_id = r.book_id
WHERE 
    -- Ngữ nghĩa: chủ đề dystopian
    (b.description_embed <-> %s::vector) < 0.5
    -- Fuzzy: pattern tên tác giả
    AND LOWER(a.last_name) LIKE '%well'
    -- SQL: lọc ngày
    AND b.publication_date > '2000-01-01'
    -- SQL: khoảng giá
    AND b.retail_price BETWEEN 12 AND 18
GROUP BY b.book_id, a.first_name, a.last_name
HAVING 
    -- Ngữ nghĩa: đánh giá nhắc tự do
    MIN(r.review_embed <-> %s::vector) < 0.6
ORDER BY desc_similarity, review_similarity
LIMIT 20;

Embeddings Được Sinh:
1. "giám sát toàn trị kiểm soát" → [...]
2. "tự do quyền lợi" → [...]
```

**Truy vấn duy nhất này**:
✅ Tìm sách tương đồng về khái niệm (embeddings)  
✅ Chấp nhận lỗi gõ trong tên tác giả (fuzzy)  
✅ Lọc theo ngày/giá chính xác (SQL)  
✅ Tìm kiếm nội dung đánh giá theo ngữ nghĩa (embeddings)  
✅ JOIN nhiều bảng  
✅ Xếp hạng theo nhiều điểm tương đồng

---

## 📊 Schema Cơ Sở Dữ Liệu

### **Các Bảng Chính**

```
authors (tác giả)
├── author_id (PK)
├── first_name, last_name
├── biography, literary_style_description
├── biography_embed (vector[1536])        ← EMBEDDING
└── literary_style_embed (vector[1536])   ← EMBEDDING

books (sách)
├── book_id (PK)
├── author_id (FK → authors)
├── publisher_id (FK → publishers)
├── title, subtitle
├── book_description
├── publication_date, retail_price
├── isbn_10, isbn_13
├── book_description_embed (vector[1536]) ← EMBEDDING
└── subtitle_embed (vector[1536])         ← EMBEDDING

publishers (nhà xuất bản)
├── publisher_id (PK)
├── publisher_name
├── country
└── publisher_description_embed (vector[1536]) ← EMBEDDING

reviews (đánh giá)
├── review_id (PK)
├── book_id (FK → books)
├── reviewer_name
├── review_text
├── rating (1-5)
└── review_text_embed (vector[1536])      ← EMBEDDING

categories (danh mục)
├── category_id (PK)
├── category_name
└── description_embed (vector[1536])      ← EMBEDDING
```

### **PostgreSQL Extensions Bắt Buộc**

```sql
-- Tìm kiếm tương đồng vector
CREATE EXTENSION IF NOT EXISTS vector;

-- Fuzzy string matching (Khoảng cách Levenshtein)
CREATE EXTENSION IF NOT EXISTS fuzzystrmatch;
```

---

## 🛠️ Các File & Component Chính

### **1. text_to_sql_agent.py** (Agent Chính)

**Class**: `AgentTextToSql`

**Các Phương Thức Chính**:
```python
generate_sql(user_request)
    → Trả về: {sql_query, need_embedding, embedding_params}

_generate_embeddings(embedding_params)
    → Gọi OpenAI API, trả về chuỗi vector

execute_query(sql_query, embeddings)
    → Thực thi SQL, trả về kết quả

generate_answer(user_request, query_results)
    → LLM chuyển đổi kết quả sang ngôn ngữ tự nhiên

process_request_with_execution(user_request)
    → PIPELINE CHÍNH: Tất cả 5 bước + logic thử lại
```

**Cấu Hình**:
- Model: `gpt-4.1` (có thể cấu hình)
- Temperature: `0.1` (thấp để nhất quán)
- Embedding Model: `text-embedding-3-small`
- Số Lần Thử Lại Tối Đa: 4 lần

---

### **2. gen_embeddings.py** (Script Thiết Lập)

**Mục Đích**: Sinh embedding cho TẤT CẢ các trường văn bản trong database

**Luồng**:
```
Khám Phá Database
    ↓
Tìm tất cả bảng có cột *_embed
    ↓
Với mỗi bảng:
    ├── Xác định trường văn bản (bỏ hậu tố "_embed")
    ├── Lấy tất cả hàng có embedding NULL
    ├── Sinh embeddings qua OpenAI
    ├── Cập nhật database với vector
    └── Theo dõi chi phí & số lượng
    ↓
Báo Cáo Tóm Tắt
    ├── Tổng số embeddings đã sinh
    ├── Chi phí ước tính
    └── Các bảng đã xử lý
```

**Tự Động Khám Phá**:
- Quét `information_schema.columns`
- Tìm các cột kết thúc bằng `_embed`
- Ánh xạ sang cột văn bản nguồn
- Xử lý tất cả bảng tự động

---

### **3. utils.py** (Tiện Ích Schema)

**Hàm Chính**: `generate_db_schema(connection)`

**Mục Đích**: Trích xuất schema database đầy đủ cho LLM prompt

**Định Dạng Đầu Ra**:
```
DATABASE SCHEMA: AUTHORS
Table: authors
COLUMNS:
  • author_id (integer) [PRIMARY KEY]
  • first_name (varchar) [NOT NULL]
  • biography_embed (vector[1536]) [EMBEDDING]

FOREIGN KEYS:
  • None

UNIQUE CONSTRAINTS:
  • email

[Lặp lại cho tất cả bảng]
```

**Được Sử Dụng Bởi**: Prompt sinh SQL để cung cấp ngữ cảnh đầy đủ cho LLM

---

### **4. prompt.py** (Kỹ Thuật Prompt)

**Các Prompt Chính**:

```python
create_text_to_sql_prompt(schema)
    → System prompt cho sinh SQL
    → Bao gồm: schema, chiến lược tìm kiếm, định dạng JSON

create_sql_retry_prompt(user_request, failed_attempts)
    → Retry prompt với ngữ cảnh lỗi
    → Giúp LLM học từ sai lầm

create_final_answer_prompt()
    → System prompt cho sinh ngôn ngữ tự nhiên
    → Giọng điệu hội thoại, hữu ích
```

**Điểm Nổi Bật Kỹ Thuật Prompt**:
- Ngữ cảnh schema chi tiết
- Hướng dẫn lựa chọn chiến lược (SQL/Fuzzy/Embeddings)
- Đặc tả định dạng đầu ra JSON
- Ràng buộc bảo mật (chỉ SELECT)
- Ví dụ cho từng chiến lược

---

### **5. main.py** (Điểm Vào)

**Các Chế Độ**:

**Chế Độ Tương Tác**:
```python
python main.py

# Nhắc người dùng nhập câu hỏi
# Hiển thị kết quả pipeline đầy đủ
# Bao gồm thông tin thử lại
# Hiển thị câu trả lời đã định dạng
```

**Ví Dụ Sử Dụng**:
```python
from text_to_sql_agent import AgentTextToSql

agent = AgentTextToSql()
result = agent.process_request_with_execution(
    "Tìm sách dystopian dưới $20"
)

print(result['final_answer'])
```

---

## 📈 Hiệu Năng & Tối Ưu Hóa

### **Theo Dõi Chi Phí Embedding**

```
Giá OpenAI (text-embedding-3-small):
- $0.00002 mỗi 1K token
- ~750 từ mỗi 1K token
- Mô tả sách trung bình: 200 từ ≈ $0.000005

Ví Dụ Database (10,000 sách):
- Tổng chi phí: ~$0.05
- Chi phí thiết lập một lần
```

### **Hiệu Năng Truy Vấn**

**Tối Ưu Vector Search**:
```sql
-- Tạo HNSW index cho tìm kiếm tương đồng nhanh
CREATE INDEX ON books 
USING hnsw (description_embed vector_cosine_ops);

-- Tăng tốc đáng kể các phép toán <->
-- Đánh đổi: Mất chút độ chính xác, tăng tốc độ rất nhiều
```

**Tối Ưu Fuzzy Matching**:
```sql
-- Tạo index cho truy vấn LIKE nhanh hơn
CREATE INDEX idx_author_lastname_lower 
ON authors (LOWER(last_name));
```

---

## 🚀 Thiết Lập & Triển Khai

### **Yêu Cầu Tiên Quyết**

1. **PostgreSQL** với các extension:
   ```sql
   CREATE EXTENSION vector;
   CREATE EXTENSION fuzzystrmatch;
   ```

2. **Python Dependencies**:
   ```
   openai
   psycopg2-binary
   python-dotenv
   sqlglot
   ```

3. **Biến Môi Trường** (`.env`):
   ```
   OPENAI_API_KEY=sk-...
   ```

### **Các Bước Khởi Tạo**

```bash
# 1. Khởi động PostgreSQL (Docker Compose)
docker-compose up -d

# 2. Khởi tạo schema database
psql -U bookadmin -d books_db -f init-db.sql

# 3. Sinh embeddings cho tất cả trường văn bản
python gen_embeddings.py

# 4. Chạy agent
python main.py
```

### **Docker Compose** (`docker-compose.yml`)

```yaml
services:
  postgres:
    image: pgvector/pgvector:pg16
    environment:
      POSTGRES_DB: books_db
      POSTGRES_USER: bookadmin
      POSTGRES_PASSWORD: bookpass123
    ports:
      - "5432:5432"
    volumes:
      - ./init-db.sql:/docker-entrypoint-initdb.d/init.sql
```

---

## 🎓 Tính Năng Nâng Cao

### **1. Hỗ Trợ Đa Ngôn Ngữ**

Hệ thống có thể xử lý truy vấn bằng bất kỳ ngôn ngữ nào (Tiếng Ý, Tây Ban Nha, v.v.):

```
Người Dùng (Tiếng Ý): "Trova libri distopici sotto 20 dollari"
LLM: Hiểu ý định, sinh SQL
Câu Trả Lời (Tiếng Ý): "Ho trovato 3 libri distopici..."
```

### **2. Xếp Hạng Tương Đồng**

Kết quả được sắp xếp theo mức độ liên quan:
```sql
ORDER BY (description_embed <-> embedding("truy vấn")) ASC
-- Khoảng cách thấp hơn = tương đồng cao hơn
```

### **3. Điểm Kết Hợp**

Kết hợp nhiều điểm tương đồng:
```sql
ORDER BY 
    0.7 * (desc_embed <-> %s) + 
    0.3 * (review_embed <-> %s)
```

### **4. Điều Chỉnh Ngưỡng**

Điều chỉnh ngưỡng tương đồng:
```sql
WHERE (description_embed <-> %s) < 0.5  -- Nghiêm ngặt
WHERE (description_embed <-> %s) < 0.7  -- Lỏng lẻo
```

---

## 🔍 Ví Dụ Trace Đầu Cuối Đến Cuối

**Câu Hỏi Người Dùng**: "Tìm sách khoa học viễn tưởng của tác giả có tên phát âm giống 'Azimov' xuất bản sau 1980"

### **Bước 1: Sinh SQL**
```json
{
  "sql_query": "SELECT b.*, a.first_name, a.last_name FROM books b JOIN authors a ON b.author_id = a.author_id JOIN categories c ON b.category_id = c.category_id WHERE LOWER(c.category_name) = 'science fiction' AND levenshtein(LOWER(a.last_name), LOWER('Azimov')) <= 2 AND b.publication_date > '1980-01-01' ORDER BY levenshtein(LOWER(a.last_name), LOWER('Azimov')), b.publication_date DESC;",
  "need_embedding": false,
  "embedding_params": []
}
```

### **Bước 2: Sinh Embedding**
- Bỏ qua (need_embedding = false)

### **Bước 3: Xác Thực**
- ✅ Bảo mật: Chỉ SELECT
- ✅ Cú pháp: SQL hợp lệ
- ✅ Cấu trúc: JOIN đúng

### **Bước 4: Thực Thi**
```sql
-- Truy vấn được thực thi tìm thấy:
-- - "Asimov" (Khoảng cách Levenshtein = 1)
-- - Sách xuất bản 1981-2024
-- - Tìm thấy 15 kết quả
```

### **Bước 5: Sinh Câu Trả Lời**
```
Tôi tìm thấy 15 cuốn sách khoa học viễn tưởng của Isaac Asimov 
(Tôi nhận thấy bạn đánh vần là 'Azimov') xuất bản sau 1980:

1. "Foundation's Edge" (1982) - $16.99
2. "Prelude to Foundation" (1988) - $18.50
...

Tất cả đều là phần của series Foundation huyền thoại của ông!
```

---

## 🎯 Những Điểm Chính Cần Nhớ

### **Tại Sao Hệ Thống Này Mang Tính Cách Mạng**

1. **Không Cần Biết SQL** - Chỉ cần ngôn ngữ tự nhiên
2. **Chấp Nhận Lỗi Gõ** - Fuzzy matching xử lý sai sót
3. **Thông Minh Về Khái Niệm** - Hiểu chủ đề, không chỉ từ khóa
4. **Sẵn Sàng Production** - Bảo mật, xác thực, xử lý lỗi
5. **Tự Phục Hồi** - Cơ chế thử lại học từ lỗi
6. **Có Thể Mở Rộng** - Hoạt động với bất kỳ schema PostgreSQL nào

### **Các Trường Hợp Sử Dụng**

- 📚 **Hệ Thống Thư Viện** - Tìm kiếm sách ngôn ngữ tự nhiên
- 🛒 **Thương Mại Điện Tử** - Khám phá sản phẩm với fuzzy matching
- 📊 **Business Intelligence** - Người dùng không kỹ thuật truy vấn dữ liệu
- 🔬 **Cơ Sở Dữ Liệu Nghiên Cứu** - Tìm kiếm bài báo/tài liệu theo ngữ nghĩa
- 📝 **Quản Lý Tài Liệu** - Truy xuất dựa trên nội dung

### **Hạn Chế**

- ❌ **Thao Tác Ghi** - Chỉ SELECT (bảo mật)
- ❌ **Phân Tích Phức Tạp** - Không tối ưu cho aggregation nặng
- ❌ **Chi Phí** - Gọi API OpenAI (LLM + embeddings)
- ❌ **Độ Trễ** - Nhiều lần gọi LLM tăng thời gian chờ

---

## 📚 Tài Nguyên Học Tập

- **YouTube Tutorial**: [Hướng Dẫn Đầy Đủ](https://youtu.be/OZ4BUW4TmsI)
- **README.md**: Tài liệu tính năng đầy đủ
- **QUESTIONS.md**: 30 test case với giải thích
- **Mermaid Workflow**: Sơ đồ luồng trực quan

---

## 🔧 Hướng Dẫn Tùy Chỉnh

### **Thêm Bảng Mới**

1. Tạo bảng với các cột `*_embed`
2. Chạy `gen_embeddings.py` (tự động khám phá)
3. Schema tự động được bao gồm trong prompt

### **Thay Đổi Ngưỡng Tìm Kiếm**

```python
# Trong prompt.py, điều chỉnh hướng dẫn:
"Sử dụng ngưỡng tương đồng < 0.5 cho khớp nghiêm ngặt"
"Sử dụng ngưỡng tương đồng < 0.7 cho khớp lỏng lẻo"
```

### **Thêm Xác Thực Tùy Chỉnh**

```python
# Trong text_to_sql_agent.py
def _custom_validation(self, sql):
    # Quy tắc tùy chỉnh của bạn
    if "your_sensitive_table" in sql.lower():
        raise ValueError("Truy cập bị từ chối")
```

---

## 💡 Lời Khuyên Thực Hành

### **Tối Ưu Hóa Temperature**

- **Sinh SQL**: Giữ temperature thấp (0.0-0.2) để đảm bảo tính nhất quán
- **Sinh Câu Trả Lời**: Có thể tăng lên (0.3-0.5) để văn bản tự nhiên hơn
- **Debugging**: Đặt 0.0 để kết quả hoàn toàn deterministic

### **Chọn Ngưỡng Levenshtein**

- **Tên ngắn (< 5 ký tự)**: Ngưỡng 1
- **Tên trung bình (5-10 ký tự)**: Ngưỡng 1-2
- **Tên dài (> 10 ký tự)**: Ngưỡng 2-3
- **Luôn test** với dữ liệu thực tế để điều chỉnh

### **Tối Ưu Hiệu Năng**

- Tạo index HNSW cho tất cả cột vector
- Tạo index cho các cột thường dùng trong WHERE
- Giới hạn kết quả trả về (LIMIT)
- Cache kết quả embedding cho các truy vấn phổ biến

---

**Phiên Bản Hệ Thống**: 1.0  
**Cập Nhật Lần Cuối**: Tháng 11, 2025  
**Cơ Sở Dữ Liệu**: PostgreSQL 16 + pgvector  
**LLM**: GPT-4.1 + text-embedding-3-small
