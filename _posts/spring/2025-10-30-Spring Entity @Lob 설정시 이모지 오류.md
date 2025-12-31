---
title: Spring Entity @Lob 설정시 이모지 오류
author: kane
date: 2025-10-30
categories:
  - SpringBoot
tags:
  - entity
render_with_liquid: false
pin: false
description:
---
## 주제

>content 컬럼에 이모지 데이터를 입력하려고 하는데 @Lob 옵션 때문에 오류 발생

## 내용

```kotlin
@Lob
@Column(columnDefinition = "text", nullable = false)  
val content: String,
```

- @Lob - JPA가 이 필드를 **대용량 데이터(LOB)** 로 인식하게 함 (text, clob, blob 등)

content 컬럼은 게시글의 내용 부분이라 대용량 설정을 하고싶어서 해당 어노테이션을 사용했는데  
이모지 입력시 이상한 20004 같은 숫자로 변환되는 현상이 생겼습니다.

### ChatGPT
Hibernate가 @Lob이 붙은 문자열을 **CLOB(Character Large Object)** 로 처리하기 때문이에요.  
PostgreSQL에서는 text 컬럼을 CLOB처럼 다룰 때, **JDBC 드라이버가 문자 스트림으로 변환**하면서 UTF-8 인코딩을 제대로 유지하지 못하는 경우가 있습니다.  
결과적으로 이모지(UTF-8 4바이트 문자)가 손실되거나 깨져서 **숫자나 이상한 값**으로 보이게 됩니다.  

| **상황**  | **Hibernate 동작**                                                        | **결과**                  |
| --------- | ------------------------------------------------------------------------- | ------------------------- |
| @Lob 있음 | JDBC가 CLOB 스트림으로 변환 → 일부 드라이버가 4-byte UTF-8 문자 처리 실패 | 이모지 깨짐 / 숫자로 표시 |
| @Lob 없음 | 단순 text 컬럼으로 String을 그대로 UTF-8로 저장                           | 🐿️🦊🎚️ 정상 저장됨 ✅         |
