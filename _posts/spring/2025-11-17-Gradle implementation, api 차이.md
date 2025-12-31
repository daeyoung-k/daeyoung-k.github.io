---
title: Gradle implementation, api 차이
author: kane
date: 2025-11-17
categories:
  - SpringBoot
tags:
  - gradle
render_with_liquid: false
pin: false
description:
---
## 주제
>Spring Gradle 프로젝트 에서 dependencies 에 라이브러리를 추가할때  
>`implementation` 과 `api` 의 차이를 알아봅니다.  

## 내용
멀티모듈로 진행중인 프로젝트에서 `Common` 모듈에 hashids 라이브러리가 필요하여 추가하였습니다.  
(`Common`: 다른 모듈에서 공통적으로 사용하는 모듈)
```gradle
dependencies {  
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")  
    api("org.hashids:hashids:1.0.3")  
}
```

바로 차이를 알아보자면
- implementation - 내가 이 라이브러리를 내부적으로만 사용할 때, 외부 모듈은 이 라이브러리를 알 필요 없을 때
- api - 내가 노출하는 public API 안에 포함되는 경우, 이 모듈을 사용하는 모듈들도 함께 필요함

공통적으로 사용해야하는 라이브러리로써 `Common` 모듈에 한번만 추가해두고 다른 모듈에서 사용할수 있게 하기 위함입니다.
```kotlin
Hashids() // ⭕ 가능 (api 덕분에 노출됨)
ObjectMapper() // ❌ (jackson은 implementation 이라 노출 안 됨)
```