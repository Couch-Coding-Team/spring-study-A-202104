# 스프링 MVC와 Thymeleaf

docs:

[https://developer.mozilla.org/ko/docs/Glossary/MVC](https://developer.mozilla.org/ko/docs/Glossary/MVC)

## Thymeleafe를 사용하는 프로젝트 생성
![Untitled](https://user-images.githubusercontent.com/76509935/115840392-980c0780-a456-11eb-9df0-637845f6e54f.png)

build.gradle에 의존성을 추가하여야 한다.

Thymeleaf를 사용할 경우 만들어진 결과를 보관하는 기능인 캐싱을 꺼두는 것이 편리하다.

## why 캐싱?

캐싱을 끄는 이유는 템플릿을 수정할 경우 즉시 템플릿의 변경값을 변환해주기 위해서다.

spring boot의 경우 default 값이 캐싱을 지원한다. 따라서 설정을 통해 캐싱기능을 막아야한다.

![Untitled 1](https://user-images.githubusercontent.com/76509935/115840393-980c0780-a456-11eb-8ea7-5abe3ed4ac3a.png)
스프링의 경우에는 MVC패턴을 사용한다.

## Spring MVC란?

디자인패턴의 하나이며 Model - View -Controller 의 약자이다. 

Model: 데이터와 비즈니스 로직을 관리.

View: 레이아웃과 화면을 처리한다.

Controller: 명령을 모델과 뷰 부분으로 라우팅한다.
![Untitled 2](https://user-images.githubusercontent.com/76509935/115840396-98a49e00-a456-11eb-8f5f-68551f0f0c19.png)
여기서 우리가 배우는 Thymeleaf가 뷰를 담당하는 역할을 한다.

컨트롤러를 사용하는 방법은 다음과 같다.
![Untitled 3](https://user-images.githubusercontent.com/76509935/115840398-993d3480-a456-11eb-9e7b-38b880ab9485.png)
controller 클래스를 만들어주고 클래스 위에 @Controller를 붙여준다. 위에 사진을 보면 @RestController라고 되어있는데 이 경우에는 @Controller + @ResponseBody가 합쳐진 것으로 반환형을 Json으로 하는 효과가 있다.

위와같이 어노테이션을 사용하면 간단하게 컨트롤러를 만들 수 있다. 또한 @GetMapping과 같이 Http메서드(GET,POST,PUT,DELETE)들과 매칭 시킬 수 있다.

교재를 보면 @GetMapping이 아니라 @RequestMapping이 되어있는데

![Untitled 4](https://user-images.githubusercontent.com/76509935/115840400-993d3480-a456-11eb-937d-85b761637c4e.png)
get매핑을 살펴보면 @RequestMapping(method = RequestMethod.GET)으로 되어 있는 것을 살펴볼 수 있다. 기존에는 @RequestMapping으로 method의 속성값을 주입시켜 매핑하였으나 요즘에는 GetMapping ,PostMapping 처럼 어노테이션으로 바로 사용할 수 있게 변화되어 왔다.

이렇게 컨트롤러의 형식과 뒤에 url을 지정해주면 입력된 주소값으로 컨트롤러에 매핑이 되고 컨트롤러에서 view로 반환을 시켜주는 형식으로 동작한다.

![Untitled 5](https://user-images.githubusercontent.com/76509935/115840388-96dada80-a456-11eb-8b5d-c5f562226df2.png)

/hello url로 들어와 hello.html이랑 매핑.

추가로 말하자면 controller에서 어떻게 url을 인식하는지 궁금하다면 DispatcherServlet을 공부해보자.

# Thymeleaf의 기본객체와 LocalDateTime

타임리프는 내부적으로 여러 객체를 지원합니다.

ex) 문자,숫자, 웹에서 사용하는 파라미터, request ,response ,session 등 다양하게 지원

jps의 경우 숫자와 날짜를 처리하기 위해서는 별도의 JSTL설정이 필요,

타임리프의 경우 #number #date등을 별도로 설정 없이 사용할 수 있다.

ex)화면에 출력하는 숫자(dto.sno)를 모두 5자리로 만들어야 하는 상황.

#numbers.formatInteger(dto.sno,5)

이렇게 내장객체들이 다양하게 지원을 하나 java 8 에 등장하는 LocalDate타입이나 LocalDateTime에 대해서는 아직까지 복잡한 방식으로 처리 해야댐. → 단점
![Untitled](https://user-images.githubusercontent.com/76509935/115840403-99d5cb00-a456-11eb-95c0-c469ab980545.png)

이를 편하게 처리하기 위해 타임리프 java8time 의존성을 추가.

#temporals라는 객체를 사용하여서 format메소드를 통해 편하게 처리할 수 있게 된다.

#temporals.format(dto.regTime,'yyyy/MM/dd')

이런 형식으로 표현.
