---
title: Kotest Transactional Rollback 설정
author: kane
date: 2024-02-19
categories: [kotlin]
tags: [kotest, transactional, rollback]
---

## Transactional Rollback

Kotest 에서 Junit 에서처럼 @Transactional 어노테이션을 선언하고 테스트 진행시 Rollback 이 안되는 이슈가 있다.

검색해보니 extensions(SpringExtension) 을 선언해줘야 한다고 해서 선언 하고 진행해봤지만 그래도 Rollback 이 안되는건 마찬가지 였고 
extensions(SpringTestExtension(SpringTestLifecycleMode.Root)) 으로 선언 해줘야 정상적으로 Rollback이 진행된다.

SpringTestLifecycleMode.Test 의 옵션도 있는데 해당 옵션으로는 Rollback 이 정상적으로 되지 않는다.

### 예시코드
![Desktop View](/assets/img/kotlin/transactional.png)

