# 02. PostgreSQL + pgvector 구축

## 목적
EP 데이터를 저장하고 Metadata, Keyword, Vector 검색을 수행하는 AI 전용 Knowledge DB를 구축한다.

## 설치
```bash
sudo apt update
sudo apt install postgresql postgresql-contrib
psql --version
sudo systemctl status postgresql
apt-cache search pgvector
```
PostgreSQL Major Version에 맞는 pgvector 패키지를 설치한다.

## DB/Extension
```sql
CREATE USER ai_user WITH PASSWORD 'CHANGE_ME';
CREATE DATABASE company_ai OWNER ai_user ENCODING 'UTF8';
\c company_ai
CREATE EXTENSION IF NOT EXISTS vector;
CREATE EXTENSION IF NOT EXISTS pg_trgm;
```

## 기본 테이블
```sql
CREATE TABLE employees (
  employee_id VARCHAR(100) PRIMARY KEY,
  login_id VARCHAR(100), employee_name VARCHAR(100), email VARCHAR(255),
  department_code VARCHAR(100), department_name VARCHAR(200),
  position_code VARCHAR(100), position_name VARCHAR(100),
  employee_status VARCHAR(30), source_updated_at TIMESTAMP,
  synced_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE documents (
  id BIGSERIAL PRIMARY KEY,
  source_type VARCHAR(30) NOT NULL,
  source_id VARCHAR(100) NOT NULL,
  document_no VARCHAR(100), title TEXT NOT NULL, body_text TEXT,
  author_id VARCHAR(100), department_code VARCHAR(100), status VARCHAR(50),
  version VARCHAR(50), created_at TIMESTAMP, approved_at TIMESTAMP,
  source_updated_at TIMESTAMP, effective_from TIMESTAMP, effective_to TIMESTAMP,
  is_current BOOLEAN DEFAULT TRUE, is_deleted BOOLEAN DEFAULT FALSE,
  source_url TEXT, content_hash VARCHAR(64), synced_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(source_type, source_id)
);

CREATE TABLE attachments (
  id BIGSERIAL PRIMARY KEY,
  document_id BIGINT NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  source_attachment_id VARCHAR(100), file_name TEXT NOT NULL,
  file_type VARCHAR(30), source_url TEXT, local_path TEXT,
  extracted_text TEXT, file_hash VARCHAR(64),
  source_updated_at TIMESTAMP, synced_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE sync_state (
  source_name VARCHAR(100) PRIMARY KEY,
  last_sync_at TIMESTAMP, last_success_at TIMESTAMP,
  status VARCHAR(30), message TEXT
);
```

BGE-M3 Dense output dimension을 실제 모델에서 확인 후:
```sql
CREATE TABLE document_chunks (
  id BIGSERIAL PRIMARY KEY,
  document_id BIGINT NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  attachment_id BIGINT REFERENCES attachments(id) ON DELETE CASCADE,
  chunk_index INTEGER NOT NULL, chunk_text TEXT NOT NULL,
  embedding VECTOR(1024), created_at TIMESTAMP DEFAULT NOW()
);
```

추가 테이블:
- `approval_lines`
- `document_acl`
- `document_relations`
- `audit_logs`

## Full Text Search
```sql
ALTER TABLE document_chunks
ADD COLUMN search_vector tsvector
GENERATED ALWAYS AS (to_tsvector('simple', coalesce(chunk_text,''))) STORED;
CREATE INDEX idx_chunks_fts ON document_chunks USING GIN(search_vector);
CREATE INDEX idx_documents_title_trgm ON documents USING GIN(title gin_trgm_ops);
```

## Vector Index
```sql
CREATE INDEX idx_chunks_embedding_hnsw
ON document_chunks USING hnsw (embedding vector_cosine_ops);
```
초기 PoC에서는 Index 없이 검증 후 문서량 증가 시 적용 가능.

## localhost 제한
`postgresql.conf`:
```text
listen_addresses = 'localhost'
```
```bash
sudo systemctl restart postgresql
ss -lntp | grep 5432
```

## 완료 기준
- `company_ai` DB 생성
- `vector`, `pg_trgm` 활성화
- 기본 Schema 생성
- Python 접속 성공
- Vector 저장/검색 성공
- 허용된 Interface에만 Listening
