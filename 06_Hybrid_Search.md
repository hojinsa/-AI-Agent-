# 06. Full Text + Vector Hybrid Search

## 목적
제품명/문서번호 같은 Exact Keyword와 자연어 의미 검색을 함께 사용한다.

## Vector Search
```sql
SELECT id, document_id, attachment_id, chunk_text,
       embedding <=> %(query_vector)s AS distance
FROM document_chunks
ORDER BY embedding <=> %(query_vector)s
LIMIT 30;
```

## Full Text Search
```sql
SELECT id, document_id, attachment_id, chunk_text,
       ts_rank(search_vector, plainto_tsquery('simple', %(query)s)) AS keyword_score
FROM document_chunks
WHERE search_vector @@ plainto_tsquery('simple', %(query)s)
ORDER BY keyword_score DESC
LIMIT 30;
```

## Hybrid Merge
초기 PoC는 RRF(Reciprocal Rank Fusion) 사용 가능.
```python
def rrf(keyword_ids, vector_ids, k=60):
    scores={}
    for rank, item_id in enumerate(keyword_ids,1):
        scores[item_id]=scores.get(item_id,0)+1/(k+rank)
    for rank, item_id in enumerate(vector_ids,1):
        scores[item_id]=scores.get(item_id,0)+1/(k+rank)
    return sorted(scores.items(), key=lambda x:x[1], reverse=True)
```

## Metadata Filter
```text
is_deleted = false
status = 승인/게시 상태
ACL = 사용자 접근 가능
필요 시 is_current = true
source_type = 요청 범위
```

## Reranker
```text
Keyword Top 30 + Vector Top 30
→ Merge
→ ACL/Metadata
→ Local Reranker
→ Top 10
```

## 완료 기준
- Keyword Search 정상
- Vector Search 정상
- Hybrid 결과 생성
- 정확한 고유명사와 자연어 모두 검색 가능
- 삭제/권한없는 문서 제외
