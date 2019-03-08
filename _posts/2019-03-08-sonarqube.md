---
layout: article
title: Sonarqube 란?
tags: ci
---
Sonarqube에 대해 알아보쟈 (draft)
<!--more-->

## 소나큐브란?
CI = continuous inspection 툴. 정적분석.

### 어떤 지표들이 있는가? 
* Bug
잘못된 코드 또는 개발자의 의도대로 동작하지 않을 코드를 표시.
* Code Smells (=구린 코드)
정상동작은 하나 유지보수 하기 힘든 코드를 표시.
ex) 중복코드, 너무 복잡한 코드, unit test에 포함되지 않은 코드
* Security Vulnerability
보안에 취약한 코드를 표시.
ex) SQL Injection, hard-coded 비밀번호, 제대로 핸들링 되지 않은 에러
* Test coverage

### 품질 정책
* 품질 프로파일
    * 코드 분석 규칙의 집합
    * 각 언어마다 다른 품질 프로파일이 존재  
* 품질 게이트 
    * 프로젝트 지표들로 설정된 임계값을 조합한 것. 
    * 품질 게이트를 통과하기 위해서는 모든 임계값 조건을 통과해야함. 

### 어떤걸 분석 할 수 있는가? 
Java, Kotlin, typescirpt, js, html, css...
다양한 플러그인으로 존재. 소나큐브에서 다운로드받아서 적용하는 방식.

## 구성 
![sonarqube1.png](/image/sonarqube1.png)
### SonarQube Server
개발자, 관리자가 사용하는 웹 서버.
코드를 분석하는 Compute Engine Server. 분석된 데이터를 데이터베이스에 저장함.
Back Search를 위한 Search Server

### SonarQube Database
SonarQube의 설정값 (보안, 플러그인 세팅)
프로젝트 정적분석 결과값
RDB만 지원하며 다음과 같은 데이터베이스를 지원함
MySQL, Oracle, PostgreSQL, MS SQL

### SonarScanner
SonarQube에서 분석할 데이터를 전송하는 client.
Build 서버에 플러그인 형태로 설치하는 것이 대부분. (ex. 젠킨스)

### 버전 
Sonarqube Version 7.1 
* Korean Pack (한국어 지원 플러그인) 이 Sonarqube 7.1 버전부터 지원함.

### 적용된 플러그인 
* SonarJava
* SonarKotlin
* Korean Pack
* LDAP (NHN ENT)
* Git 

---
현재 JDK8 버전까지 지원하고있음. (최신 유료버전도 동일). 
다른 버전의 JDK를 사용하고 있다면 먼저 빌드/테스트를 한 후 결과값을 sonarqube로 전송하는 방식으로 설정. 
## 사용방법 
현재는 두가지 방법으로 사용하고 있습니다. 

### 1. master 브랜치에 merge/push 되는 경우 
1. 개발한 코드를 remote branch에 push. 
2. master branch에 push한 branch를 Pull Request를 날림. 
3. Pull Request가 master branch에 merge 되었을 때 등록된 web-hook으로 jenkins에 알림. 
4. triggered 된 jenkins job에서 master branch의 코드를 빌드 & 정적분석 
5. 정적분석 결과를 소나큐브에서 확인할 수 있음. 


### 2. Pull request 분석 
1. 개발한 코드를 remote branch에 push.
2. 지정한 branch (develop, master)에 Pull Request를 올림. 
3. web-hook으로 pull request 전용 jenkins job이 triggered 됨. 
4. 소나큐브로 변경된 파일을 정적분석하고 결과값을 jenkins에 return. 
5. 정적분석이 완료된 jenkins는 결과값을 해당 pull request에 댓글로 리포팅.

### jenkins + ansible과 함께 추가될 사항 
CI + CD를 위해 다음과 같은 구조로 동작
![sonarqube3.png](/image/sonarqube3.png)

---
### 분석결과가 있는 경우

### 품질 프로파일 통과
![image.png](/image/sonarqube4.png)

## 적용방법 
### 프로젝트 
gradle plugin 제공.

```groovy
plugins {
    id "org.sonarqube" version "2.6.2"
}

sonarqube {
        properties {
            property "sonar.exclusions", "src/main/resources-env/**"
        }
}
```
### 젠킨스
* Sonarqube Servers 설정 

### github 
web hook 등록

(정적분석 github comment를 달게 될 계정만 해당) collaborator 에 admin 권한필요 

## 실행 
### local 
```
gradle sonarqube 
```

### kotlin project 
```
clean
sonarqube
-Dsonar.projectKey=[프로젝트 고유값, 프로젝트 이름과 같은 것을 사용해도 무방]
-Dsonar.projectName=[프로젝트 이름, 소나큐브에 보여질 프로젝트명]
-Dsonar.language=kotlin
-Dsonar.host.url=[sonarqube 주소]
-Dsonar.login=[sonarqube token]
```

### Java Project 
![image.png](/image/sonarqube6.png)

## 참고
https://blog.sonarsource.com/why-you-shouldnt-use-build-breaker/

#문서화