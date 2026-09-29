# 04. 첨부파일 수집 및 Parsing

## 목적
Attachment View의 HREF/경로를 통해 파일을 읽고 Text를 추출해 Parent Document와 연결한다.

## 권장 컬럼
```text
ATTACH_ID
DOC_ID 또는 POST_ID
FILE_NAME
FILE_EXT
FILE_SIZE
HREF 또는 FILE_PATH
UPDATED_AT
```

## HREF 유형
- HTTP/HTTPS URL
- 상대 URL
- SMB/UNC 공유경로
- File Server Path
- 서버 Local Path

서버 Local Path만 제공되면 AI 서버에서 직접 접근이 불가능할 수 있으므로 Download Endpoint 또는 공유경로가 필요하다.

## HTTP 다운로드
```python
from pathlib import Path
import requests

ALLOWED_EXT={'.pdf','.docx','.xlsx','.pptx','.txt','.hwp','.hwpx'}

def download_http(url, file_name, out_dir='/tmp/company_ai'):
    suffix=Path(file_name).suffix.lower()
    if suffix not in ALLOWED_EXT:
        raise ValueError(f'blocked extension: {suffix}')
    out=Path(out_dir); out.mkdir(parents=True, exist_ok=True)
    target=out / Path(file_name).name
    with requests.get(url, stream=True, timeout=30) as r:
        r.raise_for_status()
        with target.open('wb') as f:
            for chunk in r.iter_content(1024*1024):
                if chunk: f.write(chunk)
    return target
```

상대 URL은 `urljoin(base_url, href)` 사용.

## Parser 후보
- PDF: PyMuPDF
- DOCX: python-docx
- XLSX: openpyxl
- PPTX: python-pptx
- TXT: Python 기본 파일 읽기
- HWP/HWPX: 실제 샘플 기준 별도 검증

## 보안
- Extension Allowlist
- MIME 확인
- 파일 크기 제한
- Path Traversal 방지
- 실행파일 차단
- Parser 오류 격리
- 필요 없으면 원본 임시파일 삭제
- 악성파일 검사 연계 검토

## 완료 기준
- DOC_ID/ATTACH_ID 관계 유지
- PDF/DOCX/XLSX/PPTX Text 추출
- Parser 실패가 전체 Collector를 중단시키지 않음
- 임시파일 정책 검증
