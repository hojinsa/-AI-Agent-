# 01. EP MSSQL 연결 및 View 검증

## 목적
Linux AI 서버에서 EP MSSQL의 Read-only View를 조회하고 Python Collector 개발에 필요한 View, 컬럼, Join Key, 샘플 데이터를 검증한다.

## 사전 확보 정보
- MSSQL Host/IP, Port, Database, Instance Name
- 인증 방식, Read-only 계정
- View 목록, 컬럼 정의서, 고유 Key, View 간 Join Key
- 샘플 데이터 5~10건
- 첨부파일 HREF/경로 형식
- AI 서버 Source IP 허용 및 방화벽 정책
- TLS/인증서 정책

## Python 환경
```bash
mkdir -p ~/company_ai/collector
cd ~/company_ai/collector
python3 -m venv .venv
source .venv/bin/activate
pip install pyodbc python-dotenv
odbcinst -q -d
```

## 환경변수 예시
```text
EP_DB_HOST=10.10.20.30
EP_DB_PORT=1433
EP_DB_NAME=EPDB
EP_DB_USER=AI_READONLY
EP_DB_PASSWORD=CHANGE_ME
```
실제 Credential은 Git/문서/Source Code에 저장하지 않는다.

## 연결 시험
```python
import os, pyodbc
from dotenv import load_dotenv
load_dotenv()
conn_str=(
    'DRIVER={ODBC Driver 18 for SQL Server};'
    f"SERVER={os.environ['EP_DB_HOST']},{os.environ.get('EP_DB_PORT','1433')};"
    f"DATABASE={os.environ['EP_DB_NAME']};"
    f"UID={os.environ['EP_DB_USER']};"
    f"PWD={os.environ['EP_DB_PASSWORD']};"
    'Encrypt=yes;'
)
with pyodbc.connect(conn_str, timeout=10) as conn:
    cur=conn.cursor(); cur.execute('SELECT 1'); print(cur.fetchone())
```

## View/컬럼 확인
```sql
SELECT TABLE_SCHEMA, TABLE_NAME
FROM INFORMATION_SCHEMA.VIEWS
ORDER BY TABLE_SCHEMA, TABLE_NAME;
```
```sql
SELECT COLUMN_NAME, DATA_TYPE, CHARACTER_MAXIMUM_LENGTH, IS_NULLABLE
FROM INFORMATION_SCHEMA.COLUMNS
WHERE TABLE_NAME='VW_AI_APPROVAL_DOCUMENT'
ORDER BY ORDINAL_POSITION;
```
```sql
SELECT TOP 10 * FROM VW_AI_APPROVAL_DOCUMENT;
```

## Join Key 예시
```text
VW_AI_APPROVAL_DOCUMENT.DOC_ID = VW_AI_APPROVAL_ATTACHMENT.DOC_ID
VW_AI_APPROVAL_DOCUMENT.DOC_ID = VW_AI_APPROVAL_LINE.DOC_ID
VW_AI_APPROVAL_DOCUMENT.AUTHOR_ID = VW_AI_EMPLOYEE.EMP_ID
```

## 완료 기준
- Linux AI 서버에서 MSSQL 연결 성공
- Read-only 계정으로 대상 View SELECT 성공
- View 목록/컬럼/Join Key 확인
- 샘플 5~10건 검증
- 첨부 HREF 형식 확인
- DB/첨부 경로 접근과 방화벽 정책 확인
