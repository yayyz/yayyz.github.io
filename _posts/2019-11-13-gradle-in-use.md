---
title: "It is currently in use by another Gradle instance" 해결방법 
layout: article
sharing: true
license: true
aside:
  toc: true
show_edit_on_github: true
show_subscribe: true
pageview: true
tags: dev gradle misc
---

## 해결방법
에러로그에 찍힌 해당 .lock 파일을 직접 제거해 준다.  
log level을 `--info`로 하고 console에 살펴보면 `.lock` 파일의 절대경로를 알려준다.  
해당 파일을 제거해주고 다시 gradle 태스크를 실행하면 정상동작 하는 것을 확인 할 수 있다.  
혹시 다른 .lock 파일들도 제거하고 싶다면 다음과 같은 command로 제거하면 된다.  
```
 find ~/.gradle -type f -name "*.lock" -delete
```

### 출처
다른 .lock파일 한꺼번에 제거 :  
https://stackoverflow.com/questions/21523508/it-is-currently-in-use-by-another-gradle-instance