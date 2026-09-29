# 05. Chunking + BGE-M3 Embedding

## 목적
문서/첨부 Text를 검색 가능한 Chunk로 나누고 로컬 BGE-M3로 Embedding하여 pgvector에 저장한다.

## 기본 원칙
- 본문과 첨부의 출처 관계 유지
- Chunk Size/Overlap은 PoC 평가로 조정
- 제목/문단/표 구조 보존을 우선 검토
- 사원정보는 일반 Embedding 대상에서 제외

## 단순 Chunker 예시
```python
def chunk_text(text: str, chunk_size=1200, overlap=200):
    text=(text or '').strip()
    if not text: return []
    chunks=[]; start=0
    while start < len(text):
        end=min(len(text), start+chunk_size)
        chunks.append(text[start:end])
        if end == len(text): break
        start=max(0, end-overlap)
    return chunks
```

## BGE-M3 예시
```bash
pip install FlagEmbedding
```
```python
from FlagEmbedding import BGEM3FlagModel
model=BGEM3FlagModel('/models/bge-m3', use_fp16=True)

def embed_texts(texts):
    out=model.encode(
        texts, batch_size=8, max_length=8192,
        return_dense=True, return_sparse=False, return_colbert_vecs=False
    )
    return out['dense_vecs']
```
운영 중 외부 모델 다운로드가 발생하지 않도록 모델을 사전 배포한다.

## 재처리 정책
```text
content_hash/file_hash 변경 없음 → 기존 Chunk/Embedding 유지
변경 있음 → 기존 Chunk 삭제 → 새 Chunk → 새 Embedding → Commit
```

## 완료 기준
- 본문/첨부 Chunk 생성
- BGE-M3 Dense Vector 생성
- DB Vector Dimension과 모델 출력 일치
- pgvector 저장 성공
- 사원정보 Embedding 제외
