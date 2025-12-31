---
title: Docker PostgreSql 공간데이터 적용
author: kane
date: 2025-10-30
categories:
  - Database
tags:
  - database
  - postgresql
  - postgis
render_with_liquid: false
pin: false
description:
---
## 주제

>일반 PostgreSql에는 공간 데이터(geometry, geography)를 저장할수 있는 라이브러리가 포함되어 있지 않았습니다.  
>공간 데이터를 적용 하는 방법을 기록해 봅니다.  
>데이터베이스를 `Docker` 로 구동하기 때문에 `Docker` 활용법으로 알아보겠습니다.  

## 내용

처음으로 사용중인 데이터베이스 콘솔에서 해당 명령어를 입력해봅니다.  
오류 발생시 공간 데이터를 사용할수 있는 라이브러리가 없는것이므로 사용할수 있게 바꿔줘야 합니다.
```sql
CREATE EXTENSION IF NOT EXISTS postgis;
```

PostgreSql 에 공간데이터 라이브러리가 포함되어 있는 이미지는 `postgis/postgis` 입니다
```docker
docker pull --platform linux/amd64 postgis/postgis:18-3.6
```

- 이미지의 아케텍쳐가 `linux/amd64` 으로 되어있어 `--platform linux/amd64` 을 지정해 줍니다.
- 18-3.6 버전은 25년 9월 릴리즈 된것인데 기존의 17버전 보다 훨씬 성능이 좋아졌다고 해서 사용해 봅니다.

docker 를 실행해줍니다.
```docker
docker run -d \
  --name my-postgis-container \
  -e POSTGRES_USER=myuser \
  -e POSTGRES_PASSWORD=mypassword \
  -e POSTGRES_DB=mydatabase \
  -p 5432:5432 \
  postgis/postgis:18-3.6 
```

- `-v postgis_data:/var/lib/postgresql/data \`  이 옵션은 18버전 부터는 사용 불가

구동 후 다시 한번 명령어를 실행해 줍니다.
```docker
CREATE EXTENSION IF NOT EXISTS postgis; 
```

geography 컬럼 사용방법 예시
```sql
-- 컬럼추가
ALTER TABLE stores ADD COLUMN location GEOGRAPHY(Point, 4326);

-- 데이터 입력
-- lng(경도), lat(위도) 순으로 작성
INSERT INTO stores (name, location) 
VALUES ( '서울시청', ST_MakePoint(126.9780, 37.5665)::geography );

-- 일반 조회
select * from stores where location = ST_MakePoint(127.0276, 37.4979);

-- ST_Y, ST_X 는 geometry 전용 함수라서 cast 후 조회 
-- 위도 조회 
SELECT * FROM stores WHERE ST_Y(location::geometry) = 37.541003;  
-- 경도 조회
SELECT * FROM stores WHERE ST_X(location::geometry) = 127.086798;
```

### 좌표계 4326 사용 - Gemini 설명

>`4326`을 사용하는 이유는 그것이 **GPS가 사용하는 전 '세계 표준 좌표계'** 이기 때문입니다.  
>이 좌표계의 정식 명칭은 **WGS 84**이며, `4326`은 이 시스템을 식별하는 **EPSG 코드(ID 번호)** 입니다.  
>사용자가 개발 중인 애플리케이션(모바일 앱, 웹)에서 얻는 거의 모든 '현재 위치' 좌표가 바로 이 WGS 84 (EPSG: 4326) 형식의 **위도(Latitude)와 경도(Longitude)** 입니다.  
>따라서 **클라이언트(App)와 서버(DB)가 동일한 '언어'(좌표계)를 사용**하기 위해 `4326`을 선택하는 것이 가장 자연스럽고 효율적인 방법입니다.

### 컬럼 차이

| **비교 항목**           | **geometry (평면)**            | **geography (구체)**                      |
| ----------------------- | ------------------------------ | ----------------------------------------- |
| **관점**                | 평평한 2D 지도                 | 둥근 3D 지구                              |
| **추천 좌표계**         | 지역 좌표계 (예: EPSG:5179)    | **전 세계 표준 (EPSG:4326, WGS84)**       |
| **단위 (4326 사용 시)** | **도(Degree)** (쓸모없음)      | **미터(Meter)** (매우 유용)               |
| **계산 속도**           | 매우 빠름                      | 빠름 (약간 느림)                          |
| **"5km 이내" 검색**     | 매우 복잡함 (좌표계 변환 필요) | **매우 쉬움 (`ST_DWithin(..., 5000)`)**   |
| **사용자 케이스**       | GPS 좌표 (위도/경도)를 다룰 때 | **GPS 좌표 (위도/경도)를 다룰 때 (권장)** |

### ETC
postgresql 에서 스키마를 따로 생성후 작업중이였습니다.  
DB IDE `public` 콘솔에서 조회할때는 정상적으로 `ST_Y, ST_X` 등 함수들이 잘 작동하였는데 생성한 스키마 콘솔에서 조회시 찾을수 없는 함수라는 오류가 발생했습니다.  
해당 스키마 전용 콘솔이라 `public` 스키마 까지 검색을 하지 않아서 발행하는 이유라 아래 명령어를 해주면 편하게 조회 가능합니다.  

```sql
SET search_path = 생성한 스키마, public;
```

