---
layout: post
title:  "프로젝트 구조 만들기"
date:   2021-04-28 11:49:23 +0900
categories: [Markdown]
tags: [markdown]  
---

# 4-1 프로젝트의 와이어프레임

## 4-1-1 프로젝트의 화면 구성

하나의 엔티티 클래스를 이용해 CRUD 기능과 검색/페이징 기능을 가지는 웹 애플리케이션 제작

---

![4-1%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%AA%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%A5%E1%84%91%E1%85%B3%E1%84%85%E1%85%A6%E1%84%8B%E1%85%B5%E1%86%B7%204809a71c3e7942a280ea6dc768889558/image_1.png](image/image_1.png)

주요 기능 이미지

- 화면 개발의 목표
    - 목록 화면(번호 1) - 전체 목록을 페이징 처리해서 조회, 제목/내용/작성자 항목으로 검색과 페이징 처리를 가능하게 함
    - 등록 화면(번호 2) - 새로운 글 등록, 등록 처리 후 다시 목록 화면으로 이동
    - 조회 화면(번호 3) - 목록 화면에서 특정한 글을 선택하면 자동으로 조회 화면으로 이동

        조회 화면에서는 수정/삭제가 가능한 화면(번호 4)으로 버튼을 클릭해서 이동

    - 수정/삭제 화면(번호 4) - 수정 화면에서 삭제가 가능, 삭제 후에는 목록 페이지로 이동.

        글을 수정하는 경우에는 다시 조회 화면(번호 2)으로 이동해서 수정된 내용 확인

---

[실제 구현해야 하는 기능](https://www.notion.so/6d1282013ccc44c8abec40bdc905a6e2)

---

![4-1%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%AA%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%A5%E1%84%91%E1%85%B3%E1%84%85%E1%85%A6%E1%84%8B%E1%85%B5%E1%86%B7%204809a71c3e7942a280ea6dc768889558/__2021-04-27_214419.jpg](image/image_2.png)

프로젝트의 기본 구조

---

![4-1%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%AA%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%A5%E1%84%91%E1%85%B3%E1%84%85%E1%85%A6%E1%84%8B%E1%85%B5%E1%86%B7%204809a71c3e7942a280ea6dc768889558/__2021-04-27_214432.jpg](image/image_3.png)

DTO와 엔티티 객체의 역할

- 브라우저에서 전달되는 Request는 GuestbookController에서 DTO의 형태로 처리됨
- GuestbookRepository는 엔티티 타입을 이용하므로 중간에 Service 계층에서는 DTO와 엔티티의 변환을 처리

## 4-1-2 프로젝트 생성

![4-1%20%E1%84%91%E1%85%B3%E1%84%85%E1%85%A9%E1%84%8C%E1%85%A6%E1%86%A8%E1%84%90%E1%85%B3%E1%84%8B%E1%85%B4%20%E1%84%8B%E1%85%AA%E1%84%8B%E1%85%B5%E1%84%8B%E1%85%A5%E1%84%91%E1%85%B3%E1%84%85%E1%85%A6%E1%84%8B%E1%85%B5%E1%86%B7%204809a71c3e7942a280ea6dc768889558/InkedUntitled_LI.jpg](image/image_4.png)

        

  — MariaDB관련 JDBC 드라이버 추가, JPA와 관련된 설정 추가 —

```java
// build.gradle 파일 일부
dependencies {
	implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
	implementation 'org.springframework.boot:spring-boot-starter-thymeleaf'
	implementation 'org.springframework.boot:spring-boot-starter-web'
	compileOnly 'org.projectlombok:lombok'
	developmentOnly 'org.springframwork.boot:spring-boot-devtools'
	annotationProcessor 'org.projectlombok:lombok'
	provideRuntime 'org.springframework.boot:spring-boot-starter-tomcat'
	testImplementation('org.springframwork.boot:spring-boot-starter-test') {
			exclude group: 'org.junit.vintage', module: 'junit-vintage-engine'
	}
	compile group: 'org.mariadb.jdbc', name: 'mariadb-java-client'
	compile group: 'org.thymeleaf.extras', name: 'thymeleaf-extras-java8time'
}
```

— 데이터베이스 관련 설정 —

```java
spring.datasource.driver-class-name=org.mariadb.jdbc.Driver
spring.datasource.url=jdbc:mariadb://localhost:3306/bootex
spring.datasource.username=bootuser
spring.datasource.password=bootuser

spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.show-sql=true

spring.thymeleaf.cache=false
```

## 4-1-3 컨트롤러/화면 관련 준비

GuestbookController 클래스 생성

```java
package org.zerock.guestbook.controller;

import lombok.extern.log4j.Log4j2;
import org.springframwork.stereotype.Controller;
import org.springframwork.web.bind.annotation.GetMapping;
import org.springframwork.web.bind.annotation.RequestMapping;

@Controller
@RequestMapping("/guestbook")
@Log42
public class GuestbookController {

	@GetMapping({"/","list"})
	public String list(){
		log.info("list..............");

		return "/guestbook/list"
	}
}
```