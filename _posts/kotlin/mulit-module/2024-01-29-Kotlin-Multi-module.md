---
title: Kotlin Multi module 구축하기
author: kane
date: 2024-01-29
categories: [kotlin]
tags: [kotlin, multi module]
pin: true
---

`멀티모듈` 은 여러 개의 프로젝트 모듈을 하나의 빌드로 관리하는 방법을 제공합니다. 각 모듈은 독립적으로 개발되거나 빌드될 수 있으며, 이는 프로젝트를 더 쉽게 유지보수하고 확장할 수 있도록 도와줍니다.

1. 모듈 간의 의존성 관리: 각 모듈은 필요한 의존성을 정의하고, 이를 다른 모듈에서 사용할 수 있습니다.
2. 코드의 모듈화: 비즈니스 로직이나 데이터 액세스와 같은 특정 기능을 담당하는 모듈을 만들어 코드를 더 쉽게 이해하고 관리할 수 있습니다.
3. 독립적인 빌드와 배포: 각 모듈은 독립적으로 빌드 및 배포될 수 있어, 시스템 전체의 개발 및 유지보수를 단순화합니다.

## 구조
![Desktop View](/assets/img/struktur.png){: width="700" height="400" }

하나의 spring 프로젝트에서 여러 디렉토리 구조를 가지고 서버를 실행시킬 수 있도록 구성
common 모듈은 공통적으로 사용할 목적.
modules/모듈 여러 모듈을 modules 안에 넣어두고 관리할 목적. 
(루트 디렉토리에 나열시켜도 상관 없음. 개인취향)

## 구축하기

[[Spring Initializr](https://start.spring.io/)] spring boot 프로젝트 생성
- Project : Gradle - Kotlin
- Language : Katlin
- Spring Boot : 최신버전 (SNAPSHOT 안붙은것)
- Packagin : Jar
- Java : 17
- Dependencies : 자유

### settings.gradle.kts 파일 수정하기

```kotlin
rootProject.name = "template"

//모듈 등록
include("common")
include("modules:app")

```
추가할 모듈을 등록 

##### build.gradle.kts 파일 수정하기 

```kotlin
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

plugins {
 id("org.springframework.boot") version "3.2.2"
 id("io.spring.dependency-management") version "1.1.4"
 kotlin("jvm") version "1.9.22"
 kotlin("plugin.spring") version "1.9.22"
}


java {
 sourceCompatibility = JavaVersion.VERSION_17
}

allprojects {

 group = "com"
 version = "0.0.1-SNAPSHOT"

 repositories {
  mavenCentral()
 }

 tasks.withType<KotlinCompile> {
  kotlinOptions {
   freeCompilerArgs += "-Xjsr305=strict"
   jvmTarget = "17"
  }
 }

 tasks.withType<Test> {
  useJUnitPlatform()
 }
}

subprojects {
 apply(plugin = "org.springframework.boot")
 apply(plugin = "io.spring.dependency-management")
 apply(plugin = "org.jetbrains.kotlin.plugin.spring")
 apply(plugin = "kotlin")
 apply(plugin = "kotlin-kapt")

 tasks.register("prepareKotlinBuildScriptModel"){}  // Task 'prepareKotlinBuildScriptModel' not found in project 에러 해결 명령 전체 등록

 dependencies {
  //공통으로 사용할 종속성
  implementation("com.fasterxml.jackson.module:jackson-module-kotlin")
  implementation("org.jetbrains.kotlin:kotlin-reflect")
  implementation("io.github.oshai:kotlin-logging-jvm:6.0.3") // kotlin logger

  testImplementation("io.kotest:kotest-assertions-core-jvm:5.8.0") // kotest
  testImplementation("org.springframework.boot:spring-boot-starter-test")
 }
}
```

- allprojects {} 안에는 모든 프로젝트에 적용되어야 하는 부분 
- subprojects {} 안에는 공통으로 사용될 의존성들이 적용되어야 하는 부분

##### common 모듈의 구조
![](Kotlin-Multi-module/image%202.png)
- SpringBootApplication 을 실행하지 않는 공통 모듈로 사용 ( 메인 파일 삭제 )
- 공통적으로 사용할 객체 집합용
  - TestInterface 를 다른 모듈들이 가져가 사용가능

##### common 의 build.gradle.kts 파일 수정
```kotlin
import org.springframework.boot.gradle.tasks.bundling.BootJar

val jar: Jar by tasks
val bootJar: BootJar by tasks

bootJar.enabled = false     // Spring Boot Application의 jar파일을 비활성화
jar.enabled = true

dependencies {
}
```

- bootJar.enabled = false
  - SpringBootApplication 을 사용하지 않는 모듈로 해당 jar 파일을 비활성화 하는데 사용 
  - 빌드 시간을 줄이고 불필요한 파일이 생기는것을 방지


##### 생성한 모듈의 내부 build.gradle.kts 변경

```kotlin
import org.jetbrains.kotlin.gradle.tasks.KotlinCompile

dependencies {
    implementation(project(":common"))      // common 모듈 상속

    implementation("org.springframework.boot:spring-boot-starter-web")  // app 모듈에서만 사용할 web 종속성
}

// main class 설정
springBoot{
    mainClass.set("com.app.AppApplication")
}

```

- common 공통 모듈을 상속하여 객체를 사용할 때 만 각각의 모듈의 추가해주면 된다
  - 추가 안해주면 사용 불가능
- root / build.gradle.kts 에 web 종속성을 을 추가하지 않고 각 모듈에 따로 추가 (개인취향)
  - web 종속성을 공통으로 사용하지 않을 목적 
- springBoot {} main 의 위치 
  - 없어도 돌아가며 눈으로 확인하기 좋은 목적으로 표시


[GitHub Repository](https://github.com/daeyoung-k/kotlin-multi-module-template)
Kotlin 프로젝트를 진행시 템플릿으로 사용할 목적으로 구조만 잡아둔 상태

 #kotlin #kotlin/multi_module

