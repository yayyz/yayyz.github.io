---
title: rm command not found / rm 커맨드가 없다고 나올 때 
layout: article
sharing: true
license: true
aside:
  toc: true
show_edit_on_github: true
show_subscribe: true
pageview: true
tags: server centos
---
-bash: rm: command not found 
<!--more-->

```
[ec2-user@### apps]$ rm filebeat-7.3.0-x86_64.rpm
-bash: rm: command not found
[ec2-user@### apps]$ echo $PATH
/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/opt/aws/bin:/home/ec2-user/.local/bin:/home/ec2-user/bin
```
무슨 이유에서 인지 aws ec2서버에서 rm command가 없다는 것을 알게 되었다.  
띄워진 instance의 OS는 CentOS.  
아래의 command를 실행하면 coreutils에서 빠진 것을 리스트 해준다.  
```
[ec2-user@### travis]$ rpm -V coreutils
missing     /bin/rm
```
그럼...다시 재설치 하자! yum install rm 이 커맨드는 존재하지 않는 것 같으니.  

재설치 하고 나면! 정상적으로 rm 를 실행할 수 있게 된다!

### 참고
https://www.centos.org/forums/viewtopic.php?t=48861﻿
