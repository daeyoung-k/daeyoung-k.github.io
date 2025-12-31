---
title: PostGis DB 스키마 기능 공유 (PostgreSql)
author: kane
date: 2025-12-05
categories:
  - Database
tags:
  - postgresql
  - postgis
render_with_liquid: false
pin: false
description:
---
## 주제
>Geometry 기능을 사용하기 위하여 PostGis 도커 이미지를 사용하여 데이터베이스를 사용하고 있습니다.  
>공간 기능은 public 스키마에 모두 설치가 되는데 제가 설정해둔 구조는 도메인 별로 스키마를 분리하여 사용하고 있기 때문에 A 스키마에는 공간 기능이 없어서 사용하는데 오류가 발생하였습니다.  
>다른 스키마들에서 공간기능을 정상적으로 사용할수 있는 방법을 정리해봅니다.  

## 내용
PostGIS 기능을 해당 스키마에 새로 설치하는 것이 아니라, 기존 `public` 스키마에 설치된 PostGIS 함수와 타입을 당신의 스키마에서도 "볼 수 있게(Visible)" 설정해야합니다.  
  
PostGIS는 데이터베이스 단위로 확장이 설치되지만, 기본적으로 설치되는 위치는 `public` 스키마입니다. 다른 스키마에서 `geometry` 타입이나 `ST_Contains` 같은 함수를 호출할 때 경로를 찾지 못해 오류가 발생하는 것입니다.  
  
`search_path` 설정 변경  
  
1. 유저(Role) 단위로 설정하기  
- app_user가 접속할 때마다 my_schema와 public을 순서대로 찾아보게 함

```sql
ALTER ROLE app_user SET search_path TO my_schema, public;
```
  
2. 데이터베이스 단위로 설정하기  

```sql
ALTER DATABASE my_database_name SET search_path TO my_schema, public;
```

3. 데이터베이스 단위로 설정 (스키마가 여러개 일때)  

```sql
-- 잘못된 예 (마지막 schema_c만 남음)
ALTER DATABASE my_db SET search_path TO schema_a, public;
ALTER DATABASE my_db SET search_path TO schema_b, public;
ALTER DATABASE my_db SET search_path TO schema_c, public;

-- 올바른 예 (한 줄에 다 적어야 함)
ALTER DATABASE my_db SET search_path TO schema_a, schema_b, schema_c, public;

-- 여기서 `"$user"`는 현재 접속한 유저 이름과 같은 스키마가 있다면 그걸 최우선으로 보라는 기본 설정인데, 같이 넣어주는 것이 관례적으로 좋습니다.
ALTER DATABASE my_database_name SET search_path TO "$user", 내_스키마1, 내_스키마2, public;
```

4. 확인  

```sql
SHOW search_path;
```
