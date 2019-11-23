---
title: 에러로그 리팩토링에 대한 개발자1의 의식의 흐름 
layout: article
sharing: true
license: true
aside:
  toc: true
show_edit_on_github: true
show_subscribe: true
pageview: true
tags: devlog refactoring
---
진짜 별거 아닌 작업 같지만 한번쯤은 의식의 흐름을 남겨보고 싶었습니다. 
<!--more-->

## intro?
주문, 결제 관련 개발을 하는 파트에서 일하고 있는 개발자1입니다.   
간단한 side task 정도로 기존의 에러로그 리팩토링 업무를 받아서 진행하였는데, 간단한 업무였지만 나름의 (?!) 고민을 이것저것 하였습니다.
약간 생각의 흐름대로 작성한 리팩토링 고민 입니다. 

업무를 요약하자면 다음과 같습니다.
> 결제 관련한 에러로그를 찍을 때, 주문/결제관련 request model, 결제키를 가진 model 등 개인정보에 민감한 주문자에 대한 정보를 담고있는 객체들을 전체 출력하고 있음 (전부 toString()을 사용하여 출력). 에러 debugging 시 정말 필요한 필드만 에러로그에 남기는 방향으로 리팩토링 해야함.

그리고 이것은 기존의 코드! 
```java
protected 정의한Exception loggingErrorAndWrapping정의한Exception(Exception e, 정의한결제errorCode errorCode,
                                                                   Object... args) {
    if (e instanceof 정의한Exception) {
        return (정의한Exception)e;
    } else {
        String argsStr = "";
        for (Object arg : args) {
            if (arg != null) {
                argsStr += arg.getClass().getName() + ": " + arg + "\n";
            }
        }
        log.error("============= 결제 에러로그 =============\n" + argsStr, e);
        return new 정의한Exception(errorCode);
    }
}
```

## 특징
- 주문 도메인 전체에서 사용하지 않고 결제 관련된 서비스 코드에서만 사용하는 메소드.
- 호출할 때 에러로그에 남기고 싶은 객체를 통짜로 넘김.

## toString()을 수정하면 되지 않을까?
다른 개발자분에게 의견을 드렸더니, 그것보다는 parameter로 넘기는것으로 수정해봐라~ 라고 하시니, 일단은 그 방법은 접어두었다.  
뭔가 문제가 있어서 그런거겠지 라는 생각으로.
그리고 이미 toString()을 열렬하게 사용하는 곳이 있다면?! 그걸 다 일일히 찾아서 수정할 수도 없는 노릇이고.  
다음 글들은 생각의 흐름대로 정렬한 semi-해결방안 들이다. 개발자1의 의식의 흐름으로...!

## parameter로 필요한 필드를 넘겨준다
일단은 추천받은 방법대로 해봐야 하지 않겠나. 당연히 해봤다.  
`PayErrorInfo` 라는 클래스를 생성해서 클래스명, 클래스에서 출력하고자 하는 필드로 생성.
```java
public class PayErrorInfo {
    private String className;
    private List<String> fields; 
}
```

메소드를 호출하는 부분에서는 다음과 같이 사용
```java
List<String> fields = ArrayLists.newArrayList();
fields.add(주문번호);
정의한Exception exception = loggingErrorAndWrapping정의한Exception(e, 정의한결제errorCode.ERROR_CODE, new PayErrorInfo(주문model.getClass().getName(), fields));
```
**하지만..**
기존에는 손쉽게 object를 넘기기만 하면 끝이었는데, 지금은 불필요한 코드가 늘어난 느낌. 
깔끔하게 리팩토링 하려는 나의 의도와는 전혀 다른 결과가 나옴!  
**이것보다 더 나은 방법은 없을까?**  

## Annotation 으로 해결
annotation으로 각 model, entity에서 에러로 찍고자 하는 필드에 사용하면 좀 더 깔끔하게 해결할 수 있지 않을까?  
```java
/**
 * 에러로그에 포함하고 싶은 필드
 * @author Yeji Im
 */
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface PaymentLog {}
```

@PaymentLog를 사용하는 객체
```java
public class 주문객체 {

    @PaymentLog
    private String 주문번호;
    ...
}
```

@PaymentLog 어노테이션이 있는 필드만 출력하게끔 PayErrorInfo 수정!
```java
public class PayErrorInfo {
    private Object object;
    
    public PayErrorInfo(Object object) {
        this.object = object;
    }
    
    public String toString() {
        Class<?> clazz = object.getClass();
        Map<String, String> map = Maps.newHashMap();
        for (Field field : clazz.getDeclaredFields()) {
            field.setAccessible(true);
            if (field.isAnnotationPresent(PaymentLog.class)) {
                try {
                    map.put(field.getName(), field.get(object).toString());
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                }
            }
        }
        return map.entrySet()
                  .stream()
                  .map(entry -> "[" + entry.getKey() + "=" + entry.getValue() + "]")
                  .collect(Collectors.joining(","));
    }    
}
```

메소드를 사용하는 부분! 이전보다는 간결 해졌다.
```java
throw loggingErrorAndWrapping정의한Exception(e,정의한결제errorCode.ERROR_CODE,new PayErrorInfo(결제request))
```
이렇게 사용해서 best case 에는 다음과 같이 출력 됨.
```java
[개발자1의프로젝트] : [ERROR] 2019-04-09 14:20:32,734 [main] 이런저런패키지.결제Service 
============= 결제 에러로그 =============
[주문번호=12341234]
```
- 원래 목적이었던 몇몇의(주문번호, 주문서번호)  단일필드를 성공적으로 출력!
- 직전의 방법보다 깔끔함. 

## 하지만..
이렇게 하게 된다면, 다음과 같이 개발자1의 의도와 다르게 동작할 수 있음.
- collection 인 경우
  - List<상품> 과 같은 collection 의  내부에서 @PaymentLog를 사용한다면? => 정상적으로 동작 X
  ```java
    public class 상품 {
        @PaymentLog
        private String 상품명;
    }
  ```
- field.get(object) 처럼 직접적으로 access 하는 경우에, 해당 필드의 getter가 override된 상황이라면? 원래 getter로 받으면 정상적인 값을 받지만 직접 access 하게 된다면 **전혀 다른 값을 사용하게 될 가능성**이 있음.
- reflection을 사용하였을 때 **성능**에 대한 문제 <- 대게 그렇다고 여겨지기에...(요새 java version도 마찬가지 일까?!)
- 나의 의도와는 다르게 (단건에 대한 필드에만 동작, 정해진 몇몇 필드만 사용할 예정) 시간이 지나서 다른 개발자가 사용하게 된다면? => **에러 발생 확률 높아짐**

이와같은 이유로 다른 방법을 고려하게 됨.
## Visitor Pattern
팀원분이 Visitor pattern 적용해보는 것을 추천해주셨다. 
참고: https://sourcemaking.com/design_patterns/visitor

로그를 찍고자 하는 클래스가 추가될 때마다 VisitorImpl에 메소드가 추가 됨.

### 장점
- 로그를 찍는 사소하지만 공통적인 작업을 한 클래스 내부에서 관리할 수 있게 됨.

### 단점
- 결제Service 내부의 몇 메소드에서만 사용하는 로깅용 method 인데 굳이 이렇게 적용할 필요가 있을까? 주문 도메인 전체에서 사용하는 공통적인 에러로그 작업이라면 고려해 볼만 할 것 같다.
그래서 다른 방법을 다시 고민해 보았다.

## toString() 같은 메소드
결국에는 다시 돌아와서 toString().
아무리 생각해도 **에러 확률도 가장 적고**, 기존에 있는 코드를 수정하지 않는 선에서 할 수 있는 최선의 방법이라고 생각하였다. (처음부터 이렇게 할걸...)  
에러로깅메소드를 사용하는 몇 클래스/request model에서는 이미 toString() 을 override해서  사용하고 있어서 구분짓기 위해 `toPaymentErrorLog()` 라는 메소드를 추가하였다.  
그리고 지금까지 별로 중요하게 생각하지 않았던 부분도 있었다. 바로 method의 이름!!  

## method의 이름 재 정의
`loggingErrorAndWrapping정의한Exception`
메소드 명만 봤을때는 해당 메소드의 용도는 이해가 된다. 에러를 로그로 남기고 정의한 exception을 wrapping해서 반환한다.   
하지만 너무 길다. (물론 긴 메소드명이 틀렸다는 것은 아니지만, 더 명확하지만 짧은 메소드명을 선호한다!)  
그리고 길다고 생각해서 변경하려 하니 또 마땅히 생각나는 메소드 명은 없었다. 
그렇다면 이 메소드는 한가지 일을 하지 않는 메소드 (And을 보아 짐작할 수 있다) 여서 어려운 것이라고 생각했다.   

더 좋은 메소드명으로 변경하려면 기능을 분리해야한다!
- 정의한 exception을 wrapping 하는 곳 
- 에러로그를 남기는 곳 
 
그래서 다음과 같이 변경 하였다. 
```java
/**
 * 정의한ErrorCode를 담은 정의한Exception을 반환
 */
protected 정의한Exception generate정의한Exception(Exception e, 정의한결제errorCode errorCode, Object... args) {
    if (e instanceof 정의한Exception) {
        return (정의한Exception)e;
    } else {
        print결제에러(e, args);
        return new 정의한Exception(errorCode);
    }
}

private void print결제에러(Exception e, Object... args) {
    String argsStr = "";
    for (Object arg : args) {
        if (arg != null) {
            argsStr += arg.getClass().getName() + ": ";
            Optional<Method> method = Optional.ofNullable(ReflectionUtils.findMethod(arg.getClass(),
                                                                                     "toPaymentErrorLog"));
            if (method.isPresent()) {
                argsStr += ReflectionUtils.invokeMethod(method.get(), arg) + "\\n";
            } else {
                argsStr += arg + "\\n";
            }
        }
    }
    log.error("============= 결제 에러로그 =============\\n" + argsStr, e);
}
```
## 결과
- 기존의 method를 기능별로 분리하고 명명하였다.
- toPayErrorlog가 있는 클래스는 그 포맷으로 출력하고 없는 경우에는 toString 으로 클래스의 내용을 출력하는 방향으로 수정하였다.  

마지막으로 남긴 코드가 메소드 리팩토링 진짜_마지막_리얼_마지막_코드.java 이다. 이것보다 더 좋은 방법이 물론 있으리라 믿는다.  

