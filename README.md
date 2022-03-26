# MSSQL
***필자 사용 환경 기반***
- [습관](#습관)
- [DBMS](#dbms)
    - [RAID](#raid)
    - [파일그룹](#파일그룹)
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
Redundant Array of Inexpensive Disks, 여러 개의 디스크를 배열하여 속도/안정성/효율성/가용성을 향상시키는 기술입니다.
- RAID 0 <br />
Concatenate, Stripe로 총 두 가지 방법이 있습니다. **중복 블록이 존재하지 않아서** 사용 가능 용량은 모든 디스크를 합친 용량입니다. 디스크 일부에 장애가 발생하면, 복구가 어렵고 패리티를 지원하지 않는다는 단점이 있습니다. 
    - Concatenate <br />
    두 개 이상의 디스크를 **순차적으로 사용**합니다. 디스크 공간이 부족해도 데이터를 보존하며 용량 증설이 가능합니다.
        <p align="center"><img src="images/raid01.png" width="20%"></p>

    - Stripe(**RAID 0는 보통 Stripe**) <br />
    데이터를 두 개 이상의 디스크에 **분산 저장**합니다. I/O를 디스크 수만큼 분할하기 때문에 I/O속도가 향상되며, 디스크를 늘릴 시 기존 데이터는 모두 삭제 되어야 합니다.
        <p align="center"><img src="images/raid02.png" width="20%"></p>

- RAID 1(Mirror) <br />
2개 이상의 디스크로 구성되고 패리티를 사용하지 않으며, 같은 데이터를 중복 기록합니다. 
    - 장점 : 볼륨 내 디스크 중 하나만 정상이여도 데이터 보존이 가능하기 때문에 **가용성이 높고 상대적으로 복원이 간단**합니다. 
    - 단점 : **용량이 기본의 절반**으로 줄고, 미러링으로 인해 **입력 속도가 느려**집니다.
        <p align="center"><img src="images/raid1.png" width="20%"></p>


- RAID 2 <br />
RAID 0의 Stripe에서 에러 체크와 수정을 할 수 있도록 **Hamming code**를 사용합니다. 하드 디스크에서 ECC(Error Correction Code)를 지원하지 않기 때문에 ECC를 별도의 드라이브에 저장합니다. **RAID 4가 나온 이후 거의 사용되지 않습니다.**
        <p align="center"><img src="images/raid2.png" width="30%"></p>
- RAID 3 & RAID 4 <br />
RAID 0, 1의 단점을 보완하기 위해 나온 방식으로, 기본적으로 RAID 0의 Stripe로 구성되며 성능 보완과 디스크 용량을 온전히 사용하는 것을 보장합니다. 추가로 **패리티**정보를 별도 디스크에 저장합니다. RAID 3는 데이터를 **바이트 단위**로 나누어 디스크에 동등하게 분산 기록하며, RAID 4는 **블록 단위**로 나눠 기록합니다. **RAID 3는 드라이브 동기화가 필수적**이라 많이 사용되진 않습니다.
    - 단점 : 패리티 정보를 한 디스크에 저장하기 때문에 손상 시 데이터 복구가 어려울 수 있습니다.
        <p align="center"><img src="images/raid3.png" width="30%"></p>
- RAID 5 <br />
RAID 3, 4에서 별도의 패리티 정보 디스크를 사용하여 발생하는 문제점을 보완한 방식으로 정보를 Stripe로 구성된 디스크 내에서 처리합니다.
    - 장점 : 1개의 하드가 고장나도 남은 하드를 통해 데이터를 복구할 수 있으며, 성능/안전성/용량 세 부분을 모두 고려하기 때문에 실제로 많이 사용됩니다.
    - 단점 : RAID 0보다 성능이 부족합니다.
        <p align="center"><img src="images/raid5.png" width="30%"></p>
- RAID 6 <br />
RAID 5와 같지만, 다른 드라이브들 간에 분포되어 있는 2차 패리티 정보를 넣는다는 점에서 차이가 있습니다. RAID 5에서 데이터 안전성을 더욱 고려한 방식입니다.
    - 장점 : 2개의 하드에서 문제가 생겨도 복구 가능합니다.

### 파일그룹
- 파일을 관리하기 위한 논리적인 그룹
    - 데이터 파일(*.mdf, *.ndf)
- 성능향상을 위해 서로 다른 드라이브에 생성/운영합니다.
    - RAID 0 같은 효과를 보입니다.

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
- SERVERPROPERTY('collation') : 정렬 기본 값 조회
    ```sql
    SELECT SERVERPROPERTY('collation')
    -- Korean_Wansung_CI_AS : 한글 완성형 대소문자 구분 없이
    -- Korean_Wansung_CS_AS : 한글 완성형 대소문자 구분.
    ```
- DECLARE : 변수 선언
    ```sql
    DECLARE @num INT;       -- 선언, 필드 명과 구분짓기 위해 '@'를 사용
    SET     @num = 15000;   -- 초기화
    SET     @num = @num * 5;
    SELECT  @num AS '총합';       -- 출력
    ```