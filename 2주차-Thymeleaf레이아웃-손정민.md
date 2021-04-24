---
layout: post
title:  "Thymeleaf의 레이아웃"
date:   2021-04-23 10:23:01 +0900
categories: [Spring, Java, JPA, Thymeleaf]
tags: [thymeleaf]
---

## 왜! JPA를 사용해야 하는가?
- 생산성
  - JPA를 사용하면 자바컬렉션에 객체를 저장하듯이 JPA에게 저장할 객체를 전달하면된다.
  - INSERT SQL을 작성하고 JDBC API를 사용하는 지루하고 반복적인 코드와 CRUD용 SQL을 개발자가 직접 작성하지 않아도 된다.
- 유지보수
  - SQL을 직접 다루면 엔티티에 필드를 하나만 추가해도 관련된 SQL과 결과를 매핑하기 위한 JDBC API코드를 코두 변경해야 했지만 JPA를 사용하면 이런 과정을 JPA가 대신 처리해주므로 필드를 추가하거나 삭제해도 수정해야 할 코드가 줄어든다.
- 패러다임 불일치 해결
  - JPA는 상속, 연관관계, 객체 그래프 탐색, 비교하기와 같은 패러다임의 불일치 문제를 해결해 준다.
- 성능
  - JPA는 애플리케이션과 데이터베이스 사이에서 다양한 성능 최적화 기회를 제공한다.

   > String memberld = "helloId";
   > Member member1 = jpa.find(memberId);
   > Member member2 = jpa.find(memberId);

   같은 트랜잭션 안에서 같은 회원을 두 번 조회하는 코드의 일부분
   JDBC API를 사용해서 해당 코드를 직접 작성했다면 회원을 조회 할 때마다 SELECT SQL을 사용해서 데이터베이스와 두 번 통신 해야한다.
   JPA를 사용하면 회원을 조회하는 SELECT SQL을 한 번만 데이터베이스에 전달하고 두 번째는 조회한 객체를 재사용한다.
- 데이터 접근 추상화와 벤더 독립성
  - JPA는 애플리케이션과 데이터베이스 사이에 추상화도니 데이터 접근 계층을 제공해서 애플리케이션이 특정 데이터 베이스 기술에 종속되지 않도록 한다.

<img src="C:\GitBlog\spring-study-A-202104\image1.jpg">
 
- 표준
  - JPA는 자바 진영의 ORM 기술 표준이다.
  - 표준을 사용하면 다른 구현 기술로 손쉽게 변경할 수 있다.
## 3.4 Thymeleaf의 레이아웃
`ref` [DOCS](https://www.thymeleaf.org/documentation.html "https://www.thymeleaf.org/documentation.html")
+ Thymeleaf의 레이아웃 기능
   1. JSP의 include와 같이 <u>특정 부분을 외부 혹은 내부에서 가져와서 포함</u> 하는 형태
   2. 특정한 부분을 파라미터로 전달해서 내용에 포함하는 형태
---
### 3.4.1 include 방식의 처리
Thymelead의 기능 중에서는 특정한 부분을 다른 내용으로 변경할 수 있는 __th:insert__ 나 __th:replace__ 기능 존재
> __th:replace__ : 기존의 내용을 완전히 '대체'하는 방식
> __th:insert__ : 기본 내용의 바깥쪽 태그는 그대로 유지하면서 추가되는 방식

 #### 실습 1: 다른 파일에 있는 일부분을 조각처럼 가져와 exLayout.html에 구성

👇 SampleController에 메소드 추가
``` java
@GetMapping("/exLayout1")
public void exLayout1() {
    log.info("exLayout...............");
}
```

👇 fragment1.html 작성
``` html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

    <div th:fragment="part1">
        <h2>Part 1</h2>
    </div>
    <div th:fragment="part2">
        <h2>Part 2</h2>
    </div>
    <div th:fragment="part3">
        <h2>Part 3</h2>
    </div>

</body>
</html>
```

👇 th:fragment를 가져다 쓸 exLayout1.html 작성
``` html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf/org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

  <h1>Fragment Test</h1>

  <h1>Layout 1 - 1</h1>
  <div th:replace="~{/fragments/fragment1 :: part1}" ></div>

  <h1>Layout 1 - 2</h1>
  <div th:insert="~{/fragments/fragment1 :: part2}" ></div>

  <h1>Layout 1 - 1</h1>
  <th:block th:replace="~{/fragments/fragment1 :: part3}" ></th:block>

</body>
</html>
```

 #### 실습 2: 파일 전체를 조각으로 사용하는 방법
 
 👇 fragment2.html 작성
 ``` html
 <!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

  <div>
    <hr />
    <h2>Fragment2 File</h2>
    <h2>Fragment2 File</h2>
    <h2>Fragment2 File</h2>
    <hr />
  </div>

</body>
</html>
 ```
 👇 exLayout1.html에 파일 전체를 가져오는 부분 추가
 ``` html
 <!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf/org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

  <h1>Fragment Test</h1>
  <div style="border: 1px solid blue">
      
      <th:block th:replace="~{/fragments/fragment2}"></th:block>
      
  </div>
  <h1>Layout 1 - 1</h1>
  <div th:replace="~{/fragments/fragment1 :: part1}" ></div>

  <h1>Layout 1 - 2</h1>
  <div th:insert="~{/fragments/fragment1 :: part2}" ></div>

  <h1>Layout 1 - 1</h1>
  <th:block th:replace="~{/fragments/fragment1 :: part3}" ></th:block>

</body>
</html>
 ```
th:replace="~{/fragments/fragment2}" 부분에 '::'로 처리되는 부분이 없으므로 전체의 내용을 반영

 #### 실습 3: 파라미터 방식의 처리
👇 @GetMapping의 값을 배열로 수정
``` java
@GetMapping("/exLayout1","/exLayout2"})
public void exLayout1() {
    log.info("exLayout...............");
}
```

👇 fragment3.html
``` html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

  <div th:fragment="target(first, second)">
    
    <style>
      .c1 {
        background-color: red;
      }
      .c2 {
        background-color: blue;
      }
    </style>
    
    <div class="c1">
      <th:block th:replace = "${first}"></th:block>
    </div>
    <div class="c2">
      <th:block th:replace = "${second}"></th:block>
    </div>
    
  </div>

</body>
</html>
```
👇 exLayout2.html
``` html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf/org">

<th:block th:replace="~{/fragments/fragment3:: target(~{this:: #ulFirst} , ~{this::#ulSecond} )}">
   <!-- this: #ulFirst - this는 현재 페이지를 의미할 때 사용하는데 생략이 가능합니다.
               #ulFirst는 CSS의 id선택자를 의미 -->
  <ul id="ulFirst">
    <li>AAA</li>
    <li>BBB</li>
    <li>CCC</li>
  </ul>

  <ul id="ulSecond">
    <li>111</li>
    <li>222</li>
    <li>333</li>
  </ul>
  
</th:block>
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>

</body>
</html>
```

### 3.4.3 부트스트랩 템플릿 적용하기
`ref` [디자인](https://startbootstrap.com/templates/simple-sidebar/ "https://startbootstrap.com/templates/simple-sidebar/")

다운로드 받은 파일을 프로젝트 내에 생성된 static 폴더 내로 복사 
-> layout 폴더에 basic.html 파일을 추가하고 위의 파일들 중에서 index.html의 내용을 그대로 추가 
-> basic.html 파일에서 가장 먼저 수정해야 하는 부분은 상단에 Thymeleaf와 관련된 설정을 추가
``` html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">

<th:block th:fragment="setContent(content)">    
<head>
```
-> CSS나 Javascript의 링크를 처리하는 코드 추가
``` html
<title>Simple Sidebat - Start Bootstrap Template</title>

<!-- Bootstrap core CSS -->
<link th:href="@{/vendor/bootstrap/css/bootstrap.min.css}" rel="stylesheet">

<!-- Custom styles for this template -->
<link th:href="@{/css/simple-sidebar.css}" rel="stylesheet">

<!-- Bootstrap core Javascript -->
<script th:href="@{/vendor/jquery/jquery.min.js}"></script>
<script th:href="@{/vendor/bootstrap/js/bootstrap.min.js}"></script>
```

-> 실행파일
``` html
<th:block th:replac="~{/layout/basic :: setContent(~{this::content} )}">

<th:block th:fragment="content">
    <h1> ~~ </h1>
```