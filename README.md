# MSSQL
***필자 사용 환경 기반***
- [습관](#습관)

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
