---
layout: article
title: logrotate으로 로그 정리하기
tags: server,linux
---

Logrotate를 사용해서 로깅이 좀 더 효율적으로 서버공간을 차지하도록 만들자
<!--more-->

개발하고 있는 서비스의 로그 삭제 기준이 정해지지 않아 서버 disk 용량을 많이 차지했다.   
최대한 가볍게 사용하려면 로그관리를 해줘야함.   
![logrotate1.jpg](/image/logrotate1.jpg)

## 어떤 프로그램들이 로그를 찍는가
개발하고 있는 서비스에서 로그를 찍는 프로그램: 
* Redis
* Nginx
* Web Application
    * logback
    * tomcat 로그

## Logrotate
Web Application 은 logback 자체 설정과 톰캣의 log rotate 설정으로 인해 서버에서 특별하게 설정해 주어야 할 것은 없다. 
반면에 Redis, Nginx에서는 해당 기능이 없기 때문에 자체적으로 rotate을 적용해주어야함. -> `logrotate`

logrotate는 시스템로그를 **자동으로 rotate, 압축, 이메일 전송**을 가능하게 한다. 설정에 따라 **매일, 매주, 매달** 주기적으로 로그 관리가 가능함.
설정파일에서 로그 주기를 설정하면 알맞는 **cron job** 디렉토리에 있는 logrotate 실행파일이 실행된다.

```
/etc/cron.hourly			// 1시간마다
/etc/cron.daily 			// 매일 
/etc/cron.weekly			// 매주 
/etc/cron.monthly			// 매달
```

## Logrotate 설정
참고: 사내 계정에서는 sudo 권한이 있는 계정으로 해당 파일을 생성/수정 하였음.

### 설정값:
* daily 
	* 매일 해당 설정을 실행 (/etc/cron.daily 의 logrotate 가 실행됨).
* missingok 
	* 설정한 로그가 없는 경우에 에러 메세지를 출력하지 않는 설정.
* copytruncate
* rotate N: 지정한 숫자대로 로그를 N번 rotate.
* dateext
	*  예전 버전의 로그파일명에 숫자를 (디폴트 설정) 붙이는 것 대신 날짜를 붙이는 설정.
	* ex) nginx.log-YYYYMMDD
* notifempty
	*  로그가 비어있으면 rotate 하지 않는 설정.
* sharedscripts
	*  prerotate, postrotate 설정의 스크립트들이 한번만 실행하게 해주는 설정.
* postrotate, endscript:
	* 파일 rotate 가 완료된 후 실행되는 스크립트\. bin/sh 로 실행.

### 적용 
1. 로그를 관리하기 원하는 프로그램의 이름으로 파일을 생성합니다.
```
sudo vim /etc/logrotate.d/[파일명]
```
2. 아래와 같은 설정을 추가합니다. (nginx 설정 기준)
```
[로그가 위치하는 경로]/*.log {
    daily
    missingok
    copytruncate
    rotate 30
    dateext
    notifempty
    sharedscripts
    postrotate
        [ ! -f [로그위치경로]/nginx.pid ] || kill -USR1 `cat [로그위치경로]nginx.pid`
    endscript
}
```
* 30일 기준으로 rotate
* 매일 한개의 파일이 생성
* 날짜가 append 된 형식으로 로그 파일 생성 
* 경로 설정 시 로그파일명을 명확하게 기입해도 됨
* 경로 설정 시 전체 디렉토리에 적용시 `**` 사용 가능 
	* ex) [로그위치경로]/**/nginx.log 

3. 변경된 값을 적용합니다.
    * -f 옵션은 해당 설정을 cron job 주기보다 (daily, monthly 등) 먼저 **당장 적용해야 할 때**, logrotate 자체에서 rotate 할 로그가 없다고 판단하여도 rotate 를 강제로 실행할 수 있는 옵션입니다.    
  
  
```  
sudo logrotate -f /etc/logrotate.d/[file_name]

//위 command가 'command not found' 라는 에러가 출력될 때 
sudo /usr/sbin/logrotate -f /etc/logrotate.d/[file_name]  
```  

### 결과 (nginx 로그)
![logrotate2.png](/image/logrotate2.png)  

## 결과
로그 정리 전   
![logrotate3.png](/image/logrotate3.png)  
삭제 후  
![logrotate4.png](/image/logrotate4.png)  
서버 디스크 사용량 변화  
![logrotate5.png](/image/logrotate5.png)  

## 참고한 사이트
* [https://unix.stackexchange.com/questions/259091/nginx-log-rotation-doesnt-seem-to-be-working-correctly](https://unix.stackexchange.com/questions/259091/nginx-log-rotation-doesnt-seem-to-be-working-correctly)
* [https://support.rackspace.com/how-to/understanding-logrotate-utility/](https://support.rackspace.com/how-to/understanding-logrotate-utility/)