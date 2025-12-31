---
title: PostgreSql FK 제약조건 설정
author: kane
date: 2025-12-15
categories:
  - Database
tags:
  - postgresql
render_with_liquid: false
pin: false
description:
---
## 주제

>FK로 설정되어 있는 테이블의 update & delete 시 참조 테이블도 함께 변경되는 설정을 적용하는것을 알아봅니다.  

## 내용

회원 테이블의 이메일을 FK(외래키)로 설정하고 있는 게시글 테이블이 있습니다.  
회원의 이메일이 변경되거나 회원 삭제할때 FK로 참조하고 있는 테이블이 같이 변경되거나 삭제가 한번에 이뤄져야 하는 상황이 있습니다.  
  
테이블의 대한 외래키 조회 쿼리

```sql  
SELECT  
    tc.constraint_name,  
    kcu.column_name,  
    ccu.table_schema  AS referenced_schema,  
    ccu.table_name    AS referenced_table,  
    ccu.column_name   AS referenced_column,  
    rc.delete_rule,  
    rc.update_rule  
FROM information_schema.table_constraints tc  
JOIN information_schema.key_column_usage kcu  
  ON tc.constraint_name = kcu.constraint_name  
JOIN information_schema.referential_constraints rc  
  ON tc.constraint_name = rc.constraint_name  
JOIN information_schema.constraint_column_usage ccu  
  ON rc.unique_constraint_name = ccu.constraint_name  
WHERE tc.constraint_type = 'FOREIGN KEY'  
  AND tc.table_schema = '{사용중인 스키마}'  
  AND tc.table_name = '{조회할 테이블 명}';
  
-- constraint_name 의 조건으로 직접 조회 하는 쿼리
SELECT  
  conname,  
  pg_get_constraintdef(oid) AS def  
FROM pg_constraint  
WHERE conname = '{constraint_name~~}';

```
  
기존에 있던 외래키를 삭제 진행  
- 기존 FK 제거 

```sql 
ALTER TABLE {스키마~}.{테이블 명~}
DROP CONSTRAINT {constraint_name~~} ;
```
  
수정 옵션과 삭제 옵션 추가 진행  
- CASCADE 포함해서 FK 재생성 

```sql
ALTER TABLE {스키마~}.{테이블 명~}  
ADD CONSTRAINT {constraint_name~~}  
FOREIGN KEY ({타겟 테이블의 외래키 컬럼})  
REFERENCES {스키마~}.{테이블 명~}({타겟 테이블의 외래키 컬럼})  
ON UPDATE CASCADE  -- 업데이트 옵션) 레퍼런스 테이블의 외래키 컬럼이 변경되면 모두 적용 된다.
ON UPDATE CASCADE; -- 삭제 옵션) 레퍼런스 테이블의 로우 삭제시 관련 테이블이 삭제된다.
```
