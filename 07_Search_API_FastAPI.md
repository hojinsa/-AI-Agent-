# 07. Search API - FastAPI 구축

## 목적
OpenWebUI와 PostgreSQL 사이에서 ACL, Hybrid Search, History 검색을 담당하는 내부 검색 API를 구축한다.

## 설치
```bash
mkdir -p ~/company_ai/search_api
cd ~/company_ai/search_api
python3 -m venv .venv
source .venv/bin/activate
pip install fastapi uvicorn "psycopg[binary]" pgvector pydantic-settings
```

## 구조
```text
search_api/
├─ main.py
├─ config.py
├─ database.py
├─ schemas.py
└─ services/
   ├─ acl.py
   ├─ keyword.py
   ├─ vector.py
   └─ search.py
```

## API
- `GET /health`
- `POST /search`
- `POST /search-history`

향후:
- `/get-document`
- `/get-attachments`
- `/compare-versions`

## 처리 흐름
```text
사용자 확인
→ ACL
→ Keyword Search
→ Query Embedding
→ Vector Search
→ Metadata Filter
→ Merge/Rerank
→ Top-K + Source 반환
```

## 실행
```bash
uvicorn main:app --host 127.0.0.1 --port 9000
curl http://127.0.0.1:9000/health
```

## 보안
- 127.0.0.1 Binding 우선
- Pydantic 입력 Validation
- Parameterized Query
- Error/Stack Trace 노출 방지
- ACL을 Retrieval 전에 적용
- Credential Hard Coding 금지
- 일반 사용자 Network에 9000 직접 공개 금지

## 완료 기준
- `/health`, `/search`, `/search-history` 정상
- 권한없는 문서 미반환
- 제목/원문 URL/승인일/Version 등 Source 정보 반환
- 외부 Network 직접접속 불가
