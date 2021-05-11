# @RestController와 JSON 처리

참고자료 :

[https://round-round.tistory.com/entry/REST-API의-단점-3가지](https://round-round.tistory.com/entry/REST-API%EC%9D%98-%EB%8B%A8%EC%A0%90-3%EA%B0%80%EC%A7%80)

[https://juyeop.tistory.com/24](https://juyeop.tistory.com/24)

[https://velog.io/@surim014/JSON이란-무엇인가](https://velog.io/@surim014/JSON%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80)

## Rest 란?

Representational State Transfer"의 약자로 HTTP기반으로 필요한 자원에 쉽게 접근할 수 있게 HTTP의 장점을 최대한 활용할 수 있는 아키 텍처이다. Rest방식을 따른 시스템을 RESTful이라고 부른다.

### REST 아키텍처의 장점

1. 자원이 존재하는 Server , 자원을 요청하는 Client로 명확하게 구분된다.
2. HTTP 표준 방식을 활용한다면 어떤 플랫폼에서도 똑같이 사용 가능하다.
3. 3.URI 주소 자체가 동사 + 명사로 이루어져 어떤 기능을 수행하는지 유추 할 수 있다.

### REST 아키텍처의 단점

1. 표준 규약이 없다.
2. 안티패턴으로 설계될 가능성이 높다

→ 안티패턴 : REST의 특징을 이해하지 못하고 REST 사상에 어긋나는 패턴

3.RDBMS의 표현에 적합하지 않다.

→REST는 리소스 표현을 JSON,XML을 사용.

## API란?

"Application Programming Interface"의 약자로 응용 프로그램에서 사용할 수 있도록 운영 체제나 프로그래밍 언어가 제공하는 기능을 제어할 수 있게 만든 인터페이스.

## REST API 특징

CRUD 관점에서 알맞은 메소드들을 설명하겠다.

Create(Post) : POST 메서드를 통해 해당 리소스를 생성하겠다.

Read(GET) : GET 메서드를 통해 해당 리소스를 조회하겠다.

UPDATE(PUT): PUT메서드를 통해 해당 리소스를 수정하겠다.

DELETE(DELETE): DELETE 메서드를 통해 해당 리소스를 삭제 하겠다.

## JSON과 AJAX 통신

### JSON

:JavaScript Object Notation라는 의미의 축약어로 데이터를 저장하거나 전송할 때 많이 사용되는 경량의 DATA 교환 형식

:Javascript에서 객체를 만들 때 사용하는 표현식을 의미한다.

### 특징

자바스크립트 객체 표기법과 아주 유사하다.

다른 프로그래밍 언어를 이용해서도 쉽게 만들 수 있다.

특정 언어에 종속되지 않으며, 대부분의 프로그래밍 언어에서 JSON 포맷의 데이터를 핸들링 할 수 있는 라이브러리를 제공한다.

```java
{
  "employees": [
    {
      "name": "Surim",
      "lastName": "Son"
    },
    {
      "name": "Someone",
      "lastName": "Huh"
    },
    {
      "name": "Someone else",
      "lastName": "Kim"
    } 
  ]
}
```

우리는 스프링부트에서 @RestController를 이용해서 Json을 리턴값으로 반환해서 프론트와 통신을 한다. 프론트에서는 AJAX를 사용해서 JSON을 받아서 가공하여 사용자에게 보여준다.

## AJAX

Asynchronous Javascript And XML의 약자로서 자바스크립트의 라이브러리중 하나이다. 해석하자면 "비동기식 자바스크립트와 XML"이다. 

### 비동기 방식이란?

웹페이지를 리로드하지 않고 데이터를 불러오는 방식. Ajax를 통해 서버에 요청을 한 후 멈추어 있는 것이 아니라 그 프로그램은 계속 돌아간다는 의미를 내포하고 있다.

### AJAX 사용 이유

단순히 데이터를 조회하고 싶을때 페이지 전체를 새로고침하지 않기 위해 사용한다고 볼 수 있다. 

기존의 동기 방식 → 클라이언트에서 request를 보내고 서버쪽에서 response를 받으면 이어졌던 연결이 끊김. 화면의 내용 갱신시 다시 request를 하고 response를 하며 페이지 전체 갱신. → 자원낭비 심함

ajax 사용 방식 → 페이지 전체가 아닌 일부분만 갱신할 수 있도록 XMLHttpRequest 객체를 통해 서버에 request → 자원과 시간 절약
