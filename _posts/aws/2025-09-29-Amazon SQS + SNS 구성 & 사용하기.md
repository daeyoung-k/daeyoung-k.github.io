---
title: Amazon SQS 구성 & 사용하기
author: kane
date: 2025-09-25
categories:
  - Backend
  - AWS
tags:
  - kotlin
  - python
  - sqs
  - sns
render_with_liquid: false
pin: false
description:
---
>Amazon SQS, SNS 를 구성해보고 사용해 봅니다.  
>SQS 와 SNS를 어느때 사용하는지 알아봅니다.

회사에서 고객 접수 API를 유지보수하다 보니 여러 어려움이 있었습니다.  
하나의 API 안에 접수, 매칭, 알림, 포인트 차감 등 다양한 로직이 한꺼번에 들어있다 보니 코드가 지나치게 비대해졌고, 어느 지점에서 오류가 발생했는지 파악하기도 힘들었습니다.  
  
이를 개선하기 위해 기능을 나누어 **접수/매칭**은 API에서 처리하고, **포인트 차감 및 알림**은 배치 작업으로 돌리는 방식으로 분리했습니다. 유지보수성은 좋아졌지만, 새로운 문제가 생겼습니다.  
- 고객은 접수 후 알림을 즉시 받지 못해 경험이 떨어졌고,    
- 짧은 시간 안에 접수가 몰리면 배치 서버에 부하가 걸리며 처리 속도가 느려지는 현상까지 발생했습니다.    
  
이처럼 **실시간성**과 **안정성**을 모두 챙길 방법을 고민하다가, 이벤트 기반 구조를 제공하는 **Amazon SQS와 SNS**를 적용해보기로 했습니다.  
이번 글에서는 SQS와 SNS를 어떤 상황에서 사용하면 좋은지, 그리고 실제로 어떻게 적용할 수 있는지 정리해보겠습니다.  

---
## Amazon SQS, SNS 란?

클라우드 환경에서 서비스가 점점 많아지고 서로 데이터를 주고받는 일이 많아지면서 **메시지 기반 아키텍처**가 중요해졌습니다.  
AWS에서는 이를 위해 두 가지 대표적인 서비스, 
- **SQS(Simple Queue Service)**
- **SNS(Simple Notification Service)**  

를 제공합니다. 이름은 비슷하지만 역할과 사용 목적은 조금 다릅니다.
### **Amazon SQS (Simple Queue Service)**
- **설명**    
    SQS는 **메시지 큐(queue)** 서비스입니다.    
    발신자가 보낸 메시지를 큐에 담아두고, 수신자는 필요할 때 큐에서 메시지를 꺼내 처리합니다.    
- **특징**    
    - **비동기 처리**: 생산자(Producer)와 소비자(Consumer)가 동시에 실행되지 않아도 됩니다.        
    - **내구성 & 확장성**: 메시지는 AWS가 관리하는 분산 시스템에 안전하게 저장됩니다.        
    - **FIFO Queue 지원**: 메시지 순서를 보장해야 할 때 사용합니다.        
    - **지연 처리 가능**: 메시지를 특정 시간 후에 처리하도록 지연시킬 수 있습니다.     
- **활용 예시**    
    - 접수 API → 매칭 서비스로 메시지 전달
	    - 메시지 발신자가 특정 수신자(서비스)를 명확히 지정할 때 사용

### **Amazon SNS (Simple Notification Service)**
- **설명**    
    SNS는 **발행/구독(Pub/Sub)** 모델을 제공하는 메시징 서비스입니다.    
    한 곳에서 메시지를 발행하면, 구독자(Subscribers)들이 동시에 메시지를 받아볼 수 있습니다.    
- **특징**    
    - **멀티 전송**: 이메일, SMS, 모바일 푸시, Lambda 함수, SQS 큐 등 다양한 채널로 메시지를 보낼 수 있습니다.        
    - **실시간 알림**: 메시지가 발행되면 구독자에게 즉시 전달됩니다.        
    - **확장성**: 하나의 토픽(Topic)에 여러 구독자를 붙여 다수에게 알림 가능 
- **활용 예시**    
    - **매칭 서비스 → 포인트 차감 서비스, 메시지 알림 서비스**        
        - 메시지 발신자가 불특정 다수의 수신자(여러 서비스)에게 동일한 데이터를 전달할 때 사용 
    - **매칭된 결과를 동시에 여러 서비스가 활용**
        - 포인트 차감 서비스는 포인트 계산            
        - 메시지 알림 서비스는 고객에게 알림 발송            
        - 다른 서비스는 통계 집계        
    - 👉 이렇게 **하나의 이벤트가 여러 서비스에서 각자 다른 역할을 수행**해야 할 때 유용

---
## **Amazon SQS (Simple Queue Service)** 생성
- aws 웹사이트 -> sqs 검색 -> 대기열 생성 (FIFO 설정 시 메세지 순서 보장 가능) 으로 간단하게 생성이 가능
![Desktop View](/assets/img/aws/AmazonSQS구성&사용하기/Pastedimage20250926161751.png)

## **Amazon SNS (Simple Notification Service)** 생성
- aws 웹사이트 -> sns 검색 -> 주제 생성 (FIFO 설정 시 메세지 순서 보장 가능) 으로 간단하게 생성이 가능  
![Desktop View](/assets/img/aws/AmazonSQS구성&사용하기/Pastedimage20250926161953.png)
- 아래 구독 생성을 클릭후 만들어둔 SQS 를 등록해주면 해당 토픽으로 발송된 메세지를 SQS로 전달해준다.
  
**구독 필터 적용**  
여러 구독이 등록되어 있고 특정 구독이 받고싶지 않은 이벤트가 있을때 아래 처럼 적용하면 수신하지 않는다.
아래 등록된 구독을 클릭 이동 후 구독 필터 정책을 추가한다.
```json
{
  "eventType": [
    {
      "anything-but": [
        "kaneEvent"
      ]
    }
  ]
}
```


---
## 사용해보기

- 접수 & 매칭서버: Django (Python)
- 포인트차감 & 메세지 알림 서버: Spring boot (Kotlin)

>두개의 다른 서버가 실시간으로 통신할 수 있다는 점에서 더더욱 매력이 느껴진다.

### Python
- boto3 사용

구성하기
```python
# sqs 설정
sqs = boto3.client(  
    "sqs",  
    aws_access_key_id="계정 액세스 키",  
    aws_secret_access_key="계정 시크릿 키",  
    region_name="생성된 리전",  
)  

# sns 설정
sns = boto3.client(  
    "sns",  
    aws_access_key_id="계정 액세스 키",  
    aws_secret_access_key="계정 시크릿 키",  
    region_name="생성된 리전",  
)
```
python은 설정하는 방법이 매우 간단하다.

**Python 발송하기**
```python
# SQS 발송
# 생성한 큐의 URL정보
queue_url = "https://sqs.ap-northeast-.amazonaws.com/~~/KaneQueue.fifo" 

# 일반 문자열 메세지 - message = "kaneMessage"
messages = {  
    "message": "kaneMessage"  
}

sqs.send_message(  
    QueueUrl=queue_url,  
    MessageBody=json.dumps(message),  
    MessageGroupId="1",  
)

# SNS 발송
# 생성한 토픽의 ARN
topic_arn = "arn:aws:sns:ap-northeast-2:~~~~:kaneTopic.fifo"
events = {  
    "eventType": {  
        "DataType": "String",  
        "StringValue": "kaneEvent123"  # 이 부분이 위에 설정한 필터와 동일하면 수신하지 않는다.
    }  
}
sns.publish(  
    TopicArn=topic_arn,  
    Message=json.dumps(messages),  
    MessageAttributes=events,  
    MessageGroupId="1",  
    Subject="subject"  
)
```

### Spring 수신하기
사용 의존성
```kotlin
implementation("io.awspring.cloud:spring-cloud-aws-starter-sqs:3.4.0")  
implementation("software.amazon.awssdk:sns:2.34.2")
```
config 설정
```kotlin
@Configuration  
class AwsConfig(  
  @Value("~~~~accessKey")  
  private val accessKey: String,  
  @Value("~~~~secretKey")  
  private val secretKey: String,  
) {  
  
  private val awsCredentials = AwsBasicCredentials.create(accessKey, secretKey)  
  
  @Bean  
  fun sqsClient(): SqsClient {  
    return SqsClient.builder()  
      .region(Region.AP_NORTHEAST_2)  
      .credentialsProvider(StaticCredentialsProvider.create(awsCredentials))  
      .build()  
  }  
  
  @Bean  
  fun snsClient(): SnsClient {  
    return SnsClient.builder()  
      .region(Region.AP_NORTHEAST_2)  
      .credentialsProvider(StaticCredentialsProvider.create(awsCredentials))  
      .build()  
  }  
  
  @Bean  
  fun sqsAsyncClient(): SqsAsyncClient {  
    return SqsAsyncClient.builder()  
      .region(Region.AP_NORTHEAST_2)  
      .credentialsProvider(StaticCredentialsProvider.create(awsCredentials))  
      .build()  
  }  
  
  @Bean  
  fun defaultSqsListenerContainerFactory(sqsAsyncClient: SqsAsyncClient): SqsMessageListenerContainerFactory<Any> {  
    return SqsMessageListenerContainerFactory.builder<Any>()  
      .sqsAsyncClient(sqsAsyncClient)  
      .configure { option ->  
        option.maxConcurrentMessages(10)  // 동시처리 갯수  
        option.maxMessagesPerPoll(10)     // 한번에 가져오는 메시지 수  
        option.pollTimeout(Duration.ofSeconds(20))  // 폴링 타임아웃  
      }  
      .build()  
  
  }  
}
```

SQS 수동 수신하기
```kotlin
// 생성한 큐 이름
val queueName = "kaneQueue.fifo"
// 직접 큐를 지정해서 수동으로 수신하고 싶을때 활용
val queue = sqsClient.receiveMessage(  
  ReceiveMessageRequest.builder().queueUrl(queueName).build()  
)
```

SNS 자동 수신하기
```kotlin
// 생성한 큐 이름
@SqsListener("KaneQueue.fifo")
fun receiveKaneTopicMessage(  
  body: String,
  @Header("eventType") eventType: String?,  
) {  
  println("KaneQueue.fifo body: $body")   
  println("KaneQueue.fifo eventType: $eventType")  
}
```
![Desktop View](/assets/img/aws/AmazonSQS구성&사용하기/Pastedimage20250926171810.png)
- 서버 구동중에 실시간으로 자동 수신 된다.

---




