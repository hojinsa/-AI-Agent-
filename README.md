# 사내 AI Agent 구축 매뉴얼

단계별 구현 가이드 모음이다.

## 기준 아키텍처
```text
EP MSSQL DB View
→ Python Collector
→ PostgreSQL + pgvector + Full Text Search
→ Search API
→ OpenWebUI
→ Ollama / Gemma
```

## 문서
1. EP MSSQL 연결 및 View 검증
2. PostgreSQL + pgvector 구축
3. Python Collector v1
4. 첨부파일 수집 및 Parsing
5. Chunking + BGE-M3 Embedding
6. Full Text + Vector Hybrid Search
7. Search API - FastAPI 구축
8. ACL + History + Audit Log
9. OpenWebUI + Ollama/Gemma 연동
10. 운영 및 보안 시험 매뉴얼

## 공통 원칙
- 실제 Credential은 문서에 저장하지 않는다.
- EP는 Read-only 계정을 사용한다.
- 내부 서비스는 가능한 경우 localhost Binding한다.
- 실제 View/Column Name은 EP 정의서 수령 후 확정한다.
- 운영 전 보안성 검토 및 시험을 거친다.
