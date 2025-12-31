---
title: Kotlin PostGIS 컬럼 매핑
author: kane
date: 2025-10-30
categories:
  - SpringBoot
tags:
  - kotlin
render_with_liquid: false
pin: false
description:
---
## 주제

>Spring boot Kotlin 에서 PostGIS 컬럼 매핑하는 방법을 알아보겠습니다.

## 내용

라이브러리 설치
```gradle
// JTS (Java Topology Suite) - Point, Polygon 등 핵심 객체  
implementation("org.locationtech.jts:jts-core:1.20.0")  
// JPA가 JTS 객체를 PostGIS와 매핑  
implementation("org.hibernate.orm:hibernate-spatial:6.6.8.Final")
```
- `implementation("org.springframework.boot:spring-boot-starter-data-jpa")` 를 사용중에 hibernate-spatial 를 무작정 최신으로 설정할시 오류가 발생(현재 최신버전 7.2.0.CR1)
	- 라이브러리 내부에 `hibernate-core:6.6.8.Final` 으로 되어있습니다.
	- 해당 버전과 싱크 유지

Bean 적용
```kotlin
@Configuration  
class GeometryFactoryConfig {    
	@Bean  
	fun geometryFactory(): GeometryFactory = GeometryFactory(PrecisionModel(), 4326) 
}
```

Entity 컬럼 적용
```kotlin
@Column(columnDefinition = "geography(Point, 4326)")  
val location: Point? = null,
```
- geography 가 아닌 geometry 로 사용할땐 바꿔주면 된다.

Point 객체 생성
```kotlin
fun createLocationPointInfo(  
    longitude: Double?,  
    latitude: Double?  
): Point? {  
    return if (longitude != null && latitude != null) {  
        geometryFactory.createPoint(  
            Coordinate(longitude, latitude)  
        )  
    } else null  
}
```
