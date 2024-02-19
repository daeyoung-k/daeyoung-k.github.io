---
title: Kotest Entity OneToMany 오류
author: kane
date: 2024-02-19
categories: [kotlin]
tags: [kotest, entity, onetomany]
pin: true
---

## Entity OneToMany

oneToMany가 있는 Entity 를 사용하는 api를 서버를 킨후에  호출하면 OneToMany 컬럼의 데이터를 잘 가져 오는데… Kotest 에서 돌릴때는 null 값으로 반환되어 오류가 발생한다. 

Transactional 의 옵션과 관련이 있는것 같아 해당 파일을 확인해보았다.

### Transactional 코드
```kotlin

public @interface Transactional {
    TxType value() default Transactional.TxType.REQUIRED;

    public static enum TxType {
        REQUIRED,
        REQUIRES_NEW,
        MANDATORY,
        SUPPORTS,
        NOT_SUPPORTED,
        NEVER;

        private TxType() {
        }
    }
}
```

기본옵션으로 REQUIRED 으로 되어있다. 
해당옵션을 SUPPORTS 으로 변경해주면 정상 작동된다.


`@Transactional(Transactional.TxType.SUPPORTS)`
![Desktop View](/assets/img/kotlin/transactional_supports.png){: width="500" height="400"  }


### 종류별 설명. Spring.io 에서 한글 번역한 내용.
- REQUIRED 
  - 현재 트랜잭션을 지원하고, 존재하지 않는 경우 새 트랜잭션을 생성하세요. 동일한 이름의 EJB 트랜잭션 속성과 유사합니다. 
  - 이는 트랜잭션 주석의 기본 설정입니다. 
  
- REQUIRES_NEW
  - 새 트랜잭션을 생성하고 현재 트랜잭션이 있으면 일시 중지합니다. 동일한 이름의 EJB 트랜잭션 속성과 유사합니다.
  - **참고:** 실제 거래 중단은 모든 거래 관리자에서 기본적으로 작동하지 않습니다. 이는 특히 를 사용할 수 [JtaTransactionManager](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/jta/JtaTransactionManager.html)있도록 요구하는 에 적용됩니다 jakarta.transaction.TransactionManager(표준 Jakarta EE에서는 서버마다 다릅니다).
  
- MANDATORY
  - 현재 트랜잭션을 지원하고, 존재하지 않는 경우 예외를 발생시킵니다. 동일한 이름의 EJB 트랜잭션 속성과 유사합니다.
  
- SUPPORTS
  - 현재 트랜잭션을 지원하고, 트랜잭션이 없으면 비트랜잭션으로 실행합니다. 동일한 이름의 EJB 트랜잭션 속성과 유사합니다.
  - 참고: 트랜잭션 동기화를 사용하는 트랜잭션 관리자의 경우 SUPPORTS동기화가 적용될 트랜잭션 범위를 정의하므로 트랜잭션이 전혀 없는 것과 약간 다릅니다. 결과적으로 동일한 리소스(JDBC 연결, Hibernate 세션 등)가 지정된 전체 범위에 대해 공유됩니다. 이는 트랜잭션 관리자의 실제 동기화 구성에 따라 달라집니다.
  
- NOT_SUPPORTED
  - 비트랜잭션으로 실행하고 현재 트랜잭션이 있는 경우 일시 중지합니다. 동일한 이름의 EJB 트랜잭션 속성과 유사합니다.
  - **참고:** 실제 거래 중단은 모든 거래 관리자에서 기본적으로 작동하지 않습니다. 이는 특히 를 사용할 수 [JtaTransactionManager](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/jta/JtaTransactionManager.html)있도록 요구하는 에 적용됩니다 jakarta.transaction.TransactionManager(표준 Jakarta EE에서는 서버마다 다릅니다).
  
- NEVER
  - 비트랜잭션으로 실행하고, 트랜잭션이 존재하면 예외를 발생시킵니다. 동일한 이름의 EJB 트랜잭션 속성과 유사합니다.
