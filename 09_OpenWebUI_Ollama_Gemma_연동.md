# 09. OpenWebUI + Ollama/Gemma 연동

## 목적
기존 OpenWebUI에서 Search API 검색 결과를 Gemma Context로 전달해 근거 기반 답변을 생성한다.

## 최종 흐름
```text
사용자
→ OpenWebUI
→ Search API
→ PostgreSQL/pgvector
→ 검색 Context
→ Gemma
→ 답변 + 출처
```

## Search API 호출 예시
```bash
curl -X POST http://127.0.0.1:9000/search   -H "Content-Type: application/json"   -d '{"user_id":"E10001","query":"EV-Safer 최신 정책 알려줘","top_k":8}'
```

## Gemma Context Prompt 예시
```text
당신은 사내 업무 지원 AI Agent입니다.
아래 [검색자료]는 사용자 권한을 확인한 후 사내 Search API가 검색한 문서입니다.

규칙:
1. 검색자료에 없는 사실을 사내 사실인 것처럼 추측하지 마십시오.
2. 현재 정책은 승인/시행/버전 정보가 있는 최신 자료를 우선하십시오.
3. 과거 자료와 현재 자료를 혼동하지 마십시오.
4. 문서 간 충돌 시 제목, 승인일, 버전을 명시하십시오.
5. 답변 마지막에 근거 문서를 표시하십시오.
6. 자료가 부족하면 부족하다고 답하십시오.

[검색자료]
{{SEARCH_CONTEXT}}

[사용자 질문]
{{USER_QUERY}}
```

## Ollama 노출
```text
127.0.0.1:11434
```
```bash
ss -lntp | grep 11434
```

## 출처 전달 필드
- 문서 제목
- 문서 번호
- Source Type
- 승인일
- Version
- 원문 경로
- 첨부파일명
- 검색 Chunk

## 완료 기준
- OpenWebUI → Search API → Gemma End-to-End 성공
- 답변에 근거 표시
- 근거 부족 시 추측 억제
- 권한없는 Source 미표시
