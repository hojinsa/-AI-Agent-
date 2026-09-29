# 03. Python Collector v1 - MSSQL → PostgreSQL

## 목적
EP MSSQL View의 신규/변경 문서를 읽어 PostgreSQL에 안전하게 UPSERT한다.

## 구조
```text
collector/
├─ .env
├─ config.py
├─ mssql.py
├─ postgres.py
├─ mapper.py
├─ sync_state.py
└─ main.py
```

## 패키지
```bash
pip install pyodbc "psycopg[binary]" python-dotenv
```

## Mapping 예시
```python
import hashlib

def sha256_text(value: str) -> str:
    return hashlib.sha256((value or '').encode('utf-8')).hexdigest()

def map_approval(row):
    return {
        'source_type': 'approval',
        'source_id': str(row['DOC_ID']),
        'document_no': row.get('DOC_NO'),
        'title': row.get('TITLE') or '',
        'body_text': row.get('BODY'),
        'author_id': row.get('AUTHOR_ID'),
        'department_code': row.get('AUTHOR_DEPT_CODE'),
        'status': row.get('DOC_STATUS'),
        'created_at': row.get('CREATED_AT'),
        'approved_at': row.get('APPROVED_AT'),
        'source_updated_at': row.get('UPDATED_AT'),
        'source_url': row.get('SOURCE_URL'),
        'content_hash': sha256_text((row.get('TITLE') or '')+'
'+(row.get('BODY') or '')),
    }
```

## UPSERT 예시
```sql
INSERT INTO documents (
  source_type, source_id, document_no, title, body_text,
  author_id, department_code, status, created_at, approved_at,
  source_updated_at, source_url, content_hash
)
VALUES (
  %(source_type)s, %(source_id)s, %(document_no)s, %(title)s, %(body_text)s,
  %(author_id)s, %(department_code)s, %(status)s, %(created_at)s, %(approved_at)s,
  %(source_updated_at)s, %(source_url)s, %(content_hash)s
)
ON CONFLICT (source_type, source_id)
DO UPDATE SET
  document_no=EXCLUDED.document_no,
  title=EXCLUDED.title,
  body_text=EXCLUDED.body_text,
  author_id=EXCLUDED.author_id,
  department_code=EXCLUDED.department_code,
  status=EXCLUDED.status,
  approved_at=EXCLUDED.approved_at,
  source_updated_at=EXCLUDED.source_updated_at,
  source_url=EXCLUDED.source_url,
  content_hash=EXCLUDED.content_hash,
  synced_at=NOW();
```

## 증분 동기화
```sql
WHERE UPDATED_AT > ?
ORDER BY UPDATED_AT
```
마지막 정상 동기화 시점은 `sync_state`에 저장한다.

## 핵심 원칙
- EP 계정은 SELECT only
- Mapping 정의서를 별도 관리
- 동일 문서는 UPSERT
- `UPDATED_AT` 기반 증분수집
- `content_hash`/`file_hash`로 재처리 최소화
- 실패 실행에서 `last_sync`를 잘못 전진시키지 않음
- Credential Hard Coding 금지

## 완료 기준
- 샘플 5~10건 PostgreSQL 적재
- 재실행 시 중복 없음
- 수정된 문서만 갱신
- 실패 로그 확인 가능
