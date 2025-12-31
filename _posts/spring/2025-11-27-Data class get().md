---
title: Data class get()
author: kane
date: 2025-11-27
categories:
  - Backend
tags:
  - kotlin
render_with_liquid: false
pin: false
description:
---
## 주제

>data class 의 get() 을 알아봅니다.

## 내용

```kotlin
data class TestDto(  
    // 내근처  
    val search1: String? = null,  
    val search2: String? = null,  
) {  
    // 객체가 생성되는 순간(Init) 딱 한 번 계산됩니다. (저장 프로퍼티)
    val isSearch = search1 != null && search2 != null  
  
    // 이 변수를 용할 때마다(Access) 매번 새로 계산합니다. (커스텀 게터)
    val isSearch get() =  search1 != null && search2 != null
}
```
