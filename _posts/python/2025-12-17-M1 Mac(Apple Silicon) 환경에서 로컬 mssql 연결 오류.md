---
title: M1 Mac(Apple Silicon) 환경에서 로컬 mssql 연결 오류
author: kane
date: 2025-12-17
categories:
  - Database
tags:
  - python
  - mssql
render_with_liquid: false
pin: false
description:
---
## 주제

>M1 Mac(Apple Silicon) 환경 파이썬 프로젝트 로컬에서 pymssql 을 사용하여 mssql 으로 연결 시도시 발생하는 오류에 대해서 알아봅니다. 

## 내용

파이썬 환경, 로컬에서 `mssql`에 `pymssql`을 사용하여 연결하려고 하는데  
`ImportError: dlopen(~venv/lib/python3.11/site-packages/pymssql/_mssql.cpython-311-darwin.so, 0x0002): symbol not found in flat namespace '_bcp_batch'`
오류가 발생하였습니다.  
이 이유는 M1 Mac(Apple Silicon) 환경에서 `pymssql`과 `FreeTDS` 라이브러리의 링크가 꼬여서 발생하는 전형적인 문제입니다.  
  
**Intel Mac(`/usr/local/...`)과 Apple Silicon Mac(`/opt/homebrew/...`)의 Homebrew 설치 경로 차이** 때문에 Python이 컴파일된 C 라이브러리(`FreeTDS`)를 찾지 못해 발생하는 현상입니다.  
특히 `_bcp_batch` 심볼을 못 찾는 것은 `FreeTDS` 라이브러리가 제대로 로드되지 않았음을 의미합니다.

#### FreeTDS 경로 명시하여 재설치

```zsh
brew install freetds openssl
```
먼저 Homebrew를 통해 필요한 의존성 라이브러리가 설치되어 있는지 확인합니다.  

그 후 파이썬 프로젝트 터미널로 이동하여 깨진 패키기를 제거합니다.  
```zsh
pip uninstall pymssql
```
  
경로를 지정하여 pymssql 재설치 (핵심)  
```zsh
LDFLAGS="-L/opt/homebrew/opt/freetds/lib -L/opt/homebrew/opt/openssl@3/lib" \
CFLAGS="-I/opt/homebrew/opt/freetds/include -I/opt/homebrew/opt/openssl@3/include" \
pip install pymssql --no-binary :all:
```
  
`pip install pymssql`을 그냥 실행하면, PyPI에 올라와 있는 미리 빌드된 바이너리(Wheel)를 가져오거나, 로컬에서 빌드할 때 기본 경로(`/usr/local` 등)만 탐색합니다. M1은 라이브러리가 `/opt/homebrew/opt/freetds`에 숨어 있기 때문에, 이를 환경변수로 "여기에 파일이 있다"고 찝어주는 것입니다.
