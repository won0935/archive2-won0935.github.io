---
title: "폐쇠망에서 Jenkins 업그레이드 & 플러그인 설치하기"
date: '2022-01-25'
categories: [ Monitoring ]
tags: [ Jenkins ]
---


폐쇠망에서는 어떤 프로그램이던 의존성을 가져오기가 쉽지 않다.

현재 내가 구축 중인 Grafana(모니터링 시스템) 연동으로 인해 Jenkins 버전 업그레이드 및 플러그인 설치가 필요했다.

폐쇠망 환경에서 고군분투한 경험을 적어본다.

---

## 폐쇠망에서 Jenkins 버전 업그레이드

- 기본적으로 인터넷 환경에서 원하는 Jenkins 버전과 플러그인을 세팅해놓고 통째로 들여오는 것을 추천한다.
- 각 Jenkins 버전마다 호환되는 플러그인의 버전이 다르기 때문.

### 인터넷 PC에서

1. 인터넷이 되는 컴퓨터에서 [Jenkins 홈페이지](https://www.jenkins.io/download/) 에서 맞는 버전의 Jenkins.war 파일 다운로드
2. Jenkins.war 파일을 설치할 폐쇠망으로 이동

### 폐쇠망에서 (Linux)

3. Jenkins 중지
4. /usr/lib/jenkis/ 경로에 Jenkins.war 최신버전으로 덮어씌우기
5. Jenkins 재기동

---

## 폐쇠망에서 Jenkins 플러그인 설치

### 인터넷 PC에서

> 인터넷이 되는 PC에서 세팅 후, 통째로 들여온다고 가정

1. [Jenkins 플러그인 홈페이지](https://plugins.jenkins.io/) 에서 원하는 항목을 다운 (.hpi 파일)
   ![image](https://user-images.githubusercontent.com/55419159/150921583-fec754ee-d900-4a0c-9cf8-ef3971a7842c.png)

2. /var/lib/jenkins/plugins 경로에 플러그인 파일(.hpi)을 추가
3. Jenkins 재기동
4. /var/lib/jenkins/plugins 경로에 .hpi 파일로 .jpi 파일들이 설치된 것을 확인
5. 플러그인 파일(.jpi)들을 설치할 폐쇠망으로 이동

### 폐쇠망에서 (Linux)

6. Jenkins 중지
7. /var/lib/jenkins/plugins 경로에 가져온 플러그인 파일(.jpi) 추가
8. 플러그인 파일(.jpi)을 .hpi로 확장자 변환

   > Linux 확장자 한번에 바꾸기
   > ```shell
    > ls | grep '.jpi' | cut -d . -f 1 | while read line; do mv $line.jpi $line.hpi; done
    > ```

9. Jenkins 재기동

   > 플러그인 설치가 안되는 경우
   >
   > Jenkins Repository connector Plugin Error "failed to expand tokens for [Artifact filename:ear::version]"
   >
   > Linux 권한 문제이다. Jenkins 계정에 권한을 주어 해결했다.
   > ```shell
    > sudo chown -Rh jenkins:jenkins /var/lib/jenkins/plugins
    > ```
