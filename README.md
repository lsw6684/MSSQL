# MSSQL
***필자 사용 환경 기반***
- [습관](#습관)
- [DBMS](#dbms)
    - [RAID](#raid)
- [SSMS](#ssms)
- [명령어](#명령어)
---
## 습관
SSMS에서 CRUD 시 ROLLBACK이 불가능 하기 때문에, **BEGIN TRAN**을 사용하여 하위 쿼리로 인한 데이터 손실, 오입력을 방지합니다. <br />
실행 결과가 잘못 되었다면 **ROLLBACK**을 사용하며, 옳을 결과물이 나오면 **COMMIT**을 실행합니다.
```sql
BEGIN TRAN

UPDATE  a 
SET	a.step_3_name = b.step_3_name
FROM    acccost a 
JOIN	acccost_2 b
on   	a.dstyleno = b.dstyleno     -- KEY 총 4개.
AND	a.step_1 = b.step_1
AND	a.step_2 = b.step_2
AND	a.step_3 = b.step_3
;

ROLLBACK
COMMIT
```
---
## DBMS
데이터베이스 관리 시스템(DataBase Management System)은 다수의 사용자들이 DB 내의 데이터를 동시에 접근하여 사용할 수 있도록 합니다.

### 구조
- 데이터 파일
    - 테이블
        - 필드
            - 타입
            - 길이
            - 제약조건
- 로그 파일
    - 중간 단계
        - 트랜잭션

### 관리
- 데이터 관리 : 데이터 접근(다중 접근)
- 백업/복원
- 권한 제어
- 24/365 - 이중화로 임시DB를 만들어서 Master DB 복구 시점까지 대체.

### 특징
- 실시간 접근성
    - ms
- 계속적인 변화
    - 삽입, 수정, 삭제 등
- 동시성
    - 다수의 Client가 동시에 동일 데이터에 접근
- 내용에 의한 참조
    - 실제 데이터를 가지고 검색

### 장점 - 중앙 관리
- 데이터 중복의 최소화 
- 데이터 독립성 유지
- 데이터의 공유
- 보안성/무결성/일관성 유지

### 단점
- Overhead 발생
- 운영 비용 증가
- 오류 발생 시 복구 복잡.

### OLTP와 OLAP
- OLTP : Online Transaction Processing 
    - 실시간 실행
- OLAP : Online Analytical Processing
    - 통계, 분석
    - Analysis Service / Reporting Service

### RAID
---
## SSMS
SQL Server Management Studio
- SQL Server 관리
    - 서버 설정 변경, DB 생성, 테이블 생성
    - 보안(계정 설정, 권한)
    - 백업/복원
- 개발
    - 스토어 프로시져
    - 트리거
---
## 명령어
- USE : 사용할 DB 연결
    ```sql
    USE testDb;
    ```
- GO : 배치 단위를 구분. 독립/순차적으로 실행.
    ```sql
    SELECT * FROM 테이블
    GO
    SELECT * FROM 테이블2
    GO
    ```