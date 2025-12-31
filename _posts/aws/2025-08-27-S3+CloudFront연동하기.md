---
title: S3 + CloudFront 연동하기
author: kane
date: 2025-08-27
categories:
  - AWS
tags:
  - s3
  - cloudfront
render_with_liquid: false
pin: false
description:
---
>S3 와 CloudFront 를 연동하는 방법을 알아봅니다.

개인 프로젝트 진행중 이미지 업로드 & 다운로드 기능을 구현하게 되었습니다.
S3를 저장소로 사용하고 CloudFront 의 CDN 기능을 활용하여 효율적으로 콘텐츠를 전달하는 과정을 공유합니다.

---
## CloudFront 와의 연동 이유

S3에 파일을 업로드 한 후에 파일에 직접적으로 접근하는 방법에 거부감이 들었습니다.  
그리고 여러 파일을 한번에 조회할때 속도에 대한 생각도 하면서 CloudFront를 생각하게 되었습니다.    
S3에 직접적으로 접근하게 되면 보안을 퍼블릭으로 설정하거나 프리사인드 URL을 매번 발급을 해야하고
비용 부분에서 차이가 발생합니다.

>GPT )  
>가정: 서울 리전 S3에 1TB 이미지 저장, 한국 사용자들이 접근  
>**S3 직접 URL**: 1TB × $0.114 = **$114**/월  
>**CloudFront 경유**: 1TB × $0.085 = **$85**/월  
>**약 25% 절감**  

또한 S3에 직접 요청을 하게 되면 S3의 리전 엔드포인트로 접근하고 CloudFront 는 `글로벌 엣지` 에 캐싱을 해두고 객체를 가져오기 때문에 엄청난 속도 차이가 있어 연동을 하게 되었습니다.

---
## 연동 과정

### S3 버킷 생성 후 이미지 업로드

아마존에 가입 후 S3로 이동하여 버킷 생성을 시작합니다.
![Desktop View](/assets/img/aws/S3+CloudFront연동하기/s3bucketcreate.png){: .normal}
- 모든 퍼블릭 액세스 차단 설정은 체크해줍니다. 
	CloudFront 와 연동해서 접근할거니까 퍼블릭으로 열어줄 이유가 없습니다.

생성된 버킷에 이미지 파일 하나를 업로드 해줍니다.
	폴더 생성해서 업로드 해도 무관

### CloudFront 생성

CloudFront로 이동하여 배포 생성을 시작합니다.  
1단계 - Distribution name - 배포 이름 설정 (test-cdn)  
2단계
![Desktop View](/assets/img/aws/S3+CloudFront연동하기/cloudfrontcreate.png){: .normal}
- Browse S3 를 클릭하면 방금 생성한 S3 버킷을 선택할수 있습니다.
- Origin path 는 폴더를 생성하고 파일을 업로드 하실거면 기본 경로로 설정 할 수 있습니다.

3단계 -	비활성화 선택, 단순 이미지 조회용이라 `WAF` 는 불필요하다고 생각됩니다.
4단계 -	생성하기

S3 -> 권한탭 에서 버킷 정책에 CloudFront 에 관련한 정보가 추가 되어있는지 확인합니다.  
이제 CloudFront 생성 시 자동으로 정책을 추가해줘서 더욱더 심플해졌습니다.

---
## 확인하기

배포된 CloudFront 로 이동하여 배포 도메인 이름을 확인하고 복사합니다.
![Desktop View](/assets/img/aws/S3+CloudFront연동하기/mise.png){: .normal}
정상적으로 접근이 되는걸 확인 할 수 있습니다. 

>함께 지내고 있는 미세 사진 ㅎㅎ

---
## 회고

단순히 업로드된 이미지를 보여주기 위한 작업이었지만, 그 뒤에 따라오는 요소들이 생각보다 많다는 걸 알게 되었습니다.  
그 과정에서 특히 **S3 단독 접근과 CloudFront 경유 접근의 차이**를 깊게 이해할 수 있었던 점이 가장 큰 수확이었습니다.  

---
