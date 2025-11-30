# 📚 SemanticText2SQL - Complete System Flow Documentation

## 🎯 System Overview

**SemanticText2SQL** is an advanced AI-powered natural language to SQL system that combines three complementary technologies to enable truly intelligent database querying:

1. **Traditional SQL Filters** - Precision & speed for structured data
2. **Levenshtein Fuzzy Matching** - Typo tolerance for text fields
3. **Vector Embeddings** - Semantic understanding of concepts

---

## 🏗️ System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    SEMANTIC TEXT2SQL SYSTEM                          │
├─────────────────────────────────────────────────────────────────────┤
│                                                                       │
│  User Natural Language Query (Any Language)                          │
│            ↓                                                          │
│  ┌─────────────────────────────────────────────────────────┐        │
│  │  AGENT (text_to_sql_agent.py)                            │        │
│  │  ├── SQL Generation (LLM)                                │        │
│  │  ├── Embedding Generation (OpenAI)                       │        │
│  │  ├── Query Validation (sqlglot)                          │        │
│  │  ├── Query Execution (psycopg2)                          │        │
│  │  └── Answer Generation (LLM)                             │        │
│  └─────────────────────────────────────────────────────────┘        │
│            ↓                                                          │
│  ┌─────────────────────────────────────────────────────────┐        │
│  │  POSTGRESQL DATABASE (books_db)                          │        │
│  │  ├── 5 Tables (authors, books, publishers, etc.)         │        │
│  │  ├── Text Fields + Embedding Vectors                     │        │
│  │  ├── pgvector Extension (vector similarity)              │        │
│  │  └── fuzzystrmatch Extension (Levenshtein)               │        │
│  └─────────────────────────────────────────────────────────┘        │
│            ↓                                                          │
│  Natural Language Answer with Results                                │
│                                                                       │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 🔄 Complete Pipeline Flow (5 Steps)

### **Step 1: SQL Generation 🧠**

**Input**: User's natural language question  
**Process**: LLM analyzes intent and generates SQL strategy  
**Output**: SQL query + embedding parameters (if needed)

```
User Question
    ↓
System Prompt (with full DB schema)
    ↓
LLM Analysis
    ├── Identify query type
    ├── Determine search strategy
    │   ├── SQL Filters (dates, prices, IDs)
    │   ├── Fuzzy Matching (names with typos)
    │   ├── Vector Embeddings (concepts/themes)
    │   └── Combined (multiple strategies)
    └── Generate SQL with placeholders
    ↓
JSON Response
{
  "sql_query": "SELECT ... WHERE ...",
  "need_embedding": true/false,
  "embedding_params": [
    {
      "placeholder": "embedding_1",
      "text_to_embed": "search term",
      "table_field": "books.description_embed"
    }
  ]
}
```

**Key Components**:
- `text_to_sql_agent.py::generate_sql()`
- `prompt.py::create_text_to_sql_prompt()`
- Uses GPT-4.1 by default (configurable)
- Temperature: 0.1 (for consistency)
- Response format: JSON

---

### **Step 2: Embedding Generation 🔮**

**Triggered when**: `need_embedding = true`  
**Process**: Convert search terms to 1536-dimensional vectors  
**Output**: PostgreSQL-formatted vector strings

```
Embedding Parameters
    ↓
For each parameter:
    ├── Extract text_to_embed
    ├── Call OpenAI Embedding API
    │   ├── Model: text-embedding-3-small
    │   └── Dimensions: 1536
    ├── Receive vector [0.123, -0.456, ...]
    └── Format as PostgreSQL vector: '[0.123,-0.456,...]'
    ↓
Substitute placeholders in SQL
    ↓
Final executable SQL query
```

**Implementation**:
```python
# From text_to_sql_agent.py
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

---

### **Step 3: SQL Validation 🛡️**

**Purpose**: Security and structural validation  
**Process**: Multi-layer validation using sqlglot  
**Output**: Pass/Fail with detailed error messages

```
Generated SQL Query
    ↓
┌─────────────────────────────────────┐
│  SECURITY VALIDATION                │
├─────────────────────────────────────┤
│  ✓ Only SELECT allowed              │
│  ✓ No INSERT/UPDATE/DELETE/DROP     │
│  ✓ No SQL injection patterns        │
│  ✓ No dangerous keywords            │
│  ✓ No nested queries (configurable) │
└─────────────────────────────────────┘
    ↓
┌─────────────────────────────────────┐
│  STRUCTURAL VALIDATION              │
├─────────────────────────────────────┤
│  ✓ Valid SQL syntax (parseable)     │
│  ✓ No vector columns in GROUP BY    │
│  ✓ Proper aggregate usage           │
│  ✓ Correct JOIN syntax              │
└─────────────────────────────────────┘
    ↓
Result: PASS → Execute | FAIL → Retry/Abort
```

**Validation Rules**:
- **Security Violations**: Immediate abort (no retry)
- **Fixable Errors**: Trigger retry mechanism
- Uses `sqlglot` for parsing and validation

---

### **Step 4: Query Execution 🗄️**

**Database**: PostgreSQL with pgvector & fuzzystrmatch  
**Process**: Execute validated SQL and fetch results  
**Output**: Rows as dictionaries + metadata

```
Validated SQL Query
    ↓
Database Connection (psycopg2)
    ├── Host: localhost:5432
    ├── Database: books_db
    └── User: bookadmin
    ↓
Parameter Substitution
    ├── Manual replacement for complex queries
    └── Handle vector format properly
    ↓
Query Execution
    ├── cursor.execute(sql)
    └── cursor.fetchall()
    ↓
Result Processing
    ├── Convert to dictionaries
    ├── Extract column names
    ├── Count rows
    └── Filter out embedding columns
    ↓
Success: Query Results
Failure: Error details → Retry Mechanism
```

**Error Handling**:
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

### **Step 5: Answer Generation 💬**

**Purpose**: Convert SQL results to natural language  
**Process**: LLM formats results in conversational format  
**Output**: Human-friendly answer

```
Query Results
    ↓
Format Results
    ├── Limit to first 20 rows
    ├── Remove embedding columns
    └── Create readable structure
    ↓
LLM Prompt
    ├── Original user question
    ├── Formatted results
    └── Instructions for natural language
    ↓
Generated Answer
    ├── Conversational tone
    ├── Highlight key findings
    ├── Summarize if many results
    └── Handle empty results gracefully
    ↓
Final Natural Language Response
```

**Example**:
```
Query: "Find dystopian books under $20"
Results: 3 books found

Generated Answer:
"I found 3 dystopian books under $20:
1. '1984' by George Orwell - $15.99
2. 'Brave New World' by Aldous Huxley - $14.50
3. 'Fahrenheit 451' by Ray Bradbury - $13.99

All three are classics in dystopian literature!"
```

---

## 🔄 Retry Mechanism (Intelligent Error Recovery)

**Triggers**: SQL execution failures (except security violations)  
**Max Attempts**: 4 total (1 initial + 3 retries)  
**Strategy**: Feed error context back to LLM

```
SQL Execution Failed
    ↓
Capture Error Details
    ├── Error message
    ├── Failed SQL query
    ├── Parameters used
    └── Attempt number
    ↓
Add to Failure History
    ↓
Generate Retry Prompt
    ├── Original user question
    ├── All previous attempts
    ├── All error messages
    └── Context about what went wrong
    ↓
LLM Regenerates SQL
    ├── Learn from mistakes
    ├── Try different approach
    └── Fix syntax/logic errors
    ↓
Repeat Steps 2-4
    ↓
Retry Count < 4 → Continue
Retry Count >= 4 → Return failure with full history
```

**Retry Prompt Example**:
```python
f"""The previous SQL query failed with error: {error_message}

Failed SQL:
{failed_sql}

Previous attempts:
{json.dumps(failed_attempts, indent=2)}

Please generate a corrected SQL query that fixes this error.
"""
```

---

## 🎯 Three Search Strategies in Detail

### **1. Traditional SQL Filters 📊**

**Best For**: Structured, exact data (prices, dates, IDs, boolean flags)

**Example Query**:
```
User: "Books published after 2010 priced under $20"

Generated SQL:
SELECT * FROM books
WHERE publication_date > '2010-01-01'
  AND retail_price < 20
ORDER BY publication_date DESC;
```

**Characteristics**:
- Fast indexed lookups
- Exact comparisons
- Complex boolean logic (AND/OR/NOT)
- Numeric/date range queries
- Joins between tables

---

### **2. Fuzzy Matching (Levenshtein) 🔤**

**Best For**: Text fields with potential typos (names, titles, categories)

**Example Query**:
```
User: "Books by George Orrwell" (typo)

Generated SQL:
SELECT b.*, a.first_name, a.last_name
FROM books b
JOIN authors a ON b.author_id = a.author_id
WHERE levenshtein(LOWER(a.last_name), LOWER('Orrwell')) <= 2
ORDER BY levenshtein(LOWER(a.last_name), LOWER('Orrwell'));
```

**How It Works**:
- `levenshtein('Orrwell', 'Orwell')` = 1 (1 character difference)
- Threshold: 1-3 character edits
- Case-insensitive (`LOWER()`)
- Ranks by similarity (closest first)
- Handles: insertions, deletions, substitutions

**Levenshtein Distance Examples**:
- "Orwell" → "Orrwell" = 1 (insertion)
- "Stephen" → "Stephan" = 1 (substitution)
- "Penguin" → "Penguen" = 1 (substitution)
- "Science" → "Sciance" = 2 (2 substitutions)

---

### **3. Vector Embeddings (Semantic Search) 🎯**

**Best For**: Conceptual searches, themes, similarity, recommendations

**Example Query**:
```
User: "Books about dystopian surveillance themes"

Generated SQL:
SELECT b.*, 
       (b.description_embed <-> %s::vector) AS distance
FROM books b
WHERE (b.description_embed <-> %s::vector) < 0.5
ORDER BY distance
LIMIT 10;

Parameters:
- %s = embedding("dystopian surveillance totalitarian control")
```

**How It Works**:
1. **Text to Vector**: "dystopian surveillance" → [0.123, -0.456, ..., 0.789] (1536 dimensions)
2. **Distance Calculation**: `<->` operator (cosine distance in pgvector)
3. **Threshold**: < 0.5 means semantically similar
4. **Results**: Books about surveillance, even if they don't use those exact words

**Why It's Powerful**:
- Understands **concepts**, not just keywords
- Finds "totalitarianism" when searching "surveillance"
- Finds "liberty" when searching "freedom"
- Cross-language understanding (if embeddings support it)
- Synonym-aware ("car" finds "automobile")

**Example Matches**:
```
Search: "dystopian themes"

Finds:
✓ "1984" - description contains "totalitarian surveillance"
✓ "Brave New World" - description contains "controlled society"
✓ "Fahrenheit 451" - description contains "censorship oppression"

Doesn't Find:
✗ Romance novels (low semantic similarity)
✗ Cookbooks (completely different concept space)
```

---

## 🔗 Combined Strategy (The Real Power)

**Example: Complex Multi-Strategy Query**:

```
User Question:
"Find dystopian books similar to 1984 by authors with names 
ending in 'well', published after 2000, priced $12-$18, 
with reviews mentioning freedom"

System Processing:
┌──────────────────────────────────────────────────┐
│ STRATEGY DECOMPOSITION                           │
├──────────────────────────────────────────────────┤
│ 1. SEMANTIC SEARCH                               │
│    - "similar to 1984"                           │
│    → embedding("totalitarian surveillance")      │
│                                                  │
│ 2. FUZZY MATCHING                                │
│    - "names ending in 'well'"                    │
│    → LIKE '%well' (tolerates typos)              │
│                                                  │
│ 3. SQL FILTERS                                   │
│    - published after 2000                        │
│    → publication_date > '2000-01-01'             │
│    - priced $12-$18                              │
│    → retail_price BETWEEN 12 AND 18              │
│                                                  │
│ 4. SEMANTIC SEARCH (reviews)                     │
│    - "reviews mentioning freedom"                │
│    → review_embed <-> embedding("freedom")       │
└──────────────────────────────────────────────────┘

Generated SQL:
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
    -- Semantic: dystopian theme
    (b.description_embed <-> %s::vector) < 0.5
    -- Fuzzy: author name pattern
    AND LOWER(a.last_name) LIKE '%well'
    -- SQL: date filter
    AND b.publication_date > '2000-01-01'
    -- SQL: price range
    AND b.retail_price BETWEEN 12 AND 18
GROUP BY b.book_id, a.first_name, a.last_name
HAVING 
    -- Semantic: review mentions freedom
    MIN(r.review_embed <-> %s::vector) < 0.6
ORDER BY desc_similarity, review_similarity
LIMIT 20;

Embeddings Generated:
1. "totalitarian surveillance control" → [...]
2. "freedom liberty rights" → [...]
```

**This single query**:
✅ Finds conceptually similar books (embeddings)  
✅ Tolerates typos in author names (fuzzy)  
✅ Filters by exact date/price (SQL)  
✅ Searches review content semantically (embeddings)  
✅ Joins multiple tables  
✅ Ranks by multiple similarity scores

---

## 📊 Database Schema

### **Core Tables**

```
authors
├── author_id (PK)
├── first_name, last_name
├── biography, literary_style_description
├── biography_embed (vector[1536])        ← EMBEDDING
└── literary_style_embed (vector[1536])   ← EMBEDDING

books
├── book_id (PK)
├── author_id (FK → authors)
├── publisher_id (FK → publishers)
├── title, subtitle
├── book_description
├── publication_date, retail_price
├── isbn_10, isbn_13
├── book_description_embed (vector[1536]) ← EMBEDDING
└── subtitle_embed (vector[1536])         ← EMBEDDING

publishers
├── publisher_id (PK)
├── publisher_name
├── country
└── publisher_description_embed (vector[1536]) ← EMBEDDING

reviews
├── review_id (PK)
├── book_id (FK → books)
├── reviewer_name
├── review_text
├── rating (1-5)
└── review_text_embed (vector[1536])      ← EMBEDDING

categories
├── category_id (PK)
├── category_name
└── description_embed (vector[1536])      ← EMBEDDING
```

### **PostgreSQL Extensions Required**

```sql
-- Vector similarity search
CREATE EXTENSION IF NOT EXISTS vector;

-- Fuzzy string matching (Levenshtein distance)
CREATE EXTENSION IF NOT EXISTS fuzzystrmatch;
```

---

## 🛠️ Key Files & Components

### **1. text_to_sql_agent.py** (Main Agent)

**Class**: `AgentTextToSql`

**Key Methods**:
```python
generate_sql(user_request)
    → Returns: {sql_query, need_embedding, embedding_params}

_generate_embeddings(embedding_params)
    → Calls OpenAI API, returns vector strings

execute_query(sql_query, embeddings)
    → Executes SQL, returns results

generate_answer(user_request, query_results)
    → LLM converts results to natural language

process_request_with_execution(user_request)
    → MAIN PIPELINE: All 5 steps + retry logic
```

**Configuration**:
- Model: `gpt-4.1` (configurable)
- Temperature: `0.1` (low for consistency)
- Embedding Model: `text-embedding-3-small`
- Max Retries: 4 attempts

---

### **2. gen_embeddings.py** (Setup Script)

**Purpose**: Generate embeddings for ALL text fields in database

**Flow**:
```
Database Discovery
    ↓
Find all tables with *_embed columns
    ↓
For each table:
    ├── Identify text field (remove "_embed" suffix)
    ├── Fetch all rows with NULL embeddings
    ├── Generate embeddings via OpenAI
    ├── Update database with vectors
    └── Track cost & count
    ↓
Summary Report
    ├── Total embeddings generated
    ├── Estimated cost
    └── Tables processed
```

**Auto-Discovery**:
- Scans `information_schema.columns`
- Finds columns ending in `_embed`
- Maps to source text columns
- Handles all tables automatically

---

### **3. utils.py** (Schema Utilities)

**Key Function**: `generate_db_schema(connection)`

**Purpose**: Extract complete database schema for LLM prompt

**Output Format**:
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

[Repeat for all tables]
```

**Used By**: SQL generation prompt to give LLM full context

---

### **4. prompt.py** (Prompt Engineering)

**Key Prompts**:

```python
create_text_to_sql_prompt(schema)
    → System prompt for SQL generation
    → Includes: schema, search strategies, JSON format

create_sql_retry_prompt(user_request, failed_attempts)
    → Retry prompt with error context
    → Helps LLM learn from mistakes

create_final_answer_prompt()
    → System prompt for natural language generation
    → Conversational, helpful tone
```

**Prompt Engineering Highlights**:
- Detailed schema context
- Strategy selection guidance (SQL/Fuzzy/Embeddings)
- JSON output format specification
- Security constraints (SELECT only)
- Examples of each strategy

---

### **5. main.py** (Entry Point)

**Modes**:

**Interactive Mode**:
```python
python main.py

# Prompts user for questions
# Displays complete pipeline results
# Includes retry information
# Shows formatted answers
```

**Example Usage**:
```python
from text_to_sql_agent import AgentTextToSql

agent = AgentTextToSql()
result = agent.process_request_with_execution(
    "Find dystopian books under $20"
)

print(result['final_answer'])
```

---

## 📈 Performance & Optimization

### **Embedding Cost Tracking**

```
OpenAI Pricing (text-embedding-3-small):
- $0.00002 per 1K tokens
- ~750 words per 1K tokens
- Average book description: 200 words ≈ $0.000005

Example Database (10,000 books):
- Total cost: ~$0.05
- One-time setup cost
```

### **Query Performance**

**Vector Search Optimization**:
```sql
-- Create HNSW index for fast similarity search
CREATE INDEX ON books 
USING hnsw (description_embed vector_cosine_ops);

-- Drastically speeds up <-> operations
-- Trade-off: Slight accuracy loss, huge speed gain
```

**Fuzzy Matching Optimization**:
```sql
-- Create index for faster LIKE queries
CREATE INDEX idx_author_lastname_lower 
ON authors (LOWER(last_name));
```

---

## 🧪 Testing

### **Test Questions** (`QUESTIONS.md`)

**30 comprehensive test cases covering**:

1. **Fuzzy Matching** (typos)
   - "George Orrwell" → finds "George Orwell"
   - "Ninteen Eighty For" → finds "1984"

2. **Semantic Search** (concepts)
   - "dystopian themes" → finds relevant books
   - "books about freedom" → semantic understanding

3. **SQL Filters** (exact data)
   - "books published in 2020 under $15"
   - "authors born after 1950"

4. **Multi-Join Queries**
   - Cross-table relationships
   - Complex JOINs

5. **Aggregations**
   - COUNT, AVG, SUM, GROUP BY
   - Statistical queries

6. **Combined Strategies**
   - Multiple techniques in one query
   - Real-world complexity

---

## 🚀 Setup & Deployment

### **Prerequisites**

1. **PostgreSQL** with extensions:
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

3. **Environment Variables** (`.env`):
   ```
   OPENAI_API_KEY=sk-...
   ```

### **Initialization Steps**

```bash
# 1. Start PostgreSQL (Docker Compose)
docker-compose up -d

# 2. Initialize database schema
psql -U bookadmin -d books_db -f init-db.sql

# 3. Generate embeddings for all text fields
python gen_embeddings.py

# 4. Run the agent
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

## 🎓 Advanced Features

### **1. Multi-Language Support**

The system can handle queries in any language (Italian, Spanish, etc.):

```
User (Italian): "Trova libri distopici sotto 20 dollari"
LLM: Understands intent, generates SQL
Answer (Italian): "Ho trovato 3 libri distopici..."
```

### **2. Similarity Ranking**

Results ordered by relevance:
```sql
ORDER BY (description_embed <-> embedding("query")) ASC
-- Lower distance = higher similarity
```

### **3. Hybrid Scoring**

Combine multiple similarity scores:
```sql
ORDER BY 
    0.7 * (desc_embed <-> %s) + 
    0.3 * (review_embed <-> %s)
```

### **4. Threshold Tuning**

Adjust similarity thresholds:
```sql
WHERE (description_embed <-> %s) < 0.5  -- Strict
WHERE (description_embed <-> %s) < 0.7  -- Relaxed
```

---

## 📊 Success Metrics

### **Query Success Rate**

```
Result Tracking:
- Successful queries: Count
- Failed queries: Count + error types
- Retry rate: % of queries needing retries
- Average retries per query
```

### **User Satisfaction**

```
Answer Quality:
- Relevance to question
- Accuracy of results
- Natural language quality
- Completeness
```

---

## 🔍 Example End-to-End Trace

**User Question**: "Find science fiction books by authors whose names sound like 'Azimov' published after 1980"

### **Step 1: SQL Generation**
```json
{
  "sql_query": "SELECT b.*, a.first_name, a.last_name FROM books b JOIN authors a ON b.author_id = a.author_id JOIN categories c ON b.category_id = c.category_id WHERE LOWER(c.category_name) = 'science fiction' AND levenshtein(LOWER(a.last_name), LOWER('Azimov')) <= 2 AND b.publication_date > '1980-01-01' ORDER BY levenshtein(LOWER(a.last_name), LOWER('Azimov')), b.publication_date DESC;",
  "need_embedding": false,
  "embedding_params": []
}
```

### **Step 2: Embedding Generation**
- Skipped (need_embedding = false)

### **Step 3: Validation**
- ✅ Security: SELECT only
- ✅ Syntax: Valid SQL
- ✅ Structure: Proper JOINs

### **Step 4: Execution**
```sql
-- Executed query finds:
-- - "Asimov" (Levenshtein distance = 1)
-- - Books published 1981-2024
-- - 15 results found
```

### **Step 5: Answer Generation**
```
I found 15 science fiction books by Isaac Asimov 
(I noticed you spelled it 'Azimov') published after 1980:

1. "Foundation's Edge" (1982) - $16.99
2. "Prelude to Foundation" (1988) - $18.50
...

All of these are part of his legendary Foundation series!
```

---

## 🎯 Key Takeaways

### **Why This System is Revolutionary**

1. **No SQL Knowledge Required** - Natural language only
2. **Typo Tolerant** - Fuzzy matching handles mistakes
3. **Conceptually Intelligent** - Understands themes, not just keywords
4. **Production Ready** - Security, validation, error handling
5. **Self-Healing** - Retry mechanism learns from errors
6. **Scalable** - Works with any PostgreSQL schema

### **Use Cases**

- 📚 **Library Systems** - Natural language book search
- 🛒 **E-commerce** - Product discovery with fuzzy matching
- 📊 **Business Intelligence** - Non-technical users query data
- 🔬 **Research Databases** - Semantic paper/article search
- 📝 **Document Management** - Content-based retrieval

### **Limitations**

- ❌ **Write Operations** - SELECT only (security)
- ❌ **Complex Analytics** - Not optimized for heavy aggregations
- ❌ **Cost** - OpenAI API calls (LLM + embeddings)
- ❌ **Latency** - Multiple LLM calls add delay

---

## 📚 Learning Resources

- **YouTube Tutorial**: [Full Walkthrough](https://youtu.be/OZ4BUW4TmsI)
- **README.md**: Complete feature documentation
- **QUESTIONS.md**: 30 test cases with explanations
- **Mermaid Workflow**: Visual flowchart

---

## 🔧 Customization Guide

### **Add New Tables**

1. Create table with `*_embed` columns
2. Run `gen_embeddings.py` (auto-discovers)
3. Schema automatically included in prompts

### **Change Search Thresholds**

```python
# In prompt.py, adjust instructions:
"Use similarity threshold < 0.5 for strict matching"
"Use similarity threshold < 0.7 for relaxed matching"
```

### **Add Custom Validation**

```python
# In text_to_sql_agent.py
def _custom_validation(self, sql):
    # Your custom rules
    if "your_sensitive_table" in sql.lower():
        raise ValueError("Access denied")
```

---

**System Version**: 1.0  
**Last Updated**: November 2025  
**Database**: PostgreSQL 16 + pgvector  
**LLM**: GPT-4.1 + text-embedding-3-small
