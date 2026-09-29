# 08. ACL + History + Audit Log

## 목적
LLM에 문서를 전달하기 전에 권한을 강제하고, 최신/과거 문서 구분 및 감사로그를 구현한다.

## ACL
```sql
CREATE TABLE document_acl (
  id BIGSERIAL PRIMARY KEY,
  document_id BIGINT NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  subject_type VARCHAR(30) NOT NULL,
  subject_id VARCHAR(100) NOT NULL,
  permission VARCHAR(30) NOT NULL DEFAULT 'READ'
);
CREATE INDEX idx_document_acl_subject
ON document_acl(subject_type, subject_id, document_id);
```

`subject_type`: `USER`, `DEPARTMENT`, `ROLE`, `ALL`

## ACL 원칙
```text
잘못된 방식: 모든 문서 검색 → Gemma → Gemma가 권한 판단
올바른 방식: 사용자 식별 → ACL → 검색 → 허용 결과만 Gemma
```

## History Relation
```sql
CREATE TABLE document_relations (
  id BIGSERIAL PRIMARY KEY,
  document_id BIGINT NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  related_document_id BIGINT NOT NULL REFERENCES documents(id) ON DELETE CASCADE,
  relation_type VARCHAR(50) NOT NULL
);
```
관계 예: `superseded_by`, `follow_up`, `related`.

## 최신 문서 판정
```text
승인/게시 상태
→ 시행일
→ Version
→ 승인일
→ 수정일
```

## Audit Log
```sql
CREATE TABLE audit_logs (
  id BIGSERIAL PRIMARY KEY,
  user_id VARCHAR(100), department_code VARCHAR(100), source_ip VARCHAR(64),
  event_type VARCHAR(50) NOT NULL, query TEXT,
  document_id BIGINT, attachment_id BIGINT, result VARCHAR(50),
  created_at TIMESTAMP DEFAULT NOW()
);
```
이벤트 예: `SEARCH`, `DOCUMENT_VIEW`, `ATTACHMENT_VIEW`, `ACL_DENY`, `ADMIN_CHANGE`, `COLLECTOR_FAIL`.

## 완료 기준
- 권한이 다른 사용자로 동일 Query 시험
- 접근불가 문서가 Search Result와 LLM Context에 포함되지 않음
- Current/History 질의 구분
- 주요 이벤트 Audit Log 생성
